---
title: Troubleshooting VLANs
date: '2023-12-11 17:00:00 +0300'
categories:
  - Linux
tags:
  - vlan
render_with_liquid: false
published: true
---

![switch](/assets/img/common/switch1.jpg)

## SITUACION

Tenemos un servidor con dos interfaces creadas de tipo VLAN asignadas a un mismo bond (varias tarjetas de red configuradas como si fuesen una sola)
*Este ejemplo aplica de la misma manera para una interfaz de red simple. Donde pone "bond0" aparecerá como "ethX" o "enoX".

VLAN 100 y 200

## OBJETIVO

Confirmar que la VLAN está bien extendida hasta el servidor (con VLAN tagging)

## PRUEBAS

Vamos a ver las dos formas que hay para levantar el tcpdump y ver si hay tráfico.

Listamos todas las interfaces. Se puede ver que hay dos interfaces de tipo VLAN sobre la interfaz bond0
```plaintext
[root@PRUEBAS01 ~]# nmcli con show | grep bond0
bond0            4531b0e7-c01a-4183-a7f6-eeb993fac155  bond      bond0
bond0.100       7ea38a18-16b5-4b69-a246-9ae1a4c12a52  vlan      bond0.100
bond0.200       7ea38r1d-1l45-4029-as46-9aloa4c12a51  vlan      bond0.200
```

**Comprobación del VLAN ID 100 sobre la interfaz bond0 que lleva el tráfico taggeado**
```plaintext
[root@PRUEBAS01 ~]# tcpdump -i bond0 -n -e '(vlan 100)'
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on bond0, link-type EN10MB (Ethernet), capture size 262144 bytes
^C
0 packets captured
86 packets received by filter
54 packets dropped by kernel
```
0 paquetes capturados. No hay tráfico.

**Comprobación del VLAN ID 200 sobre la interfaz bond0.200. Aquí no especificamos el VLAN ID ya que el tráfico ya no va taggeado en esta interfaz**
```plaintext
[root@PRUEBAS01 ~]# tcpdump -i bond0.200 -n -e
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on bond0.1654, link-type EN10MB (Ethernet), capture size 262144 bytes
12:29:03.391232 b4:de:11:db:0b:99 > 01:00:0d:ce:cc:dc, 802.3, length 70: LLC, dsap SNAP (0xaa) Individual, ssap SNAP (0xaa) Command, ctrl 0x03: oui Cisco (0x00000c), pid Unknown (0x0139), length 62:
       0x0000:  aaaa 0300 000c 0139 0104 0000 0001 0204  .......9........
       0x0010:  0000 00cc 030c 0000 00cb 0000 000a 6440  ..............d@
       0x0020:  c460 0404 1600 001c 0504 62e8 fb52 0614  .`........b..R..
       0x0030:  b94a 6ddf 5dae edef 9378 52d8 ccd4 f3a5  .Jm.]....xR.....
```	   

## CONCLUSIONES

La VLAN 100 no está bien extendida hasta el servidor, no se vé tráfico por ella.
La VLAN 200 si parece estar bien pasada, vemos tráfico

Esta prueba es muy interesante ya que la podemos ejecutar sobre la interfaz bond0 (o "ethX, "enoX", etc) antes de configurar la interfaz (configurar ip, netmask, gateway/rutas, etc)
