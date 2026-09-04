# Tutoriel searcher pour débutants sous Linux

Ce tutoriel vous apprend à utiliser **searcher**, un petit programme qui
retrouve des fichiers de documents sur votre ordinateur **par nom de fichier
ou par ce qui est écrit à l'intérieur**. Aucune expérience préalable de Linux
n'est requise : chaque étape est expliquée, et toutes les commandes peuvent
être copiées et collées.

À la fin, vous saurez :

- lancer `searcher` depuis le terminal,
- retrouver des fichiers par nom et par contenu,
- combiner des mots de recherche avec `AND`, `OR` et des `"phrases exactes"`,
- limiter la recherche aux PDF, aux fichiers Word et aux autres formats,
- comprendre en quoi `searcher` diffère de `find`, `locate` et `grep`.

---

## 1. Qu'est-ce que searcher ?

`searcher` est un programme en un seul fichier. Vous lui donnez :

1. **où chercher** — un dossier (ou un seul fichier), et
2. **quoi chercher** — un ou plusieurs mots, ou une phrase exacte.

Il répond avec la liste des fichiers correspondants, plus un court aperçu des
lignes concernées. Il comprend les fichiers en texte brut (`txt`, `md`,
`csv`, …) et les vrais documents : **PDF**, **Word** (`docx`),
**OpenDocument** (`odt`), **RTF**, **ePub**, **PowerPoint** (`pptx`) et
**Excel** (`xlsx`).

Exemple de résultat :

```text
searcher: content:'"quarterly results"' under /home/ana/docs (recursive)
────────────────────────────────────────────────────────────────────────
  [1] report_2024.pdf
    │ ...*quarterly results* grew 12 percent...

────────────────────────────────────────────────────────────────────────
1 match(es)
```

## 2. En quoi searcher diffère-t-il de find, locate et grep ?

Linux possède déjà des outils de recherche classiques. Chacun excelle dans un
domaine, et `searcher` comble le vide qu'ils laissent. Voici la version
courte :

| Tâche | `find` | `locate` | `grep` | `searcher` |
|---|---|---|---|---|
| Retrouver des fichiers **par nom** | oui | oui (instantané) | non | oui |
| Retrouver des fichiers **par contenu** | seulement avec `grep` | non | oui (fichiers texte) | oui |
| Lire à l'intérieur des fichiers **PDF / Word** | non | non | non | oui |
| `"phrases exactes"`, `AND`, `OR` | non (astuces regex requises) | non | partiel (regex) | oui, syntaxe simple |
| Nécessite une base d'index | non (parcours en direct) | oui (reconstruite ~1 fois/jour) | non (parcours en direct) | non (parcours en direct) |
| Affiche des extraits d'aperçu | non | non | oui (`-C`) | oui |

En mots simples :

- **`find`** parcourt les dossiers et compare les *noms* de fichiers (ou les
  dates, tailles, …). Il ne sait rien du *contenu* des fichiers. Pour regarder
  à l'intérieur, il faut le combiner avec `grep`, ce qui devient vite
  laborieux.
- **`locate`** est très rapide car il cherche dans une liste de noms de
  fichiers préparée à l'avance plutôt que sur votre disque. Inconvénients : il
  ne connaît que les noms (jamais les contenus), et sa liste est en général
  reconstruite une fois par jour — les fichiers créés il y a une heure
  peuvent donc manquer.
- **`grep`** (souvent utilisé comme `grep -r`) cherche *à l'intérieur* des
  fichiers et est excellent pour le texte brut et le code. Mais il ne peut pas
  lire les fichiers PDF ou Word — dans ces cas il dit seulement "Binary file
  matches" (ou rien d'utile), et n'a pas de logique simple `AND` / `OR` /
  phrases.
- **`searcher`** est l'outil du type « dites-moi quels documents mentionnent
  X » : recherche en direct (pas de base obsolète), vrais formats de
  documents, et un langage de recherche fait de mots courants au lieu
  d'expressions régulières.

Quand utiliser chacun :

- « Où ai-je enregistré ce fichier appelé invoice ? » → `locate invoice` ou
  `find ~ -name "*invoice*"` (rapide, simple).
- « Quels fichiers de code contiennent tel nom de fonction ? » →
  `grep -rn "def foo" .` (`grep` reste imbattable pour le code source).
