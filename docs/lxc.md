---
title: lxc
icon: lucide/cpu
---

чтобы из рф начал работать lxc-create включите vpn , так как не скачиваются ресурсы с https://fra1lxdmirror01.do.letsbuildthe.cloud/images/debian/trixie/amd64/default/ ( вечная загрузка 0 байт )


создать контейнер:
``` bash
lxc-create -n container -t download -- -d debian -r trixie -a amd64
```

