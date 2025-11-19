---
title: на примере GitLab CI:
---

``` yaml

.deploy_argocd:
  stage: deploy
  image: alpine
  variables:
    NAME: ""
    TAG: ""
    ITAG: "$CI_COMMIT_REF_SLUG.${PIPELINE_ID}"
  tags:
  - kubernetes
  before_script:
  - yes | argocd login argocd-server.argocd.svc.cluster.local --username $ARGOCD_USERNAME --password $ARGOCD_PASSWORD --insecure --grpc-web || true
  script:
  - echo "Синхронизация с ArgoCD NAME=$NAME, ITAG=$ITAG"
  - argocd app set "$NAME" --parameter "$TAG=$ITAG"
  - |
    timeout=120
    echo "Ожидание распространения нового tag в ArgoCD (TAG=$ITAG)"
    start_time=$(date +%s)
    found=0
    while true; do
      found=$(argocd app get "$NAME" -o json \
        | jq -r --arg TAG "$ITAG" '
            (.status.operationState.syncResult.resources // [])
            | map(
                select(.kind == "Deployment")
                | select((.images // [])[] | endswith(":"+$TAG))
              )
            | length
          ')
      if [ "$found" -gt 0 ]; then
        echo "Тег найден, продолжаю"
        break
      fi
      now=$(date +%s)
      elapsed=$((now - start_time))
      if [ "$elapsed" -ge "$timeout" ]; then
        echo "достигнут тайм-аут ожидания распространения тега в ArgoCD ( timeout=$timeout )"
        exit 1
      fi
      sleep 1
    done
  # Проверяем миграции, если есть - ожидаем их завершения ( допустима лишь одна job в PreSync , иначе развалится и нужно будет переписать этот блок для нескольких джоб одновременно)
  - |
    pre_sync_job=$(argocd app get "$NAME" -o json | jq -r '
      .status.operationState.syncResult.resources[]
      | select(.hookType=="PreSync")
      | .name
      ')
    if [ -n "$pre_sync_job" ]; then
      echo "PreSync job $pre_sync_job detected, waiting..."
      if ! argocd app wait "$NAME" \
          --resource batch:Job:$pre_sync_job \
          --operation \
          --timeout 300; then
        echo "PreSync job failed or timed out, showing logs"
        argocd app logs "$NAME" --kind Job
        exit 1
      fi
      
    else
      echo "No PreSync job detected, continuing..."
    fi
  # Получаем изменённые деплойменты ( если были джобы важно чтобы они были выполнены и завершены до этого этапа)
  - |
    deploys=$(argocd app get "$NAME" -o json \
      | jq -r --arg TAG "$ITAG" '
        [.status.operationState.syncResult.resources[]
        | select(.kind == "Deployment")
        | select(.images[] | endswith(":"+$TAG))
        | .name] | unique | join(" ")
      ')
  # Если ничего не изменилось — выходим
  - |
    if [ -z "$deploys" ]; then
      echo "No deployments changed, exiting"
      exit 0
    fi
  - echo "список деплойментов с указанным image tag - $deploys"
  # Формируем аргументы --resource
  - |
    resource_args=""
    for dep in $deploys; do
      resource_args="$resource_args --resource apps:Deployment:$dep "
    done
  # Ожидаем синхронизацию только изменённых деплойментов
  - argocd app wait "$NAME" $resource_args --health --operation --timeout 300

```

