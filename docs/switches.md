# Configurações relevantes dos switches

Configurações dos trunks da LAN e das portas dos switches WAN.

## SW_MATRIZ

As VLANs 10, 20 e 30 já existiam. A alteração central foi adicionar o trunk do segundo FortiGate em Ethernet1/0, preservando o trunk original e as portas access.

```text
hostname SW_MATRIZ
!
interface Ethernet0/0
 description TRUNK_FW_MATRIZ
 switchport trunk allowed vlan 10,20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 description VLAN10_MATRIZ
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast edge
!
interface Ethernet0/2
 description VLAN20_MATRIZ
 switchport access vlan 20
 switchport mode access
 spanning-tree portfast edge
!
interface Ethernet0/3
 description VLAN30_MATRIZ
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast edge
!
interface Ethernet1/0
 description TRUNK_FW_MATRIZ_HA02
 switchport trunk allowed vlan 10,20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
```

Os trunks Ethernet0/0 e Ethernet1/0 encaminham as VLANs 10, 20 e 30. Após a ativação do segundo trunk, a porta entrou em forwarding com a convergência do STP.

## SW_WAN_CLARO

```text
configure terminal
hostname SW_WAN_CLARO
interface Ethernet0/0
 description WAN_CLARO_FGT_MTZ_01
 switchport mode access
 spanning-tree portfast edge
 no shutdown
interface Ethernet0/1
 description WAN_CLARO_FGT_MTZ_02
 switchport mode access
 spanning-tree portfast edge
 no shutdown
interface Ethernet0/2
 description WAN_CLARO_ISP
 switchport mode access
 spanning-tree portfast edge
 no shutdown
end
write memory
```

## SW_WAN_VIVO

```text
configure terminal
hostname SW_WAN_VIVO
interface Ethernet0/0
 description WAN_VIVO_FGT_MTZ_01
 switchport mode access
 spanning-tree portfast edge
 no shutdown
interface Ethernet0/1
 description WAN_VIVO_FGT_MTZ_02
 switchport mode access
 spanning-tree portfast edge
 no shutdown
interface Ethernet0/2
 description WAN_VIVO_ISP
 switchport mode access
 spanning-tree portfast edge
 no shutdown
end
write memory
```

## Por que os switches WAN não têm IP?

Cada switch apenas transporta quadros Ethernet entre o roteador do provedor e os dois membros do cluster. As três portas permanecem na VLAN 1 padrão, pois nenhuma VLAN access diferente foi definida. Endereço IP seria necessário para gerenciamento do switch, não para esse encaminhamento Layer 2.

PortFast edge foi usado nas ligações aos equipamentos finais deste desenho. Não se deve estender esse ajuste indiscriminadamente a enlaces entre switches.

## Conferência após a configuração

```text
show vlan brief
show interfaces status
show interfaces trunk
show spanning-tree interface Ethernet1/0
show mac address-table
```

No SW_MATRIZ, conferir VLANs existentes, os dois trunks e estado forwarding. Nos switches WAN, conferir as três portas access no mesmo segmento de cada provedor e o aprendizado de MAC (Media Access Control).

## SW_RIO_01 — trunks LAN

Et0/0 para FGT01 e Et1/0 para FGT02 transportam VLANs 10,20,30 como trunks independentes, sem agregação de enlaces. Em cada trunk, `switchport trunk encapsulation dot1q` seleciona 802.1Q, `switchport mode trunk` define o modo e `switchport trunk allowed vlan 10,20,30` limita as VLANs transportadas.

## SW_WAN_LAGG — LACP e VLANs WAN do Rio

O Po1 reúne Et0/2 e Et1/0 para o firewall 01. O Po2 reúne Et0/3 e Et1/1 para o firewall 02. Ambos transportam VLAN 100 (Claro) e VLAN 200 (Vivo). Cada agregado conecta duas portas de um único FortiGate.

Trechos de referência da configuração validada:

```text
vlan 100
 name WAN_CLARO
vlan 200
 name WAN_VIVO
interface Ethernet0/0
 switchport access vlan 100
 switchport mode access
 spanning-tree portfast edge
interface Ethernet0/1
 switchport access vlan 200
 switchport mode access
 spanning-tree portfast edge
interface range Ethernet0/2, Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 100,200
 channel-group 1 mode active
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 100,200
interface range Ethernet0/3, Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 100,200
 channel-group 2 mode active
interface Port-channel2
 description LACP_FGT_RIO_INFERIOR
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 100,200
```

### Agregado e interfaces WAN no FortiGate do Rio

```text
config system interface
    edit "AGG_WAN"
        set vdom "root"
        set type aggregate
        set member "port1" "port2"
        set lacp-speed fast
    next
    edit "WAN_CLARO"
        set vdom "root"
        set interface "AGG_WAN"
        set vlanid 100
        set ip 172.16.10.6 255.255.255.252
        set allowaccess ping
        set role wan
    next
    edit "WAN_VIVO"
        set vdom "root"
        set interface "AGG_WAN"
        set vlanid 200
        set ip 172.16.20.6 255.255.255.252
        set allowaccess ping
        set role wan
    next
end
```

Esses recortes documentam as interfaces do LAB; não são backups completos. A configuração do cluster é sincronizada entre os membros.

### Conferência

```text
show etherchannel summary
show lacp neighbor
show interfaces trunk
```

No FortiGate:

```text
diagnose netlink aggregate name AGG_WAN
get system ha status
```

Na validação, Po1 e Po2 ficaram `SU`, com os membros em `P`. As VLANs 100 e 200 estavam permitidas e em forwarding nos dois Port-channels. Veja as [capturas de LAG e failover](lag-failover-rio.md).

[Voltar ao início](../README.md)