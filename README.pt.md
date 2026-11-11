# grep Cheatsheet · Folha de referência de comandos grep

> 🌍 [中文](README.md) · [Español](README.es.md)

Uma **folha de referência de uma única página** com 30 usos reais do `grep`: as opções de busca do dia a dia; recursão e filtros de arquivos; os três sabores de regex e quanto da linha precisa coincidir; contexto e formatação da saída; e o comportamento em pipelines do qual os scripts dependem (códigos de saída, arquivos binários, registros NUL, fluxos ao vivo). Cada entrada responde apenas três perguntas: **o que faz, um comando pronto para copiar e colar, e etiquetas de uso**. Sem semântica inventada: as 30 entradas foram **executadas de verdade**, uma a uma, contra uma árvore de arquivos de teste (dois logs, notas, fontes em C e Python, um binário, um arquivo com separador NUL, links simbólicos, um nome com espaços) no GNU grep 3.11, e a semântica foi conferida no manual do GNU Grep. As diferenças entre GNU e BSD estão assinaladas no fim da folha.

## Para quem é

Engenheiros de operações (SRE/DevOps), desenvolvedores backend e qualquer pessoa que tenha um log de milhões de linhas e só queira saber quais falharam, ou que precise saber exatamente o que o `grep` retorna dentro de um script. Se as suas buscas sempre chegam atrasadas —a recursão entra onde não deve, um binário só responde «matches», a saída de um pipeline não aparece—, esta folha é para você.

## Como usar

1. Vá direto ao **[CHEATSHEET.md](CHEATSHEET.md)**: o corpo da referência; os comandos em inglês qualquer desenvolvedor entende.
2. Identifique primeiro o seu grep: `grep --version` diz GNU grep e a versão na primeira linha; o macOS traz o grep do BSD, que não tem `-P`. As 30 entradas estão verificadas no GNU grep 3.11 e as notas de implementação no final são leitura obrigada.
3. Siga a ordem da folha —conceitos básicos → padrões → contexto e saída → pipelines e scripts— e teste cada comando à mão sobre arquivos de prática. Em uma hora você domina.

## Experimente em 30 segundos

```bash
grep --version | head -1               # confirma a implementação: GNU grep e versão
mkdir -p /tmp/proj && printf 'INFO ok\nERROR disk full\n' > /tmp/proj/app.log
grep -c 'ERROR' /tmp/proj/app.log      # 1: a contagem de linhas que coincidem, o gesto clássico do grep
```

## A combinação de ouro, pronta para usar

```bash
grep -rn --include='*.log' 'ERROR' proj
```

Recursão + números de linha + só arquivos de log: todas as linhas ERROR de todos os logs da árvore, com arquivo e linha, sem tocar em fontes nem binários. O primeiro comando de qualquer postmortem costuma ser este.

## O que inclui

| Categoria | Conteúdo |
|---|---|
| 🧱 Conceitos básicos | busca simples / -i ignorando maiúsculas / -v inverter / -c contar / -n números de linha / -l e -L listas de arquivos / -r recursão / -R seguindo links / --include, --exclude e --exclude-dir (9) |
| 🎯 Padrões | -E regex estendida e a armadilha das barras no BRE / -F texto literal / -w palavras inteiras / -x linha inteira / -e vários padrões / -f arquivo de padrões / âncoras ^ e $ (7) |
| 🖍️ Contexto e saída | -A depois / -B antes / -C ambos e a fusão de grupos / --color / -o só a parte coincidente / -H e -h o prefixo de arquivo (6) |
| 🔧 Pipelines e scripts | -q e o contrato de saída 0/1/2 / -s silenciar erros / -m parar cedo / o trio binário (aviso, -a, -I) / -z registros NUL e coincidências entre linhas / --line-buffered / -P PCRE com lookarounds / padrões que começam com - (8) |

