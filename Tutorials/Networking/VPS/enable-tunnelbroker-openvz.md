# Enable tunnelbroker on OpenVZ VPS

Enable tunnelbroker on OpenVZ.

## Requirement

A VPS based on OpenVZ with `ifconfig` `route`

## Step 1

Build [tb_userspace](https://code.google.com/archive/p/tb-tun/)

```bash
wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/tb-tun/tb-tun_r18.tar.gz
tar -vxf tb-tun_r18.tar.gz
gcc tb_userspace.c -l pthread -o tb_userspace
chmod 500 tb_userspace
mv tb_userspace /usr/sbin
```

## Step 2

Work with `systemd`

Edit `/etc/systemd/system/he-ipv6-tb-userspace.service` and add the following content.

```
[Unit]
Description=he.net IPv6 tunnel - tb_userspace
After=network.target

[Service]
Type=simple
# Stop show me the annoying log
StandardOutput=null
ExecStart=/root/tb_userspace tb [Server_IP_Address] [Client_IP_Address] sit
ExecStop=/usr/bin/killall tb_userspace

[Install]
WantedBy=multi-user.target
```

Edit `/etc/systemd/system/he-ipv6.service` and add the following content.

```
[Unit]
Description=he.net IPv6 tunnel - main
After=he-ipv6-tb-userspace.service


[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/ifconfig tb up
ExecStart=/usr/sbin/ifconfig tb inet6 add [*Routed* IPv6 address with /128 CIDR]
ExecStart=/usr/sbin/ifconfig tb mtu 1480
ExecStart=/usr/sbin/route -A inet6 add ::/0 dev tb
# If your host provides ipv6 route, uncomment this
# ExecStart=/usr/sbin/route -A inet6 del ::/0 dev venet0
ExecStop=/usr/sbin/ifconfig tb down

[Install]
WantedBy=multi-user.target
```

## Step 3

You're all set, just fire up the services

```bash
systemctl start he-ipv6-tb-userspace.service
systemctl start he-ipv6.service
# Additionally, set them to auto-start
systemctl enable he-ipv6-tb-userspace.service
systemctl enable he-ipv6.service
```

Don't forget to accept the connection from tunnelbroker `protocol number:41` from the tunnelbroker server in iptables and configure your ip6tables.