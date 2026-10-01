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

[Voltar ao início](../README.md)
