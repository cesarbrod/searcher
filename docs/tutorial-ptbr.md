# Tutorial do searcher para iniciantes no Linux

Este tutorial ensina como usar o **searcher**, um pequeno programa que encontra
arquivos de documentos no seu computador **pelo nome do arquivo ou pelo que
está escrito dentro deles**. Não é preciso saber Linux de antemão: cada passo
é explicado, e todos os comandos podem ser copiados e colados.

Ao final, você será capaz de:

- rodar o `searcher` a partir do terminal,
- encontrar arquivos pelo nome e pelo conteúdo,
- combinar palavras de busca com `AND`, `OR` e `"frases exatas"`,
- limitar a busca a PDFs, arquivos do Word e outros formatos,
- entender como o `searcher` difere de `find`, `locate` e `grep`.

---

## 1. O que é o searcher?

O `searcher` é um programa de arquivo único. Você informa a ele:

1. **onde procurar** — uma pasta (ou um único arquivo), e
2. **o que procurar** — uma ou mais palavras, ou uma frase exata.

Ele responde com uma lista dos arquivos encontrados, mais uma pequena prévia
das linhas correspondentes. Ele entende arquivos de texto simples (`txt`,
`md`, `csv`, …) e documentos de verdade: **PDF**, **Word** (`docx`),
**OpenDocument** (`odt`), **RTF**, **ePub**, **PowerPoint** (`pptx`) e
**Excel** (`xlsx`).

Exemplo de como é um resultado:

```text
searcher: content:'"quarterly results"' under /home/ana/docs (recursive)
────────────────────────────────────────────────────────────────────────
  [1] report_2024.pdf
    │ ...*quarterly results* grew 12 percent...

────────────────────────────────────────────────────────────────────────
1 match(es)
```

## 2. Em que o searcher difere de find, locate e grep?

O Linux já tem ferramentas clássicas de busca. Cada uma é boa em algo, e o
`searcher` preenche a lacuna que elas deixam. Aqui vai a versão resumida:

| Tarefa | `find` | `locate` | `grep` | `searcher` |
|---|---|---|---|---|
| Achar arquivos **pelo nome** | sim | sim (instantâneo) | não | sim |
| Achar arquivos **pelo conteúdo** | só junto com o `grep` | não | sim (arquivos de texto) | sim |
| Ler dentro de arquivos **PDF / Word** | não | não | não | sim |
| `"frases exatas"`, `AND`, `OR` | não (exige truques de regex) | não | parcial (regex) | sim, sintaxe simples |
| Precisa de banco de dados de índice | não (varredura ao vivo) | sim (atualizado ~1x ao dia) | não (varredura ao vivo) | não (varredura ao vivo) |
| Mostra trechos de prévia | não | não | sim (`-C`) | sim |

Em palavras simples:

- **`find`** percorre pastas e combina *nomes* de arquivos (ou datas,
  tamanhos, …). Ele não sabe nada sobre o *conteúdo* dos arquivos. Para olhar
  dentro dos arquivos, é preciso combiná-lo com o `grep`, o que logo fica
  complicado.
- **`locate`** é muito rápido porque pesquisa uma lista pronta de nomes de
  arquivos em vez do seu disco. Desvantagens: ele só conhece nomes (nunca
  conteúdos), e a lista dele em geral é reconstruída uma vez por dia — então
  arquivos criados há uma hora podem não aparecer.
- **`grep`** (muito usado como `grep -r`) pesquisa *dentro* dos arquivos e é
  excelente para texto simples e código. Mas ele não consegue ler arquivos PDF
  ou Word — nesses casos ele só diz "Binary file matches" (ou nada útil), e
  não tem lógica simples de `AND` / `OR` / frases.
- **`searcher`** é a ferramenta do tipo "me diga quais documentos mencionam
  X": busca ao vivo (sem banco de dados desatualizado), formatos reais de
  documentos e uma linguagem de busca feita de palavras comuns em vez de
  expressões regulares.

Quando usar cada um:

- "Onde salvei aquele arquivo chamado invoice?" → `locate invoice` ou
  `find ~ -name "*invoice*"` (rápido, simples).
