---
title: на примере GitLab CI:
---

``` yaml
stages:
   - build

build_buildah:
  rules:
  - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  - if: $CI_COMMIT_BRANCH == "dev" || $CI_COMMIT_BRANCH == "master" || $CI_COMMIT_BRANCH =~ /^release\/[0-9-\.]+$/
  - if: $CI_PIPELINE_SOURCE == "push"
    when: manual
  stage: build
  tags:
  - kubernetes
  image:
    name: quay.io/buildah/stable:latest
    entrypoint: [ "" ]
  before_script:
  - buildah login -u ${CI_REGISTRY_USER} -p "${CI_REGISTRY_PASSWORD}" ${CI_REGISTRY}
  - mkdir -p /cache
  script:
  - echo "Building image with buildah ..."
  # Собираем образ
  - buildah bud --layers --root /cache/root --runroot /cache/run -t "$DOCKER_TAG_DEV"
  # Пушим образ
  - echo "Pushing image..."
  - buildah push --root /cache/root --runroot /cache/run "$DOCKER_TAG_DEV"
  variables:
    PROJECT_TAG: "$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME:$CI_COMMIT_REF_SLUG.${CI_PIPELINE_ID}"
    DOCKER_TAG_DEV: "$CI_REGISTRY_IMAGE:latest"
    DOCKER_TAG_COMMIT: "$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG.$CI_PIPELINE_ID"
```