- « Lesquels de mes PDF et docs Word mentionnent quarterly results ? » →
  `searcher`. Ni `find`, ni `locate`, ni `grep` ne font cela correctement.

## 3. Ouvrir un terminal et lancer sa première recherche

Sur la plupart des systèmes Linux, appuyez sur `Ctrl + Alt + T` pour ouvrir
un terminal. Vous verrez quelque chose comme `ana@pc:~$`. Le `~` désigne
votre dossier personnel (home).

Supposez que vous avez téléchargé `searcher` dans `~/searcher` (le dossier du
projet). D'abord, allez-y et rendez le programme exécutable (une seule fois) :

```bash
cd ~/searcher
chmod +x searcher
./searcher --help
```

Notes pour débutants :

- `cd` signifie « change directory » (entrer dans un dossier).
- `chmod +x` signifie « autoriser ce fichier à s'exécuter comme programme ».
- Le `./` devant signifie « exécute le programme qui est dans *ce*
  dossier ». Linux n'exécute pas les programmes du dossier courant sauf si
  vous le demandez avec `./` — c'est normal, pas une erreur.
- Si vous voyez `command not found`, vous avez probablement oublié le `./`.
- Si vous voyez `Permission denied`, relancez la ligne `chmod +x searcher`.

`./searcher --help` affiche le manuel complet. Essayez maintenant — tout ce
tutoriel y est aussi résumé.

> **Astuce :** pour lancer `searcher` depuis n'importe où sans `cd`,
> placez-le dans votre `PATH` une fois :
> `ln -s "$PWD/searcher" ~/.local/bin/searcher`. Ensuite, il suffit de taper
> `searcher` dans n'importe quel dossier. (Déconnectez-vous et reconnectez-vous
> si le terminal ne le trouve pas aussitôt.)

## 4. Vos premières recherches

La forme de base est :

```bash
./searcher [OÙ] [QUOI] [options]
```

- `OÙ` est un dossier (fouillé avec ses sous-dossiers) ou un seul fichier.
  Si vous l'omettez, le dossier courant (`.`) est utilisé.
- `QUOI` est ce que vous cherchez. Seul, il signifie **chercher à
  l'intérieur du contenu des fichiers**.

Essayez ceci (remplacez `~/docs` par un dossier que vous avez vraiment) :

```bash
# Quels fichiers mentionnent "annual report" ? (les deux mots, n'importe où dans le fichier)
./searcher ~/docs "annual report"

# Chercher dans le dossier courant, à la place
./searcher . "annual report"

# Compter seulement les résultats (utile dans les scripts)
./searcher ~/docs "annual report" --count
```

Pendant l'exécution, vous verrez un affichage d'état d'une ligne : d'abord
combien de fichiers ont été trouvés, puis une barre de progression avec le
fichier en cours de lecture et, à la fin, un résumé comme
`searcher: 3 match(es) in 120 file(s) (0.8s)`. Cela passe par un canal séparé
(stderr), donc ne pollue jamais la sortie redirigée (pipes). Ajoutez `-q` (ou
`--quiet`) pour le désactiver, ex. : `./searcher ~/docs "x" --count -q`.

## 5. Chercher par nom de fichier

Ajoutez `-n` (ou `--name`) pour comparer avec les noms de fichiers plutôt que
le contenu :

```bash
# Fichiers dont le NOM contient "report" ou "invoice"
./searcher ~/docs --name "report OR invoice"

# Les noms suivent la même logique de mots que le contenu (voir section suivante)
./searcher ~/docs --name "2024 budget"
```

Vous pouvez même combiner les deux : le nom doit correspondre à X **et** le
contenu doit correspondre à Y :

```bash
# Fichiers nommés comme "invoice" dont le contenu mentionne "paid" ou "overdue"
./searcher ~/docs -n invoice -s "paid OR overdue"
```

