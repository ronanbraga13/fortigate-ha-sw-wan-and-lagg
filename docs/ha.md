# Configuração do HA

## Parâmetros do cluster

| Parâmetro | FGT_MTZ_01 | FGT_MTZ_02 |
|---|---|---|
| Grupo | MTZ-HA / ID 0 | MTZ-HA / ID 0 |
| Modo | Active-Passive | Active-Passive |
| Heartbeat | port4 e port5 | port4 e port5 |
| Prioridade | 200 | 100 |
| Sincronização de configuração | Habilitada | Habilitada |
| Override | Desabilitado | Desabilitado |
| Session pickup | Desabilitado | Desabilitado |
| Interface reservada de gerenciamento | Desabilitada | Desabilitada |
| Monitoramento de interfaces pelo HA | Não configurado | Não configurado |

## FGT_MTZ_01

Configuração de HA do primeiro membro:

```text
config system ha
    set group-id 0
    set group-name "MTZ-HA"
    set mode a-p
    set password "<DEFINIR_SEGREDO_HA_LOCALMENTE>"
    set hbdev "port4" 0 "port5" 0
    set sync-config enable
    set override disable
    set priority 200
    set session-pickup disable
    set ha-mgmt-status disable
    unset monitor
end
```

## FGT_MTZ_02

Configuração de HA do segundo membro:

```text
config system ha
    set group-id 0
    set group-name "MTZ-HA"
    set mode a-p
    set password "<DEFINIR_SEGREDO_HA_LOCALMENTE>"
    set hbdev "port4" 0 "port5" 0
    set sync-config enable
    set override disable
    set priority 100
    set session-pickup disable
    set ha-mgmt-status disable
    unset monitor
end
```

Defina a senha do HA localmente, usando o mesmo valor nos dois membros.

## Funcionamento do cluster

O grupo utiliza FGCP (FortiGate Clustering Protocol) e dois enlaces dedicados de heartbeat. A formação e a sincronização devem ser verificadas antes de executar falhas controladas.

Com override desabilitado, a maior prioridade não significa que o FGT_MTZ_01 será sempre Primary. No teste, ele retornou como Secondary e o FGT_MTZ_02 continuou ativo.

Com `session-pickup disable`, o cluster não sincroniza sessões para sua continuidade após o failover. Os testes avaliaram a recuperação da conectividade ICMP.

A interface de gerenciamento já participava de roteamento e de objetos VIP (Virtual IP) do laboratório; por isso não foi convertida em interface reservada de HA. Não foi configurado monitoramento de interfaces para disparar failover por perda de enlace; os testes desligaram o nó ativo.

## Validação

No membro ativo:

```text
get system status
get system ha status
diagnose sys ha checksum cluster
```

Verificar versão/build, dois membros presentes, papéis Primary/Secondary, ambos in-sync, checksums compatíveis e port4/port5 ativas.

O acesso ao outro membro pode ser feito com:

```text
execute ha manage <INDICE_DO_MEMBRO> <USUARIO_ADMINISTRATIVO>
```

Obtenha o índice atual em get system ha status; ele não deve ser presumido após uma eleição.

## Ocorrência durante a montagem

Durante a montagem, os checksums da tabela `dlp.data-type` divergiram entre os membros. Após reiniciar os dois firewalls, ambos ficaram `in-sync`, com checksums gerais idênticos. Os objetos DLP não foram alterados manualmente.

## Referência

[Fortinet — HA Active-Passive no FortiOS 7.2.8](https://docs.fortinet.com/document/fortigate/7.2.8/administration-guide/900885/ha-active-passive-cluster-setup).

[Voltar ao início](../README.md)