- "Quais arquivos de código contêm tal nome de função?" →
  `grep -rn "def foo" .` (o `grep` continua imbatível para código-fonte).
- "Quais dos meus PDFs e docs do Word mencionam quarterly results?" →
  `searcher`. Nem `find`, nem `locate`, nem `grep` fazem isso direito.

## 3. Abrindo um terminal e fazendo a primeira busca

Na maioria dos sistemas Linux, pressione `Ctrl + Alt + T` para abrir um
terminal. Você verá algo como `ana@pc:~$`. O `~` significa a sua pasta pessoal
(home).

Suponha que você baixou o `searcher` para `~/searcher` (a pasta do projeto).
Primeiro, entre nela e torne o programa executável (isso só se faz uma vez):

```bash
cd ~/searcher
chmod +x searcher
./searcher --help
```

Observações para iniciantes:

- `cd` significa "change directory" (entrar em uma pasta).
- `chmod +x` significa "permitir que este arquivo seja executado como programa".
- O `./` na frente significa "execute o programa que está *nesta* pasta". O
  Linux não executa programas da pasta atual a menos que você peça com `./`
  — isso é normal, não é erro.
- Se aparecer `command not found`, provavelmente você esqueceu o `./`.
- Se aparecer `Permission denied`, rode de novo a linha `chmod +x searcher`.

O `./searcher --help` mostra o manual completo. Experimente agora — tudo
neste tutorial também está resumido lá.

> **Dica:** para rodar o `searcher` de qualquer lugar sem `cd`, coloque-o no
> seu `PATH` uma vez: `ln -s "$PWD/searcher" ~/.local/bin/searcher`. Depois
> disso, basta digitar `searcher` em qualquer pasta. (Saia e entre de novo se
> o terminal não o encontrar de imediato.)

## 4. As primeiras buscas

O formato básico é:

```bash
./searcher [ONDE] [O QUE] [opções]
```

- `ONDE` é uma pasta (pesquisada incluindo as subpastas) ou um único arquivo.
  Se você omitir, a pasta atual (`.`) é usada.
- `O QUE` é o que você procura. Sozinho, significa **pesquisar dentro do
  conteúdo dos arquivos**.

Teste estes (troque `~/docs` por uma pasta que você realmente tenha):

```bash
# Quais arquivos mencionam "annual report"? (as duas palavras, em qualquer lugar do arquivo)
./searcher ~/docs "annual report"

# Pesquisar na pasta atual, em vez disso
./searcher . "annual report"

# Só contar os resultados (útil em scripts)
./searcher ~/docs "annual report" --count
```

Enquanto ele roda, você verá uma linha de status: primeiro quantos arquivos
foram encontrados, depois uma barra de progresso com o arquivo sendo lido e,
no fim, um resumo como `searcher: 3 match(es) in 120 file(s) (0.8s)`. Isso vai
para um canal separado (stderr), então nunca suja a saída usada em
redirecionamentos (pipes). Adicione `-q` (ou `--quiet`) para desligar, ex.:
`./searcher ~/docs "x" --count -q`.

## 5. Pesquisando pelo nome do arquivo

Adicione `-n` (ou `--name`) para comparar com os nomes dos arquivos em vez do
conteúdo:

```bash
# Arquivos cujo NOME contém "report" ou "invoice"
./searcher ~/docs --name "report OR invoice"

# Nomes seguem a mesma lógica de palavras do conteúdo (veja a próxima seção)
./searcher ~/docs --name "2024 budget"
```

Dá até para combinar os dois: o nome precisa combinar com X **e** o conteúdo
precisa combinar com Y:

```bash
# Arquivos chamados como "invoice" cujo conteúdo menciona "paid" ou "overdue"
./searcher ~/docs -n invoice -s "paid OR overdue"
```

(`-s` / `--content` deixa explícito que é busca de conteúdo.)

## 6. A linguagem de busca: AND, OR e frases exatas

Este é o coração do `searcher`. As mesmas regras valem para buscas por nome e
por conteúdo. A comparação ignora maiúsculas/minúsculas, a menos que você
passe `--case-sensitive`.

