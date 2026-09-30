# Testes de failover controlado

**Data do laboratório:** 30/09/2026.  
**Resultado principal:** 2 pacotes ICMP (Internet Control Message Protocol) perdidos em cada teste.

## Condição inicial e método

O cluster da Matriz estava sincronizado, com FGT_MTZ_01 Primary e FGT_MTZ_02 Secondary. Os dois enlaces de heartbeat estavam ativos. O registro do laboratório descreve os túneis CLARO_TO_RJ e VIVO_TO_RJ como UP antes do teste.

Foi mantido um ping contínuo de um host da Matriz (10.0.20.1) para o endereço 172.16.10.6, identificado no laboratório como destino do lado Rio:

```text
ping 172.16.10.6 -c 10000
```

O teste consistiu em desligar somente o firewall ativo no PNETLab, observar os timeouts e confirmar a eleição no outro membro com get system ha status. O ping para 8.8.8.8 foi considerado uma observação adicional; não foi usado como prova principal porque a interface de gerenciamento também participava do roteamento para a Internet.

## Teste 1 — Desligamento do FGT_MTZ_01

| Etapa | Observação |
|---|---|
| Antes | FGT_MTZ_01 Primary; FGT_MTZ_02 Secondary |
| Falha | FGT_MTZ_01 desligado |
| Eleição | FGT_MTZ_02 assumiu como único membro disponível |
| ICMP | Timeout nas sequências 144 e 145 |
| Recuperação | Resposta retomada na sequência 146 |

Transcrição resumida do resultado registrado na conversa:

```text
seq=143 -> resposta
seq=144 -> timeout
seq=145 -> timeout
seq=146 -> resposta
seq=147 -> resposta
```

O log textual do HA registrou a perda do primeiro membro às 16:17:20 e a seleção do outro como Primary às 16:17:21, no relógio do equipamento. Esse intervalo de registro não mede sozinho a indisponibilidade do tráfego.

Enquanto o primeiro membro estava desligado, o HA indicou o membro perdido. Após seu retorno, o registro mostrou FGT_MTZ_02 Primary e FGT_MTZ_01 Secondary, ambos Synchronized. A prioridade 200 do primeiro nó não provocou retomada automática do papel ativo nesse teste.

## Teste 2 — Desligamento do FGT_MTZ_02

Com o FGT_MTZ_01 reintegrado e sincronizado, foi desligado o FGT_MTZ_02, então Primary.

| Etapa | Observação |
|---|---|
| Antes | FGT_MTZ_02 Primary; FGT_MTZ_01 Secondary |
| Falha | FGT_MTZ_02 desligado |
| Eleição registrada | FGT_MTZ_01 assumiu |
| ICMP | Timeout nas sequências 378 e 379 |
| Recuperação | Resposta retomada na sequência 380 |

Transcrição resumida do resultado registrado na conversa:

```text
seq=377 -> resposta
seq=378 -> timeout
seq=379 -> timeout
seq=380 -> resposta
```

## Interpretação dos resultados

A conectividade observada foi restabelecida nos dois sentidos de troca do membro ativo, com perda de 2 pacotes por teste. Não há base para transformar esse número em um tempo exato ou em garantia de desempenho: seriam necessários intervalo de envio e temporização completos da captura.

Session pickup estava desabilitado. Não foram realizados testes de continuidade de sessões TCP (Transmission Control Protocol), nem medições separadas de reconvergência BGP, renegociação IPsec ou seleção de caminho SD-WAN.

O destino do ping coincide com o endereço associado ao túnel Claro no registro. Sem tabela de rotas, seletores, contadores ou captura de pacotes, o ping isolado não comprova que o fluxo passou dentro do IPsec. A evidência de túneis UP é complementar; a documentação não apresenta esse ping como teste conclusivo de payload criptografado.

## Evidências adicionais para uma próxima rodada

- Capturar rotas e contadores IPsec antes e depois, usando também um host da LAN do Rio como destino.
- Registrar estado e tempo de recuperação das vizinhanças BGP.
- Testar tráfego TCP de longa duração.
- Testar separadamente falha de provedor, switch WAN e um enlace de heartbeat.
- Registrar o retorno e a sincronização dos dois membros após o segundo teste.

Esses itens são extensões propostas, não resultados já obtidos.

[Voltar ao início](../README.md)
