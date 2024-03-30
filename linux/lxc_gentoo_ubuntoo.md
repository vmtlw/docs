---
sidebar_position: 2
title: Create lxc container Gentoo in Ubuntu
---

### чтобы при emerge не было ошибок добавить в конфиг `$lxc_name/config`:

```
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```
