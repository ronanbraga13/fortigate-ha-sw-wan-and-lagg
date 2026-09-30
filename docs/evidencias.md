# Origem das evidências e critérios de publicação

## Fontes do laboratório

A documentação foi reconstruída a partir da conversa “Próximo LAB de failover”, dos comandos fornecidos pelo autor, das configurações discutidas e dos resultados ali registrados em 30/09/2026.

| Informação | Base disponível |
|---|---|
| FortiOS 7.2.8 build 1639 nos dois membros | Saídas textuais de get system status |
| HA A-P, grupo MTZ-HA, heartbeat port4/port5 | Configuração e saídas textuais do HA |
| Prioridades 200 e 100 | Configuração do primeiro nó e bloco do segundo nó |
| Sincronização e checksums compatíveis | Saída textual após reinicialização dos dois membros |
| VLANs e portas do SW_MATRIZ | Running-config e registro da inclusão/validação do segundo trunk |
| Switches WAN | Blocos de configuração e sequência de execução registrada |
| Dois pacotes perdidos em cada teste | Resultados das capturas transcritos na conversa e confirmados pelo autor para esta publicação |
| Primeiro failover e eleição | Saída textual do HA, além do registro do teste |
| Retorno do primeiro membro e segundo failover | Descrição das capturas na conversa |

Os excertos de ping são transcrições resumidas, não arquivos de captura bruta. Imagens da conversa não foram republicadas nesta versão. Não foram produzidos prints sintéticos, logs fictícios ou configurações completas inferidas.

## Sanitização

Foram publicados somente os trechos necessários para explicar o laboratório. A senha de HA foi substituída por um marcador; valores ENC, chaves pré-compartilhadas, credenciais administrativas e números de série foram excluídos. Nenhum backup integral ou histórico bruto da conversa acompanha o repositório.

Os endereços privados citados pertencem ao cenário de laboratório e ajudam a identificar o teste. Não há endereços públicos ou dados de acesso publicados.

## Organização inicial do repositório

Antes da escrita, a consulta ao GitHub confirmou repositório vazio, sem branches e sem arquivos. A publicação cria uma estrutura inicial de README e documentação temática, sem substituir conteúdo existente.

## O que esta publicação não reconstrói

Não há configuração integral de SD-WAN, BGP ou IPsec na fonte recuperada. Por isso, ASNs, prefixos anunciados, regras de seleção, parâmetros criptográficos e políticas não foram inventados. O portfólio documenta a integração e o HA realizado, mas ainda não constitui um pacote completo de reprodução de todo o ambiente.

[Voltar ao início](../README.md)