| Você digita | Você obtém |
|---|---|
| `apple` | arquivos contendo `apple` |
| `apple banana` | arquivos contendo `apple` **E** `banana` (em qualquer lugar, em qualquer ordem) |
| `apple OR banana` | arquivos contendo `apple` **ou** `banana` (pelo menos um) |
| `"apple pie"` | arquivos contendo a frase exata `apple pie` |
| `"apple pie" "banana bread"` | arquivos contendo **ambas** as frases exatas |
| `"apple pie" OR "banana bread"` | arquivos contendo **pelo menos uma** das frases |

### Aspas no terminal (importante!)

O terminal (shell) também usa aspas, então você precisa embrulhar sua busca
em aspas para ela chegar inteira ao programa. Regras práticas:

- **Sempre envolva a busca em aspas duplas**: `"annual report"`.
- **Frases exatas precisam de aspas simples por fora e duplas por dentro**:
  `'"quarterly results"'`. (Aspas duplas por fora fariam o shell "comer" as
  de dentro.)
- A palavra `OR` precisa estar em maiúsculas para funcionar como "ou". Para
  pesquisar a palavra literal "or", coloque-a entre aspas: `'"or"'`.

Exemplos para copiar:

```bash
./searcher ~/docs "budget forecast"                        # AND (E)
./searcher ~/docs "budget OR forecast"                     # OR (OU)
./searcher ~/docs '"quarterly results"'                    # frase exata
./searcher ~/docs '"quarterly results" "annual summary"'   # ambas as frases
./searcher ~/docs '"quarterly results" OR "annual summary"' # qualquer uma das frases
```

## 7. Limitando quais arquivos são pesquisados

Por padrão, a busca de conteúdo olha arquivos tipo documento (`txt`, `md`,
`pdf`, `docx`, `odt`, `rtf`, `epub`, `pptx`, `xlsx`, …) e pula código-fonte.
Você pode mudar isso:

```bash
# Só arquivos PDF
./searcher ~/docs "contract" --ext pdf

# Vários formatos (separados por vírgula, com ou sem ponto)
./searcher ~/docs "contract" --ext pdf,docx,txt

# Incluir também código-fonte e scripts
./searcher ~/projects "TODO" --all-text

# Ver a lista completa de extensões suportadas
./searcher --list-exts
```

Outras opções úteis de alcance:

```bash
./searcher ~/docs "x" --no-recursive   # só a pasta atual, sem subpastas
./searcher ~/docs "x" --hidden         # incluir também arquivos/pastas ocultos
./searcher ~/docs "x" --max-size 10    # pular arquivos maiores que 10 MB
./searcher ~/docs "X" --case-sensitive # "X" não combina mais com "x"
./searcher ~/docs "x" --absolute       # mostrar caminhos completos em vez de curtos
```

## 8. Lendo os resultados

Cada acerto mostra o arquivo mais até 2 linhas de prévia, com as palavras
encontradas marcadas `*assim*`:

```bash
./searcher ~/docs "warranty" --lines 5    # mostra 5 linhas de prévia por arquivo
./searcher ~/docs "warranty" --lines 0    # sem prévias, só nomes de arquivos
./searcher ~/docs "warranty" --limit 5    # mostra só os 5 primeiros acertos
./searcher ~/docs "warranty" --count      # mostra só o número, ex.: 14
```

Códigos de saída (úteis ao combinar com outros comandos): `0` = achou algo,
`1` = nada encontrado, `2` = erro (ex.: pasta não existe).

## 9. As ferramentas clássicas, lado a lado

As mesmas tarefas, com ferramentas diferentes. Rode estes para sentir a
diferença (usando uma pasta `~/docs` com um arquivo `notes.txt` contendo
"call Ana about the warranty", ou seja, "ligar para Ana sobre a garantia"):

