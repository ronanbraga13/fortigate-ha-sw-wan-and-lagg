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

## Etapa pendente

A implementação de HA no Rio de Janeiro será a próxima etapa do laboratório.

[Voltar ao início](../README.md)
