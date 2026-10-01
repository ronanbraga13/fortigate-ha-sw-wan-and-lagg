# FortiGate HA + Dual WAN + SD-WAN + BGP + IPsec

Implementação de FortiGate HA Active-Passive na Matriz, integrado a Dual WAN, SD-WAN, BGP e VPN IPsec. O laboratório valida sincronização do cluster e failover automático entre os firewalls, mantendo a comunicação entre Matriz e Rio de Janeiro.

**Status: EM CONSTRUÇÃO.** O HA da Matriz está implementado e validado. A próxima etapa é implementar o HA no site Rio de Janeiro.

**Resultado dos testes:** 2 pacotes ICMP perdidos em cada um dos dois testes de desligamento do firewall ativo, com retomada das respostas após a eleição do outro membro.

## Visão geral

Laboratório virtual no PNETLab com dois FortiGates na Matriz, dois provedores (Claro e Vivo), switches WAN dedicados e comunicação com o Rio de Janeiro. O HA foi acrescentado ao ambiente de Dual WAN, SD-WAN, BGP e IPsec já utilizado no laboratório.

| Componente | Implementação |
|---|---|
| Firewalls da Matriz | FGT_MTZ_01 e FGT_MTZ_02; FortiOS-VM64-KVM 7.2.8 build 1639 |
| HA (High Availability) | Active-Passive com FGCP (FortiGate Clustering Protocol) |
| Heartbeat | Dois enlaces diretos: port4 ↔ port4 e port5 ↔ port5 |
| LAN (Local Area Network) | SW_MATRIZ com trunks 802.1Q para VLANs 10, 20 e 30 |
| Dual WAN (Wide Area Network) | Claro e Vivo, cada uma com seu próprio switch Layer 2 |
| SD-WAN (Software-Defined Wide Area Network) | Seleção de caminhos entre Claro e Vivo |
| BGP (Border Gateway Protocol) | Roteamento dinâmico entre os sites |
| VPN (Virtual Private Network) IPsec (Internet Protocol Security) | Túneis CLARO_TO_RJ e VIVO_TO_RJ |
| Validação | Sincronização, eleição, retorno do membro e failover nos dois sentidos |

## Topologia

Diagrama lógico; os rótulos de portas detalhados estão em [Arquitetura](docs/arquitetura.md).

```mermaid
flowchart TB
    C["Provedor Claro"] --- SC["SW_WAN_CLARO"]
    V["Provedor Vivo"] --- SV["SW_WAN_VIVO"]
    SC --- F1["FGT_MTZ_01"]
    SC --- F2["FGT_MTZ_02"]
    SV --- F1
    SV --- F2
    F1 <-->|"Heartbeat port4 + port5"| F2
    F1 ---|"Trunk VLANs 10,20,30"| L["SW_MATRIZ"]
    F2 ---|"Trunk VLANs 10,20,30"| L
    C -.->|"Caminho IPsec Claro"| RJ["Rio de Janeiro"]
    V -.->|"Caminho IPsec Vivo"| RJ
```

## Documentação e configurações

- [Arquitetura e mapeamento de portas](docs/arquitetura.md)
- [Configuração do HA e validação do cluster](docs/ha.md)
- [Configurações dos switches](docs/switches.md)
- [Testes e resultados de failover](docs/failover.md)
- [Validações do laboratório](docs/evidencias.md)

## Resultados

| Teste | Ação controlada | Novo Primary | Perda ICMP | Resultado |
|---|---|---|---|---|
| 1 | Desligamento do FGT_MTZ_01 ativo | FGT_MTZ_02 | 2 pacotes | Respostas retomadas |
| 2 | Desligamento do FGT_MTZ_02 ativo | FGT_MTZ_01 | 2 pacotes | Respostas retomadas |

Após o primeiro teste, o FGT_MTZ_01 retornou como Secondary, enquanto o FGT_MTZ_02 permaneceu Primary; ambos ficaram sincronizados.

Os dois testes restabeleceram a comunicação Matriz ↔ Rio de Janeiro, com perda de 2 pacotes ICMP em cada failover. Após o retorno, o membro foi reintegrado e sincronizado com o cluster. Os detalhes estão em [Testes de failover](docs/failover.md).

## Próxima etapa

Implementar o HA no Rio de Janeiro e validar o failover do site. Essa etapa permanece pendente.

## Configurações

Os arquivos apresentam os trechos de configuração do HA e dos switches usados no laboratório. A senha do HA deve ser definida localmente, com o mesmo valor nos dois membros.

## Referências técnicas

- [Fortinet — HA Active-Passive no FortiOS 7.2.8](https://docs.fortinet.com/document/fortigate/7.2.8/administration-guide/900885/ha-active-passive-cluster-setup)
- [Fortinet — FGCP](https://docs.fortinet.com/document/fortigate/7.2.8/administration-guide/62403/fgcp)
- [Fortinet — Failover protection](https://docs.fortinet.com/document/fortigate/7.2.8/administration-guide/489324)