(`-s` / `--content` rend explicite qu'il s'agit d'une recherche de contenu.)

## 6. Le langage de recherche : AND, OR et phrases exactes

C'est le cœur de `searcher`. Les mêmes règles s'appliquent aux recherches par
nom et par contenu. La comparaison ignore majuscules/minuscules, sauf si vous
passez `--case-sensitive`.

| Vous tapez | Vous obtenez |
|---|---|
| `apple` | les fichiers contenant `apple` |
| `apple banana` | les fichiers contenant `apple` **ET** `banana` (n'importe où, dans n'importe quel ordre) |
| `apple OR banana` | les fichiers contenant `apple` **ou** `banana` (au moins l'un) |
| `"apple pie"` | les fichiers contenant la phrase exacte `apple pie` |
| `"apple pie" "banana bread"` | les fichiers contenant **les deux** phrases exactes |
| `"apple pie" OR "banana bread"` | les fichiers contenant **au moins l'une** des phrases |

### Les guillemets dans le terminal (important !)

Le terminal (shell) utilise lui aussi des guillemets ; vous devez donc
entourer votre recherche de guillemets pour qu'elle arrive entière au
programme. Règles pratiques :

- **Entourez toujours la recherche de guillemets doubles** :
  `"annual report"`.
- **Les phrases exactes exigent des guillemets simples dehors et doubles
  dedans** : `'"quarterly results"'`. (Des guillemets doubles dehors
  feraient « manger » ceux de dedans par le shell.)
- Le mot `OR` doit être en majuscules pour agir comme « ou ». Pour chercher
  le mot littéral « or », mettez-le entre guillemets : `'"or"'`.

Exemples à copier :

```bash
./searcher ~/docs "budget forecast"                        # AND (ET)
./searcher ~/docs "budget OR forecast"                     # OR (OU)
./searcher ~/docs '"quarterly results"'                    # phrase exacte
./searcher ~/docs '"quarterly results" "annual summary"'   # les deux phrases
./searcher ~/docs '"quarterly results" OR "annual summary"' # l'une ou l'autre phrase
```

## 7. Limiter les fichiers fouillés

Par défaut, la recherche de contenu examine les fichiers type document
(`txt`, `md`, `pdf`, `docx`, `odt`, `rtf`, `epub`, `pptx`, `xlsx`, …) et saute
le code source. Vous pouvez changer cela :

```bash
# Seulement les fichiers PDF
./searcher ~/docs "contract" --ext pdf

# Plusieurs formats (séparés par des virgules, avec ou sans points)
./searcher ~/docs "contract" --ext pdf,docx,txt

# Inclure aussi le code source et les scripts
./searcher ~/projects "TODO" --all-text

# Voir la liste complète des extensions prises en charge
./searcher --list-exts
```

Autres options utiles de portée :

```bash
./searcher ~/docs "x" --no-recursive   # dossier courant seul, sans sous-dossiers
./searcher ~/docs "x" --hidden         # inclure aussi fichiers/dossiers cachés
./searcher ~/docs "x" --max-size 10    # sauter les fichiers de plus de 10 Mo
./searcher ~/docs "X" --case-sensitive # "X" ne correspond plus à "x"
./searcher ~/docs "x" --absolute       # afficher les chemins complets au lieu des courts
```

## 8. Lire les résultats

Chaque résultat montre le fichier plus jusqu'à 2 lignes d'aperçu, avec les
mots trouvés marqués `*comme ceci*` :

```bash
./searcher ~/docs "warranty" --lines 5    # affiche 5 lignes d'aperçu par fichier
./searcher ~/docs "warranty" --lines 0    # pas d'aperçus, juste les noms de fichiers
./searcher ~/docs "warranty" --limit 5    # affiche seulement les 5 premiers résultats
./searcher ~/docs "warranty" --count      # affiche seulement le nombre, ex. : 14
```

Codes de sortie (utiles en combinaison avec d'autres commandes) : `0` =
quelque chose trouvé, `1` = rien trouvé, `2` = erreur (ex. : dossier
inexistant).

## 9. Les outils classiques, côte à côte

Mêmes tâches, outils différents. Lancez-les pour sentir la différence (avec
un dossier `~/docs` contenant un fichier `notes.txt` avec « call Ana about
the warranty », c'est-à-dire « appeler Ana au sujet de la garantie ») :

```bash
# --- par NOM ---
find ~/docs -type f -name "*warranty*"     # parcours en direct, motif glob
locate warranty                            # instantané, mais peut rater les fichiers récents
./searcher ~/docs --name warranty          # parcours en direct, logique de mots

# --- par CONTENU (texte brut) ---
grep -ri "warranty" ~/docs                 # classique, montre chaque ligne correspondante
grep -rli "warranty" ~/docs                # -l : noms de fichiers seuls
grep -rn -C 2 "warranty" ~/docs            # -n : numéros de ligne, -C 2 : 2 lignes de contexte
./searcher ~/docs "warranty"               # noms de fichiers + courts aperçus

# --- par CONTENU, deux mots n'importe où (AND/ET) ---
grep -ril "warranty" ~/docs | xargs grep -li "ana"   # façon maladroite en deux étapes
./searcher ~/docs "warranty ana"                     # pareil, directement

# --- dans les PDF : seul searcher fonctionne ---
grep -ri "warranty" ~/docs                 # "Binary file report.pdf matches" — inutile
./searcher ~/docs "warranty" --ext pdf     # lit vraiment le texte du PDF
```

Un mot sur `locate` : s'il ne trouve jamais vos fichiers récents, sa base de
données est obsolète. La rafraîchir (`sudo updatedb`) exige des droits
d'administrateur — une raison de plus pour laquelle la recherche en direct de
`searcher` est pratique.

## 10. Recettes : tâches courantes

```bash
# Tous les PDF qui mentionnent "LinkedIn"
./searcher ~/Documents "LinkedIn" --ext pdf

# Factures (par nom) qui mentionnent "overdue" (en retard)
./searcher ~/docs -n invoice -s overdue

# Combien de notes de réunion mentionnent "budget" ou "forecast" ?
./searcher ~/notes "budget OR forecast" --count

# Message d'erreur exact dans plusieurs manuels (tout format)
./searcher ~/manuals '"paper jam in tray 2"'

# Tout ce qui mentionne le client, seulement les 10 premiers (en limitant la portée)
./searcher ~/clients/acme "acme" --ext pdf,docx --limit 10

# Chercher dans un seul fichier
./searcher ~/docs/handbook.pdf "parental leave"

# Voir POURQUOI certains fichiers ont été sautés
./searcher ~/docs "x" --errors
```

## 11. Dépannage

**`command not found`** — vous avez oublié le `./` (lancez `./searcher`, pas
`searcher`), sauf si vous l'avez installé dans votre `PATH` (voir section 3).

**`Permission denied`** — lancez `chmod +x searcher` une fois.

**Aucun résultat, mais le fichier contient sûrement le mot** — vérifiez trois
choses :
1. L'extension du fichier fait-elle partie de l'ensemble fouillé ?
   (`./searcher --list-exts`). Le code source exige `--all-text` ; tout le
   reste exige `--ext`.
2. Est-ce un vieux fichier `.doc` (pas `.docx`) ? Ceux-là sont sautés avec un
   avertissement — convertissez en `.docx` d'abord (ex. : avec LibreOffice).
3. Est-ce un PDF *numérisé* (photos de pages, sans vrai texte) ? Aucun outil
   de recherche textuelle ne les lit sans logiciel d'OCR.

**Avertissements PDF bizarres** — vous ne devriez en voir aucun ; `searcher`
rend muet le bavardage de l'analyseur PDF et saute les fichiers corrompus en
silence. Ajoutez `--errors` pour lister les fichiers sautés et pourquoi.

**« PDF fallback extractor in use »** — installez `pypdf` une fois
(`pip install pypdf`) pour une bien meilleure extraction du texte des PDF.

**Recherche lente** — réduisez la portée : `--ext pdf,docx`, `--max-size 20`,
`--no-recursive`, ou visez un dossier plus petit. Les très grandes
arborescences sont parcourues fichier par fichier (un à la fois), comme avec
`grep -r`.

## 12. Aide-mémoire

```bash
./searcher . "words"                    # contenu, dossier courant, récursif
./searcher ~/docs "a b"                 # AND (ET)
./searcher ~/docs "a OR b"              # OR (OU)
./searcher ~/docs '"exact phrase"'      # phrase exacte
./searcher ~/docs --name "invoice"      # par nom de fichier
./searcher ~/docs -n inv -s "paid"      # nom ET contenu
./searcher ~/docs "x" --ext pdf,docx    # ces formats seulement
./searcher ~/docs "x" --count           # juste le nombre
./searcher ~/docs "x" -q                # sans barre d'état
./searcher --help                       # manuel complet
```

Bonne recherche !
