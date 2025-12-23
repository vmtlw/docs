---
title: longhorn
icon: lucide/cpu
---

#### На каждой ноде выполните:
```bash
apt install -y nfs-common cryptsetup dmsetup open-iscsi
modprobe -a iscsi_tcp dm_crypt
echo -e "iscsi_tcp\ndm_crypt" > /etc/modules-load.d/longhorn.conf
systemctl enable --now iscsid.service
```


