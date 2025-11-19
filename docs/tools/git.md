---
title: Лайфхаки в Git
icon: lucide/git-branch
---

### Можно не вводить час пароль даже при https:// ( 1 раз надо будет для сохраненния в кэше все равно)
``` bash
git config --global credential.helper 'cache --timeout=3600' 
```

### Сквошим все коммиты начиная от рута 
``` bash
git rebase -i --root 
!!! note "" "вместо pick с 2 по последний коммит и вот мы сквошнули все коммиты в один"