```bash
# --- pelo NOME ---
find ~/docs -type f -name "*warranty*"     # varredura ao vivo, padrão glob
locate warranty                            # instantâneo, mas pode perder arquivos novos
./searcher ~/docs --name warranty          # varredura ao vivo, lógica de palavras

# --- pelo CONTEÚDO (texto simples) ---
grep -ri "warranty" ~/docs                 # clássico, mostra cada linha correspondente
grep -rli "warranty" ~/docs                # -l: só nomes de arquivos
grep -rn -C 2 "warranty" ~/docs            # -n: números das linhas, -C 2: 2 linhas de contexto
./searcher ~/docs "warranty"               # nomes de arquivos + prévias curtas

# --- pelo CONTEÚDO, duas palavras em qualquer lugar (AND/E) ---
grep -ril "warranty" ~/docs | xargs grep -li "ana"   # jeito desajeitado em duas etapas
./searcher ~/docs "warranty ana"                     # o mesmo, direto

# --- dentro de PDFs: só o searcher funciona ---
grep -ri "warranty" ~/docs                 # "Binary file report.pdf matches" — inútil
./searcher ~/docs "warranty" --ext pdf     # lê de fato o texto do PDF
```

Uma nota sobre o `locate`: se ele nunca acha seus arquivos novos, o banco de
dados dele está desatualizado. Atualizá-lo (`sudo updatedb`) exige permissão
de administrador — mais um motivo pelo qual a busca ao vivo do `searcher` é
conveniente.

## 10. Receitas: tarefas comuns

```bash
# Todos os PDFs que mencionam "LinkedIn"
./searcher ~/Documents "LinkedIn" --ext pdf

# Faturas (pelo nome) que mencionam "overdue" (em atraso)
./searcher ~/docs -n invoice -s overdue

# Quantas notas de reunião mencionam "budget" ou "forecast"?
./searcher ~/notes "budget OR forecast" --count

# Mensagem exata de erro em vários manuais (qualquer formato)
./searcher ~/manuals '"paper jam in tray 2"'

# Tudo que menciona o cliente, só os 10 primeiros (limitando o alcance)
./searcher ~/clients/acme "acme" --ext pdf,docx --limit 10

# Pesquisar um único arquivo
./searcher ~/docs/handbook.pdf "parental leave"

# Ver POR QUE alguns arquivos foram pulados
./searcher ~/docs "x" --errors
```

## 11. Solução de problemas

**`command not found`** — você esqueceu o `./` (rode `./searcher`, não
`searcher`), a menos que o tenha instalado no seu `PATH` (veja a seção 3).

**`Permission denied`** — rode `chmod +x searcher` uma vez.

**Sem resultados, mas o arquivo com certeza contém a palavra** — confira três
coisas:
1. A extensão do arquivo está no conjunto pesquisado?
   (`./searcher --list-exts`). Código-fonte precisa de `--all-text`;
   qualquer outra coisa precisa de `--ext`.
2. É um arquivo `.doc` antigo (não `.docx`)? Esses são pulados com um aviso —
   converta para `.docx` primeiro (ex.: com o LibreOffice).
3. É um PDF *escaneado* (fotos de páginas, sem texto de verdade)? Nenhuma
   ferramenta de busca de texto lê esses sem um programa de OCR.

**Avisos estranhos de PDF** — você não deveria ver nenhum; o `searcher`
silencia a tagarelice do analisador de PDF e pula arquivos corrompidos em
silêncio. Adicione `--errors` para listar quais arquivos foram pulados e por
quê.

**"PDF fallback extractor in use"** — instale o `pypdf` uma vez
(`pip install pypdf`) para uma extração de texto de PDFs bem melhor.

**Busca lenta** — estreite o alcance: `--ext pdf,docx`, `--max-size 20`,
`--no-recursive`, ou aponte para uma pasta menor. Árvores muito grandes são
varridas arquivo por arquivo (um de cada vez), como no `grep -r`.

## 12. Cola rápida

```bash
./searcher . "words"                    # conteúdo, pasta atual, recursivo
./searcher ~/docs "a b"                 # AND (E)
./searcher ~/docs "a OR b"              # OR (OU)
./searcher ~/docs '"exact phrase"'      # frase exata
./searcher ~/docs --name "invoice"      # pelo nome do arquivo
./searcher ~/docs -n inv -s "paid"      # nome E conteúdo
./searcher ~/docs "x" --ext pdf,docx    # só estes formatos
./searcher ~/docs "x" --count           # só o número
./searcher ~/docs "x" -q                # sem barra de status
./searcher --help                       # manual completo
```

Boa busca!
