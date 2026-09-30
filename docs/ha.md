# Configuração do HA

## Parâmetros finais

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

Os nomes FGT_MTZ_01 e FGT_MTZ_02 identificam os nós na documentação. Nos registros, também aparecem os hostnames FW_MATRIZ e FortiOS-VM64-KVM.

## FGT_MTZ_01

Trecho sanitizado da configuração relevante do primeiro membro:

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

Trecho consolidado da configuração do segundo membro, com os estados finais explicitados:

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

O marcador de senha não é uma credencial utilizável: substitua-o localmente pelo mesmo segredo nos dois membros. Os blocos são excertos para documentação, não substituem a preparação e a validação do cluster.

## Decisões do laboratório

O grupo utiliza FGCP (FortiGate Clustering Protocol) e dois enlaces dedicados de heartbeat. A formação e a sincronização devem ser verificadas antes de executar falhas controladas.

Com override desabilitado, a maior prioridade não significa que o FGT_MTZ_01 será sempre Primary. No teste, ele retornou como Secondary e o FGT_MTZ_02 continuou ativo.

Session pickup permaneceu desabilitado. A validação apresentada é de recuperação da conectividade ICMP, não de preservação de sessões de aplicações.

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

Houve divergência de sincronização associada à tabela dlp.data-type. Após reinicialização dos dois membros, o registro mostrou ambos in-sync e checksums gerais idênticos. Não foi feita alteração manual dos objetos DLP (Data Loss Prevention). O registro permite descrever a recuperação observada, mas não atribuir causa raiz ou confirmar um bug.

## Referência

[Fortinet — HA Active-Passive no FortiOS 7.2.8](https://docs.fortinet.com/document/fortigate/7.2.8/administration-guide/900885/ha-active-passive-cluster-setup).

[Voltar ao início](../README.md)
