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

Tenemos un servidor físico en el que hay configuradas una tarjeta o un bond (varias tarjetas)
Desde comunicaciones nos indican que está todo bien configurado hasta el switch pero no tenemos conectividad en el servidor.


**Configuraciones posibles**

- Puerto en el switch configurado como puerto de acceso. La mas habitual
- Puerto en el swtich configurado con VLAN tagging. Por este puerto nos pueden venir el tráfico de una o más VLANs.
- Puerto híbrido. Puerto de acceso + una o más VLANs tageadas.

## PRUEBAS

Para esta prueba nos centraremos en el caso de que nos pasen el tráfico con VLAN tagging, ya que en el caso de ser un puerto de acceso sería únicamente lanzar un tcpdump indicando la interfaz.

Consultamos primero que configuración tenemos con nmcli. Aquí podemos ver un bond configurado y sobre este dos VLANs levantadas
```plaintext
[root@PRUEBAS01 ~]# nmcli con show | grep bond0
bond0            4531b0e7-c01a-4183-a7f6-eeb993fac155  bond      bond0
bond0.100       7ea38a18-16b5-4b69-a246-9ae1a4c12a52  vlan      bond0.100
bond0.200       7ea38r1d-1l45-4029-as46-9aloa4c12a51  vlan      bond0.200
```
Vamos a comprobar primero que tenemos tráfico de la VLAN 100 para esto lo podemos hacer de dos formas

**Lanzando un TCPDUMP sobre la interfaz bond0. Para esto es necesario especificar la VLAN**
```plaintext
[root@PRUEBAS01 ~]# tcpdump -i bond0 -n -e '(vlan 2222)'
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on bond0, link-type EN10MB (Ethernet), capture size 262144 bytes
^C
0 packets captured
86 packets received by filter
54 packets dropped by kernel
```
0 paquetes capturados. No hay tráfico.

Ahora vamos a consultar si hay tráfico para la VLAN 200. En este caso lo haremos directamente sobre la interfaz de la VLAN. No será necesario especificar la VLAN.
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
	   
Aquí sí vemos tráfico capturado.

Con esta evidencia podemos confirmar que hay problemas con la VLAN 100. La VLAN 200 está correctamente pasada.
