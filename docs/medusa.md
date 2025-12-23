---
title: medusa
icon: lucide/ethernet-port
---

[посление релизы](`https://github.com/jonasvinther/medusa/releases/tag/v0.7.3`)

#### выведет в консоль что на сервере, тут же можем перенаправить в файл этот вывод:
```
medusa export internal/namespace/common/ --address="https://vault.server.online" --token="hvs.*****" > /.medusa-dump.yaml
```

#### запишет на сервер то, что в файле .medusa-dump-edited.yaml (в yaml уже сохранены полные пути, поэтому указываем корень kv2 хранилища удаленного сервера):
```
medusa import internal/ .medusa-dump-edited.yaml --address="https://vault.server.online" --token="hvs.**" --insecure
```


