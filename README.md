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
cd /workspaces/*/labs/08-analise-de-malware
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

## 📝 Tabela de triagem (preencha)

| Item | Resultado |
|---|---|
| Nome / extensão aparente | |
| Tipo real (`file`) | |
| SHA-256 | |
| Consta na lista de IOCs? | |
| Domínio / IP de C2 | |
| Ações descritas na 2ª etapa | |
| Mecanismos de persistência | |

## ❓ Perguntas
1. Por que o atacante usou a extensão `.pdf.sh`?
2. O que é Base64? É criptografia? Por que ele foi usado aqui?
3. Quais são os dois mecanismos de persistência encontrados? Qual o equivalente no Windows?
4. Por que o perito **não** executa a amostra, nem "só para ver"?
5. Que recomendações você faria à empresa (usuário, e-mail, estação)?
