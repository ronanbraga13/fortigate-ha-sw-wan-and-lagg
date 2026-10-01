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

## SW_WAN_LAGG — WAN do Rio por VLAN

O switch mantém Claro e Vivo em VLANs separadas, entregues por trunks aos dois FortiGates. Não há port-channel nem LACP/802.3ad.

```text
configure terminal
hostname SW_WAN_LAGG
vlan 100
 name WAN_CLARO
vlan 200
 name WAN_VIVO
interface Ethernet0/0
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast edge
 no shutdown
interface Ethernet0/1
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast edge
 no shutdown
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 100,200
 no shutdown
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 100,200
 no shutdown
end
write memory
```

| Comando | Função |
|---|---|
| `configure terminal` | Entra no modo de configuração global |
| `hostname SW_WAN_LAGG` | Define o nome do switch |
| `vlan 100` / `vlan 200` | Cria ou seleciona a VLAN de cada provedor |
| `name WAN_CLARO` / `name WAN_VIVO` | Identifica a VLAN pelo provedor |
| `interface Ethernet0/x` | Seleciona a porta a configurar |
| `switchport mode access` | Define a porta do roteador como access |
| `switchport access vlan 100` / `200` | Associa os quadros sem tag à VLAN do provedor |
| `spanning-tree portfast edge` | Acelera a entrada em forwarding das portas access ligadas aos roteadores |
| `switchport trunk encapsulation dot1q` | Seleciona encapsulamento IEEE 802.1Q |
| `switchport mode trunk` | Habilita o transporte das VLANs por tags |
| `switchport trunk allowed vlan 100,200` | Restringe a trunk às duas VLANs WAN |
| `no shutdown` | Habilita administrativamente a interface |
| `end` | Sai do modo de configuração |
| `write memory` | Salva a configuração para persistir após reinicialização |

### Interfaces WAN no FortiGate do Rio

```text
config system interface
    edit "port1"
        set alias "AGG_WAN"
        set ip 0.0.0.0 0.0.0.0
    next
    edit "WAN_CLARO"
        set interface "port1"
        set vlanid 100
        set ip 172.16.10.6 255.255.255.252
    next
    edit "WAN_VIVO"
        set interface "port1"
        set vlanid 200
        set ip 172.16.20.6 255.255.255.252
    next
end
```

`config system interface` abre a configuração das interfaces; `edit` seleciona ou cria a interface; `set alias` define o rótulo; `set interface` vincula a subinterface à porta física; `set vlanid` define a tag 802.1Q; `set ip` define endereço e máscara (zero na interface pai); `next` encerra a edição de cada interface e `end` fecha o bloco.

### Conferência dos switches

| Comando | Função |
|---|---|
| `show vlan brief` | Lista VLANs e portas access |
| `show interfaces status` | Exibe o estado das portas |
| `show interfaces trunk` | Confere trunks e VLANs permitidas |
| `show spanning-tree interface Ethernet1/0` | Confere o estado STP da trunk LAN |
| `show mac address-table` | Confere o aprendizado de MAC por porta/VLAN |

No SW_WAN_LAGG, conferir Et0/0 na VLAN 100, Et0/1 na VLAN 200 e Et0/2/Et0/3 como trunks permitindo 100,200.

[Voltar ao início](../README.md)
