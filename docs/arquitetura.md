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

Os nomes das interfaces LAN/WAN correspondentes no FortiOS não foram inventados. Confirme-os na topologia e na configuração antes de reproduzir o cabeamento.

## Heartbeat e ambiente virtual

- FortiOS port4 do FGT_MTZ_01 ↔ port4 do FGT_MTZ_02.
- FortiOS port5 do FGT_MTZ_01 ↔ port5 do FGT_MTZ_02.
- No template usado no PNETLab, a interface de gerenciamento ocupa a primeira posição: as portas exibidas como 5 e 6 no desenho correspondem a port4 e port5 no FortiOS. Esse deslocamento é específico do template.

Ambas as VMs foram verificadas com FortiOS 7.2.8 build 1639, 2 CPUs e aproximadamente 987 MB de memória apresentados pelo equipamento. Esses são dados do laboratório, não um dimensionamento recomendado para produção.

## Dual WAN, SD-WAN, BGP e IPsec

O HA foi integrado ao ambiente já existente, mantendo os dois provedores e os túneis CLARO_TO_RJ e VIVO_TO_RJ. O registro anterior ao failover descreve ambos os túneis como UP.

| Elemento | Papel no laboratório | Detalhamento disponível |
|---|---|---|
| Dual WAN | Dois caminhos de provedor | Claro e Vivo via switches dedicados |
| SD-WAN | Seleção de caminhos no ambiente existente | Sem exportação das regras e dos health checks nesta fonte |
| BGP | Roteamento dinâmico do ambiente existente | Sem saída de vizinhança, ASNs, prefixos e route-maps nesta fonte |
| IPsec | Comunicação entre Matriz e Rio | Nomes dos dois túneis e registro de estado UP |


## Limites da redundância

A falha de um switch WAN remove o acesso ao respectivo provedor para os dois firewalls. O outro provedor possui infraestrutura Layer 2 separada, mas a troca automática de caminho dependerá das regras, rotas e verificações configuradas; esse cenário não foi testado nesta etapa.

O SW_MATRIZ é um ponto único de falha da LAN. HA de firewall não elimina falhas comuns de switch, energia ou hipervisor. O HA do Rio não foi validado nesta etapa.

[Voltar ao início](../README.md)
