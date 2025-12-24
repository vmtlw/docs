git config --global credential.helper 'cache --timeout=3600' , то час можно не вводить пароль даже при https:// ( 1 раз надо будет для сохраненния в кэше все равно)

git rebase -i --root + s вместо pick с 2 по последний коммит и вот мы сквошнули все коммиты в один 
