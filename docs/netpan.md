
1) удалить networking, ifupdown, networkmanager
2) поставить systemd-resolvd netplan

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1: {}
  bridges:
    br0:
      interfaces: [eno1]
      dhcp4: true
      nameservers:
        search: [vmtlw.ru]
      parameters:
        stp: false
        forward-delay: 0
```
