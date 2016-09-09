title: centos7 firewalld with pptpd
date: 2016-09-09 07:19:23
tags:
---


Install pptpd with: 
```
yum -y install ppp pptpd
```

```
# pptpd settings
echo 'localip 10.10.0.1' >> /etc/pptpd.conf
echo 'remoteip 10.10.0.100-199' >> /etc/pptpd.conf
echo 'ms-dns 8.8.8.8' >> /etc/ppp/options.pptpd
echo 'ms-dns 8.8.4.4' >> /etc/ppp/options.pptpd
echo 'USERNAME pptpd PASSWORD *' >> /etc/ppp/chap-secrets


# system ipv4 forward
sysctl_file=/etc/sysctl.conf
if grep -xq 'net.ipv4.ip_forward' $sysctl_file; then
sed -i.bak -r -e "s/^.*net.ipv4.ip_forward.*/net.ipv4.ip_forward = 1/" $sysctl_file
else
echo 'net.ipv4.ip_forward = 1' >> $sysctl_file
fi

sysctl -p
```


``` bash

firewall-cmd --permanent --new-service=pptpd

cat > /etc/firewalld/services/pptpd.xml<<EOF
<?xml version="1.0" encoding="utf-8"?>
<service>
  <port protocol="tcp" port="1723"/>
  </service>
EOF

firewall-cmd --permanent --zone=public --add-service=pptpd
firewall-cmd --permanent --zone=public --add-masquerade

firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p gre -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv6 filter INPUT 0 -p gre -j ACCEPT
firewall-cmd --reload

```


```
# start pptpd
systemctl start pptpd
systemctl enable pptpd.service
```
