# configuring a dns

install dnsmasq and put this in /etc/dnsmasq.conf
```
interface=wg0
bind-interfaces
domain=YOUR-DOMAIN
expand-hosts
 
domain-needed
bogus-priv
no-resolv
server=8.8.8.8
 
addn-hosts=/etc/dnsmasq.hosts
```

in `/etc/dnsmasq.hosts` put this (change things for your case)
```
10.xxx.xxx.xxx NAME.YOUR-DOMAIN
```

later restart the service with:
```sh
sudo systemctl restart dnsmasq.service
```

and you should be good to go.
Remember about configuring your vpn (covered in the vpn section)
