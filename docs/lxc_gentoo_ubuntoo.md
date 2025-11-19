---
title: lxc Gentoo in Ubuntu
icon: lucide/package-open
---

чтобы при emerge не было ошибок добавить в конфиг `lxc_name/config`:

```bash
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```
