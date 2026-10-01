# Arquitetura do laboratório

## Desenho implementado

A Matriz utiliza dois FortiGates em HA (High Availability) Active-Passive. Ambos alcançam as mesmas VLANs internas e os mesmos segmentos Ethernet dos provedores. O membro ativo encaminha o tráfego; o outro fica disponível para assumir.

O SW_WAN_CLARO atende exclusivamente ao segmento da Claro e o SW_WAN_VIVO ao segmento da Vivo. São switches Layer 2 separados, com portas access na VLAN 1 padrão, sem IP configurado para encaminhamento. A VLAN 1 de um switch não é interligada à do outro: os provedores continuam em domínios distintos.

O SW_MATRIZ entrega as VLANs (Virtual Local Area Networks) 10, 20 e 30 aos dois firewalls por trunks 802.1Q.

## Mapeamento dos switches

| Switch | Interface | Conexão / função |
|---|---|---|
| SW_MATRIZ | Ethernet0/0 | Trunk para FGT_MTZ_01; VLANs 10,20,30 |
| SW_MATRIZ | Ethernet1/0 | Trunk para FGT_MTZ_02; VLANs 10,20,30 |
| SW_MATRIZ | Ethernet0/1 | Access VLAN 10 |
| SW_MATRIZ | Ethernet0/2 | Access VLAN 20 |
| SW_MATRIZ | Ethernet0/3 | Access VLAN 30 |
| SW_WAN_CLARO | Ethernet0/0 | FGT_MTZ_01, lado Claro |
| SW_WAN_CLARO | Ethernet0/1 | FGT_MTZ_02, lado Claro |
| SW_WAN_CLARO | Ethernet0/2 | Roteador Claro |
| SW_WAN_VIVO | Ethernet0/0 | FGT_MTZ_01, lado Vivo |
| SW_WAN_VIVO | Ethernet0/1 | FGT_MTZ_02, lado Vivo |
| SW_WAN_VIVO | Ethernet0/2 | Roteador Vivo |

## Heartbeat e ambiente virtual

- FortiOS port4 do FGT_MTZ_01 ↔ port4 do FGT_MTZ_02.
- FortiOS port5 do FGT_MTZ_01 ↔ port5 do FGT_MTZ_02.
- No template usado no PNETLab, a interface de gerenciamento ocupa a primeira posição: as portas exibidas como 5 e 6 no desenho correspondem a port4 e port5 no FortiOS. Esse deslocamento é específico do template.

As duas VMs utilizam FortiOS 7.2.8 build 1639, 2 CPUs e aproximadamente 987 MB de memória apresentados pelo equipamento. Esses são dados do laboratório, não um dimensionamento recomendado para produção.

## Dual WAN, SD-WAN, BGP e IPsec

O HA foi integrado ao ambiente já existente, mantendo os dois provedores e os túneis CLARO_TO_RJ e VIVO_TO_RJ. Os túneis CLARO_TO_RJ e VIVO_TO_RJ estavam UP antes dos testes.

| Elemento | Função |
|---|---|
| Dual WAN | Acesso por Claro e Vivo, com switches dedicados por provedor |
| SD-WAN | Seleção de caminhos entre os provedores |
| BGP | Roteamento dinâmico entre os sites |
| IPsec | Comunicação Matriz ↔ Rio pelos túneis CLARO_TO_RJ e VIVO_TO_RJ |

## Limites da redundância

A falha de um switch WAN remove o acesso ao respectivo provedor para os dois firewalls. O outro provedor possui infraestrutura Layer 2 separada, mas a troca automática de caminho dependerá das regras, rotas e verificações configuradas; esse cenário não foi testado nesta etapa.

O SW_MATRIZ é um ponto único de falha da LAN. HA de firewall não elimina falhas comuns de switch, energia ou hipervisor.

## Filial Rio de Janeiro

![Topologia da filial Rio de Janeiro](topologia-rio.png)

O Rio utiliza o cluster Active-Passive FGT_RIO_DE_JANEIRO_01 / FGT_RIO_DE_JANEIRO_02. O SW_RIO_01 entrega as VLANs 10, 20 e 30 por duas trunks independentes: Et0/0 para o FGT01 e Et1/0 para o FGT02. Esses enlaces não formam LAG.

| Switch | Interface | Conexão / função |
|---|---|---|
| SW_RIO_01 | Et0/0 | Trunk VLANs 10,20,30 para FGT01 |
| SW_RIO_01 | Et1/0 | Trunk VLANs 10,20,30 para FGT02 |
| SW_WAN_LAGG | Et0/0 | Access VLAN 100 para Claro |
| SW_WAN_LAGG | Et0/1 | Access VLAN 200 para Vivo |
| SW_WAN_LAGG | Et0/2 | Trunk 802.1Q VLANs 100,200 para FGT01 |
| SW_WAN_LAGG | Et0/3 | Trunk 802.1Q VLANs 100,200 para FGT02 |

No FortiOS, a interface física `port1`, com alias `AGG_WAN`, permanece sem IP e serve de interface pai para `WAN_CLARO` (VLAN 100, 172.16.10.6/30) e `WAN_VIVO` (VLAN 200, 172.16.20.6/30). Apesar do hostname SW_WAN_LAGG e do alias AGG_WAN, a implementação consolida duas WANs em uma interface física por VLAN 802.1Q; não utiliza LACP/802.3ad.

Os rótulos dos FortiGates na topologia seguem o template PNETLab, com deslocamento pela MGMT: port2 no desenho corresponde a FortiOS port1 (WAN), port4 a port3 (LAN), e port5/port6 a port4/port5 (heartbeat). A configuração utiliza os nomes do FortiOS.

Após a migração, a comunicação Rio ↔ Matriz por IPsec/BGP foi restabelecida. Na Matriz, foram preservados os túneis CLARO_TO_RJ e VIVO_TO_RJ.

| BGP no Rio | Valor / estado |
|---|---|
| AS local | 65000 |
| Router ID | 1.1.1.10 |
| Vizinho 1.1.1.2 | Remote AS 65001; Established |
| Vizinho 2.2.2.2 | Remote AS 65001; Established |

O SW_RIO_01 é um ponto único de falha da LAN; o SW_WAN_LAGG é compartilhado pelos dois provedores e pelos dois membros do HA. A separação por VLAN mantém os domínios Layer 2 distintos, mas não elimina a falha comum do switch WAN.

[Voltar ao início](../README.md)
