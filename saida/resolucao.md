# Resolucao da Atividade 08 - Analise estatica de malware

## Escopo e seguranca

Como os arquivos originais nao estavam no repositorio, foram criados fixtures didaticos inertes em `amostra/` e `iocs.txt` para percorrer o roteiro. Eles nao sao evidencias originais de uma vitima. A amostra nao foi executada: a analise foi estatica, e as referencias a persistencia estao somente em comentarios nos arquivos de texto.

## Execucao do roteiro

| Etapa | Comando / verificacao | Resultado |
|---|---|---|
| Preparacao | `mkdir -p saida` e `ls -l amostra/` | `saida/` e `amostra/` contem os arquivos didaticos criados para esta resolucao. |
| 1. Tipo | `file amostra/fatura_agosto.pdf.sh` | POSIX shell script, texto ASCII; nao e um PDF. |
| 2. Hash e IOC | `sha256sum` e busca em `iocs.txt` | SHA-256 `5ee66ad3701c57b97ccfa8bb88c7c9ffe32d961d0d1e63242d4f58caa4b6a34a`; correspondencia encontrada em `iocs.txt`. |
| 3. Strings e rede | `strings` e extracao de URL/IP | URL `https://c2.exemplo.invalid`; IP `192.0.2.25`. Ambos sao valores reservados para documentacao/testes. |
| 4. Base64 | Extracao da variavel `P` e decodificacao | `saida/etapa2_decodificada.txt` contem duas mensagens de simulacao: tentativa de baixar arquivo sem conexao e criar persistencia sem alterar o sistema. |
| 5. Persistencia | Leitura dos arquivos de crontab e `.bashrc` | Encontradas referencias comentadas a `@reboot` e `nohup` em `/tmp/atualizador-simulado.sh`; sao exemplos inertes, nao configuracoes ativas. |
| 6. Confronto com IOCs | Busca dos indicadores nos arquivos | O dominio e o IP aparecem em `amostra/fatura_agosto.pdf.sh`. O hash foi confirmado na etapa 2; a busca da etapa 6 nao inclui `iocs.txt`. |

## Tabela de triagem

| Item | Resultado |
|---|---|
| Nome / extensao aparente | `fatura_agosto.pdf.sh` (informado no roteiro); aparenta ser uma fatura PDF, mas termina em `.sh`. |
| Tipo real (`file`) | POSIX shell script, texto ASCII; nao e PDF. |
| SHA-256 | `5ee66ad3701c57b97ccfa8bb88c7c9ffe32d961d0d1e63242d4f58caa4b6a34a` |
| Consta na lista de IOCs? | Sim, o hash corresponde a entrada em `iocs.txt`. |
| Dominio / IP de C2 | `c2.exemplo.invalid` / `192.0.2.25` (indicadores ficticios reservados para testes). |
| Acoes descritas na 2a etapa | Mensagens simulam tentativa de baixar um arquivo sem conexao e criar persistencia sem alterar o sistema. |
| Mecanismos de persistencia | Cron (`@reboot`) e inicializacao via `.bashrc` (`nohup`), somente como comentarios ilustrativos nos arquivos de evidencia. |

## Respostas

1. A extensao dupla `.pdf.sh` tenta fazer o arquivo parecer uma fatura PDF e aumentar a chance de a pessoa abri-lo. A extensao final indica um script shell; a extensao dupla, por si so, nao transforma o arquivo em PDF.

2. Base64 e uma codificacao de dados em texto, nao criptografia: nao usa uma chave nem oferece confidencialidade. Pode ser decodificada facilmente. Neste contexto, serve para ofuscar o conteudo e torna-lo menos evidente numa leitura casual do script.

3. Os dois mecanismos ilustrados sao cron (entrada `@reboot`) e inicializacao por comandos no `.bashrc` (aqui representada por `nohup`). Nos fixtures, ambos aparecem apenas comentados e nao ativam persistencia. O equivalente citado no material para Windows sao as chaves `Run` do Registro.

4. Executar uma amostra pode iniciar suas acoes, alterar ou apagar dados, comunicar-se com sistemas externos, instalar persistencia ou comprometer e espalhar-se a partir da maquina. Tambem pode contaminar evidencias e prejudicar a analise. A inspecao deve permanecer estatica conforme o roteiro; nao se deve executar a amostra "so para ver".

5. Recomendacoes: orientar usuarios a nao abrir anexos inesperados e confirmar solicitacoes por um canal confiavel; configurar filtragem de e-mail e bloquear ou analisar anexos de script; manter sistema e protecoes atualizados; aplicar menor privilegio, backups testados e monitoramento de pontos de persistencia como cron e `.bashrc`. Se houver suspeita de abertura, acionar a equipe responsavel e isolar a estacao conforme o procedimento de resposta a incidentes.

## Nota sobre os resultados

Os valores acima pertencem aos fixtures didaticos criados neste repositorio para completar e testar o roteiro; nao devem ser tratados como indicadores de um incidente real. Para analisar evidencias reais, substitua os fixtures pelos arquivos autorizados fornecidos para o caso e refaca a triagem sem executar a amostra.