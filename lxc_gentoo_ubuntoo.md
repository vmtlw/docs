---
title: Подготовка lxc-контейнера Gentoo в Ubuntu
hide_title: true
---

чтобы при emerge не было ошибок добавить в конфиг `lxc_name/config`:

```
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```