A documentação oficial vinculada é apenas https://www.gnu.org/software/grep/manual/; todos os comandos foram conferidos no manual do GNU Grep e executados com sucesso no GNU grep 3.11; os exemplos operam somente sobre arquivos de teste criados na hora, sem dados reais.

## Por que estas 30

- **O contrato de códigos de saída tinha de estar**: 0 = há coincidência, 1 = não há, 2 = erro. Metade dos acidentes com grep em scripts vem de tratar o 1 como se fosse erro, ou de não esperar que `-c` imprima 0 saindo com código 1. A folha fixa isso por escrito e mede.
- **-r e -R diferem por uma letra e muito em comportamento**: `-r` só segue os links simbólicos nomeados na linha de comando e pula os que encontra dentro da árvore; `-R` segue todos. Sem saber disso, uma busca recursiva omite arquivos em silêncio.
- **O «falso negativo» dos binários, assinalado**: a linha coincidente é suprimida e só fica um aviso no stderr (`binary file matches`); quem só olha o stdout acha que não havia nada. `-a` força a linha e `-I` pula o arquivo: as duas saídas corretas estão dadas.
- **-z tem um requisito invisível para cruzar linhas**: a quebra de linha do padrão precisa ser real (aspas `$'…'` do bash); um `\n` entre aspas normais não faz nada. Medido antes de escrever.

## Pull Requests são bem-vindos

Um comando incorreto, uma opção obsoleta, uma tradução a melhorar? Pull Requests são bem-vindos. Mantenha o formato das entradas (uma linha de explicação / bloco bash / etiquetas) e **apenas comandos reais e verificados; na dúvida, deixe de fora**.

## 📚 Mais folhas de referência

Outras folhas do mesmo autor, também em chinês / espanhol / português:

- [Folha de SSH](https://github.com/Ezra-Zhao/ssh-cheatsheet) — 28 comandos de conexão, chaves, túneis e endurecimento
- [Folha de FFmpeg](https://github.com/Ezra-Zhao/ffmpeg-cheatsheet) — 28 comandos de áudio e vídeo
- [Folha de Git](https://github.com/Ezra-Zhao/git-cheatsheet) — 30 comandos do dia a dia e de resgate
- [Folha de Docker](https://github.com/Ezra-Zhao/docker-cheatsheet) — 28 comandos de contêineres
- [Folha de curl](https://github.com/Ezra-Zhao/curl-cheatsheet) — 29 comandos para depurar APIs
- [Folha de expressões regulares](https://github.com/Ezra-Zhao/regex-cheatsheet) — 30 padrões práticos
- [Folha de jq](https://github.com/Ezra-Zhao/jq-cheatsheet) — 30 filtros para JSON
- [Folha de superpoderes de terminal](https://github.com/Ezra-Zhao/terminal-superpowers-cheatsheet) — 14 ferramentas de produtividade
- [Folha de find + xargs](https://github.com/Ezra-Zhao/find-xargs-cheatsheet) — 30 comandos para buscar arquivos e agir em lote
- [Folha de awk](https://github.com/Ezra-Zhao/awk-cheatsheet) — 30 comandos de campos, totais e junção de dois arquivos
- [Folha de kubectl](https://github.com/Ezra-Zhao/kubectl-cheatsheet) — comandos habituais do Kubernetes
- [Folha de SQL](https://github.com/Ezra-Zhao/sql-cheatsheet) — consultas e funções habituais
- [Folha de Vim](https://github.com/Ezra-Zhao/vim-cheatsheet) — 30 comandos de edição, busca e macros
- [Folha de tmux](https://github.com/Ezra-Zhao/tmux-cheatsheet) — 30 comandos de sessões, janelas e painéis
- [Folha de systemd](https://github.com/Ezra-Zhao/systemd-cheatsheet) — 30 comandos de serviços e registros

## Licença

MIT © 2026 Guangyi Zhao. Veja [LICENSE](LICENSE).
