# Validações do laboratório

O HA da Matriz foi validado com desligamento do firewall ativo nos dois sentidos.

| Validação | Resultado |
|---|---|
| FortiOS | Versão 7.2.8 build 1639 nos dois membros |
| Cluster | FGCP Active-Passive, grupo MTZ-HA |
| Heartbeat | port4 e port5 ativos |
| Sincronização | Dois membros in-sync, com checksums gerais idênticos |
| LAN | Trunks do SW_MATRIZ encaminhando as VLANs 10, 20 e 30 |
| WAN | Switch dedicado para cada provedor: Claro e Vivo |
| IPsec antes dos testes | CLARO_TO_RJ e VIVO_TO_RJ UP |
| Failover de FGT_MTZ_01 para FGT_MTZ_02 | 2 pacotes ICMP perdidos; comunicação Matriz ↔ Rio restabelecida |
| Failover de FGT_MTZ_02 para FGT_MTZ_01 | 2 pacotes ICMP perdidos; comunicação Matriz ↔ Rio restabelecida |
| Retorno do membro | Reintegração e sincronização com o cluster |

Os procedimentos e resultados estão em [Testes de failover](failover.md). Os comandos de conferência do cluster estão em [Configuração do HA](ha.md).

## Validações da filial Rio de Janeiro

| Validação | Resultado |
|---|---|
| Cluster | Active-Passive; prioridades 200/100; override disable |
| Heartbeat | FortiOS port4 e port5 |
| Sincronização | Ambos os membros Synchronized após reintegração |
| LAN | Duas trunks independentes no SW_RIO_01, VLANs 10,20,30 |
| WAN Claro | Gateway 172.16.10.5: 5/5 respostas, 0% de perda |
| WAN Vivo | Gateway 172.16.20.5: 5/5 respostas, 0% de perda |
| IPsec/BGP após migração | Comunicação Rio ↔ Matriz restabelecida |
| BGP | AS local 65000; Router ID 1.1.1.10 |
| Vizinhos BGP | 1.1.1.2 e 2.2.2.2, Remote AS 65001, ambos Established |
| Failover FGT01 → FGT02 | 2 pacotes ICMP perdidos; comunicação restabelecida |
| Retorno do FGT01 | Secondary sincronizado, sem preempção |
| Failover FGT02 → FGT01 | 2 pacotes ICMP perdidos; comunicação restabelecida |
| Alteração com FGT02 desligado | Retorno Not Synchronized → Synchronized automaticamente |

### Comandos de validação WAN e BGP

```text
execute ping-options repeat-count 5
execute ping 172.16.10.5
execute ping 172.16.20.5
get router info bgp summary
```

`execute ping-options repeat-count 5` define cinco solicitações por teste; `execute ping` verifica a resposta de cada gateway. `get router info bgp summary` apresenta o AS local, Router ID e o estado dos vizinhos.

Os testes HA e a ressincronização estão detalhados em [Testes de failover](failover.md).

[Voltar ao início](../README.md)
