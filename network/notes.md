

https://forum.proxmox.com/threads/howto-proxmox-ve-8-1-2-wifi-w-snat.142831/



https://systemzone.net/static-routing-configuration-in-mikrotik-router/


https://pve.proxmox.com/wiki/Setup_Simple_Zone_With_SNAT_and_DHCP


### Added to Mikrotik

/ip route
add distance=1 dst-address=10.10.0.0/24 gateway=192.168.88.25


sudo nano /etc/network/interfaces.d/sdn
```
auto vnet1
iface vnet1 static
        address 10.10.0.1/24
        bridge_ports none
        bridge_stp off
        bridge_fd 0
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s '10.10.0/24' -o wlp2s0 -j MASQUERADE
        post-up iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone zone1
        post-down iptables -t nat -D POSTROUTING -s '10.10.0.0/24' -o wlp2s0 -j MASQUERADE
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone zone1
```


When I tried the "systemctl restart wpa_supplicant && systemctl restart networking":
```
● networking.service - Network initialization
     Loaded: loaded (/lib/systemd/system/networking.service; enabled; preset: enabled)
     Active: active (exited) since Sun 2025-03-23 23:21:15 PDT; 5min ago
       Docs: man:interfaces(5)
             man:ifup(8)
             man:ifdown(8)
    Process: 26994 ExecStart=/usr/share/ifupdown2/sbin/start-networking start (code=exited, status=0/SUCCESS)
   Main PID: 26994 (code=exited, status=0/SUCCESS)
        CPU: 1.117s

Mar 23 23:21:13 cc-pve-mint networking[26994]: networking: Configuring network interfaces
Mar 23 23:21:14 cc-pve-mint networking[27013]: error: /etc/network/interfaces.d/sdn: line4: iface vnet1: unsupported address family 'static'
Mar 23 23:21:14 cc-pve-mint /usr/sbin/ifup[27013]: error: /etc/network/interfaces.d/sdn: line4: iface vnet1: unsupported address family 'static'
Mar 23 23:21:15 cc-pve-mint networking[27013]: warning: vnet1: post-up cmd 'iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone zone1' failed: returned 2 (iptables v1.8.9 (legacy): Cannot parse zone1 as a zone ID
Mar 23 23:21:15 cc-pve-mint networking[27013]: Try `iptables -h' or 'iptables --help' for more information.
Mar 23 23:21:15 cc-pve-mint networking[27013]: )
Mar 23 23:21:15 cc-pve-mint /usr/sbin/ifup[27013]: warning: vnet1: post-up cmd 'iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone zone1' failed: returned 2 (iptables v1.8.9 (legacy): Cannot parse zone1 as a zone ID
                                                   Try `iptables -h' or 'iptables --help' for more information.
                                                   )
Mar 23 23:21:15 cc-pve-mint networking[27013]: error: >>> Full logs available in: /var/log/ifupdown2/network_config_ifupdown2_265_Mar-23-2025_23:21:14.225240 <<<
Mar 23 23:21:15 cc-pve-mint /usr/sbin/ifup[27013]: >>> Full logs available in: /var/log/ifupdown2/network_config_ifupdown2_265_Mar-23-2025_23:21:14.225240 <<<
Mar 23 23:21:15 cc-pve-mint systemd[1]: Finished networking.service - Network initialization.
```
