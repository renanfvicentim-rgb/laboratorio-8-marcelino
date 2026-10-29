# Atividade 08 – Análise de malware (análise estática)

> **Professor Esp. Marcelino Dias da Silva Junior** · UNINOVE · Disciplina: **Auditoria Forense de Sistemas Digitais**
> **Aula relacionada:** Análise de malwares (reforça: Introdução ao pensamento hacker)

> 🛡️ **Amostra didática e inofensiva.** O arquivo em `amostra/` foi criado para a disciplina: o conteúdo "escondido" contém apenas comandos `echo` que **descrevem** o que um malware faria. Mesmo assim, siga a regra de ouro: **nunca execute uma amostra**. Esta atividade é de análise **estática**.

## 🎯 Objetivo
Fazer a **triagem estática** de um arquivo suspeito recebido por e-mail: identificar o tipo real, comparar o hash com os **IOCs**, extrair indicadores e encontrar a **persistência** na máquina da vítima.

## 📚 Conceito em 1 minuto
- **Análise estática:** examinar o arquivo **sem executá-lo** (tipo, hash, strings, código).
- **Análise dinâmica:** executar num ambiente isolado (sandbox) e observar. **Não** faremos aqui.
- **IOC (indicador de comprometimento):** hash, domínio, IP ou arquivo que identifica um ataque conhecido.
- **Persistência:** como o malware volta a rodar após reiniciar. No Windows usa chaves `Run` do Registro. No Linux usa `crontab`, `.bashrc` e serviços.

## 🧪 Passo a passo

```bash
cd /workspaces/laboratorio-8-marcelino
mkdir -p saida
ls -l amostra/
```

**1. Tipo real e extensão dupla.** É mesmo um PDF?

```bash
file amostra/fatura_agosto.pdf.sh
```

📌 A **extensão dupla** `.pdf.sh` engana quem vê só o começo do nome. É um **script**, não um PDF.

**2. Hash e comparação com os IOCs**

```bash
sha256sum amostra/fatura_agosto.pdf.sh
cat iocs.txt
grep -F "$(sha256sum amostra/fatura_agosto.pdf.sh | cut -d' ' -f1)" iocs.txt
```

**3. Strings e indicadores de rede**

```bash
strings amostra/fatura_agosto.pdf.sh | head -n 12
grep -o -E 'https?://[^" ]+' amostra/fatura_agosto.pdf.sh
grep -o -E '([0-9]{1,3}\.){3}[0-9]{1,3}' amostra/fatura_agosto.pdf.sh
```

**4. Decodificando o conteúdo ofuscado (SEM executar)**

O texto longo na variável `P` está em **Base64**, uma forma de esconder o conteúdo de quem lê. Decodifique para um arquivo `.txt` e **leia**:

```bash
grep '^P=' amostra/fatura_agosto.pdf.sh | cut -d'"' -f2 | base64 -d > saida/etapa2_decodificada.txt
cat saida/etapa2_decodificada.txt
```

**5. Persistência na máquina da vítima**

```bash
cat amostra/crontab_vitima.txt
cat amostra/bashrc_vitima.txt
grep -n -E "/tmp/|@reboot|nohup" amostra/crontab_vitima.txt amostra/bashrc_vitima.txt
```

**6. Confronte tudo com os IOCs**

```bash
for ioc in $(grep -v '^#' iocs.txt | cut -d';' -f2); do
  echo "== IOC: $ioc"; grep -l -F "$ioc" amostra/* saida/* 2>/dev/null
done
```

## 📝 Tabela de triagem (resolvida)

> **Nota:** os resultados abaixo correspondem aos fixtures didaticos deste repositorio, criados para exercitar a analise estatica. O dominio e o IP sao reservados para documentacao e nao representam infraestrutura real. As referencias de persistencia estao comentadas e nao ativam comandos.

| Item | Resultado |
|---|---|
| Nome / extensão aparente | `fatura_agosto.pdf.sh`; parece uma fatura PDF, mas a extensao final e `.sh`. |
| Tipo real (`file`) | POSIX shell script, texto ASCII; nao e um PDF. |
| SHA-256 | `5ee66ad3701c57b97ccfa8bb88c7c9ffe32d961d0d1e63242d4f58caa4b6a34a` |
| Consta na lista de IOCs? | Sim. O hash corresponde a entrada em `iocs.txt`. |
| Domínio / IP de C2 | `c2.exemplo.invalid` / `192.0.2.25` (valores reservados para documentacao/testes). |
| Ações descritas na 2ª etapa | Mensagens simulam uma tentativa de baixar arquivo sem conexao e criar persistencia sem alterar o sistema. |
| Mecanismos de persistência | Cron (`@reboot`) e inicializacao via `.bashrc` (`nohup`), somente como comentarios ilustrativos nos arquivos de evidencia. |

## ❓ Perguntas
1. **Por que o atacante usou a extensão `.pdf.sh`?** Para fazer o arquivo parecer uma fatura PDF e aumentar a chance de alguem abri-lo. A extensao final revela que e um script shell; a extensao dupla nao o transforma em PDF.
2. **O que é Base64? É criptografia? Por que ele foi usado aqui?** Base64 e uma codificacao de dados em texto, nao criptografia: nao usa chave nem garante confidencialidade. E facil de reverter e pode ser usada para ofuscar o conteudo de uma leitura casual.
3. **Quais são os dois mecanismos de persistência encontrados? Qual o equivalente no Windows?** Cron por meio de `@reboot` e inicializacao de comandos pelo `.bashrc` (aqui representada por `nohup`). Nos fixtures, aparecem apenas como comentarios, sem persistencia ativa. O material cita as chaves `Run` do Registro do Windows como equivalente.
4. **Por que o perito não executa a amostra, nem "só para ver"?** A execucao poderia alterar ou apagar dados, iniciar comunicacao externa, instalar persistencia ou comprometer a maquina, alem de contaminar evidencias. A analise solicitada e estatica, portanto a amostra deve ser inspecionada sem execucao.
5. **Que recomendações você faria à empresa (usuário, e-mail, estação)?** Orientar usuarios a desconfiar de anexos inesperados e confirmar pedidos por outro canal; filtrar e bloquear anexos de script; manter sistemas e protecoes atualizados; aplicar menor privilegio, manter backups testados e monitorar pontos de inicializacao como cron e `.bashrc`. Se houver suspeita de abertura, acionar a equipe responsavel e isolar a estacao conforme o plano de resposta a incidentes.
