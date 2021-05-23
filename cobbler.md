   dhcp：protocol, 67/udp, 68/udp
	  dhcpd(isc), dnsmasq
	  dhcp:dhcpd(ipv4,ipv6),dhcrelay

   tftp server: tftp protocol, 69/udp
	  systemctl start tftp.socket

   pxe: dhcp, tftp, yum repository

   cobbler:
	  distro(repository)--> profile(kickstart)--> system(ip/mask)

	1.vim /etc/cobbler/setting
	  server:x.x.x.x
	  next_server:x.x.x.x
	  default_password_crypted: "xxxxxxx"
	  
	  # cobbler check	
	2.cp /usr/share/syslinux/{pxelinux.0,memu.c32} /var/lib/cobbler/loaders/
	  # cobbler sync

	cobbler-web
		http://host/cobbler_web/