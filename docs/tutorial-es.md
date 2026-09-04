# Tutorial de searcher para principiantes en Linux

Este tutorial te enseña a usar **searcher**, un pequeño programa que encuentra
archivos de documentos en tu computadora **por el nombre del archivo o por lo
que está escrito dentro de ellos**. No se asume experiencia previa en Linux:
cada paso está explicado y todos los comandos se pueden copiar y pegar.

Al final serás capaz de:

- ejecutar `searcher` desde la terminal,
- encontrar archivos por nombre y por contenido,
- combinar palabras de búsqueda con `AND`, `OR` y `"frases exactas"`,
- limitar la búsqueda a PDF, archivos de Word y otros formatos,
- entender en qué se diferencia `searcher` de `find`, `locate` y `grep`.

---

## 1. ¿Qué es searcher?

`searcher` es un programa de un solo archivo. Le indicas:

1. **dónde buscar** — una carpeta (o un solo archivo), y
2. **qué buscar** — una o más palabras, o una frase exacta.

Responde con una lista de los archivos coincidentes, más una breve vista
previa de las líneas correspondientes. Entiende archivos de texto simple
(`txt`, `md`, `csv`, …) y documentos reales: **PDF**, **Word** (`docx`),
**OpenDocument** (`odt`), **RTF**, **ePub**, **PowerPoint** (`pptx`) y
**Excel** (`xlsx`).

Ejemplo de cómo se ve un resultado:

```text
searcher: content:'"quarterly results"' under /home/ana/docs (recursive)
────────────────────────────────────────────────────────────────────────
  [1] report_2024.pdf
    │ ...*quarterly results* grew 12 percent...

────────────────────────────────────────────────────────────────────────
1 match(es)
```

## 2. ¿En qué se diferencia searcher de find, locate y grep?

Linux ya tiene herramientas clásicas de búsqueda. Cada una es buena en algo,
y `searcher` llena el hueco que dejan. Aquí va la versión corta:

| Tarea | `find` | `locate` | `grep` | `searcher` |
|---|---|---|---|---|
| Encontrar archivos **por nombre** | sí | sí (instantáneo) | no | sí |
| Encontrar archivos **por contenido** | solo junto con `grep` | no | sí (archivos de texto) | sí |
| Leer dentro de archivos **PDF / Word** | no | no | no | sí |
| `"frases exactas"`, `AND`, `OR` | no (requiere trucos de regex) | no | parcial (regex) | sí, sintaxis sencilla |
| Necesita base de datos de índice | no (recorrido en vivo) | sí (actualizada ~1 vez al día) | no (recorrido en vivo) | no (recorrido en vivo) |
| Muestra fragmentos de vista previa | no | no | sí (`-C`) | sí |

En palabras sencillas:

- **`find`** recorre carpetas y compara *nombres* de archivos (o fechas,
  tamaños, …). No sabe nada del *contenido* de los archivos. Para mirar
  dentro hay que combinarlo con `grep`, lo que pronto se vuelve engorroso.
- **`locate`** es muy rápido porque busca en una lista ya preparada de nombres
  de archivos en vez de en tu disco. Desventajas: solo conoce nombres (nunca
  contenidos), y su lista por lo general se reconstruye una vez al día — así
  que los archivos creados hace una hora pueden no aparecer.
- **`grep`** (muy usado como `grep -r`) busca *dentro* de los archivos y es
  excelente para texto simple y código. Pero no puede leer archivos PDF ni de
  Word — en esos casos solo dice "Binary file matches" (o nada útil), y no
  tiene una lógica sencilla de `AND` / `OR` / frases.
- **`searcher`** es la herramienta del tipo "dime qué documentos mencionan
  X": búsqueda en vivo (sin base de datos desactualizada), formatos reales de
  documentos y un lenguaje de búsqueda hecho de palabras comunes en vez de
  expresiones regulares.

Cuándo usar cada una:

- "¿Dónde guardé ese archivo llamado invoice?" → `locate invoice` o
  `find ~ -name "*invoice*"` (rápido, sencillo).
- "¿Qué archivos de código contienen tal nombre de función?" →
  `grep -rn "def foo" .` (`grep` sigue siendo imbatible para código fuente).
- "¿Cuáles de mis PDF y docs de Word mencionan quarterly results?" →
  `searcher`. Ni `find`, ni `locate`, ni `grep` hacen esto bien.

## 3. Abrir una terminal y hacer la primera búsqueda

En la mayoría de los sistemas Linux, pulsa `Ctrl + Alt + T` para abrir una
terminal. Verás algo como `ana@pc:~$`. El `~` significa tu carpeta personal
(home).

Supón que descargaste `searcher` en `~/searcher` (la carpeta del proyecto).
Primero, entra allí y haz el programa ejecutable (esto solo se hace una vez):

```bash
cd ~/searcher
chmod +x searcher
./searcher --help
```

Notas para principiantes:

- `cd` significa "change directory" (entrar en una carpeta).
- `chmod +x` significa "permitir que este archivo se ejecute como programa".
- El `./` delante significa "ejecuta el programa que está en *esta*
  carpeta". Linux no ejecuta programas de la carpeta actual a menos que lo
  pidas con `./` — esto es normal, no es un error.
- Si ves `command not found`, probablemente olvidaste el `./`.
- Si ves `Permission denied`, ejecuta de nuevo la línea `chmod +x searcher`.

`./searcher --help` muestra el manual completo. Pruébalo ahora — todo lo de
este tutorial también está resumido allí.

> **Consejo:** para ejecutar `searcher` desde cualquier lugar sin `cd`,
> ponlo en tu `PATH` una vez: `ln -s "$PWD/searcher" ~/.local/bin/searcher`.
> Después bastará con escribir `searcher` en cualquier carpeta. (Cierra y
> abre sesión de nuevo si la terminal no lo encuentra de inmediato.)

## 4. Las primeras búsquedas

La forma básica es:

```bash
./searcher [DÓNDE] [QUÉ] [opciones]
```

- `DÓNDE` es una carpeta (se busca incluyendo las subcarpetas) o un solo
  archivo. Si lo omites, se usa la carpeta actual (`.`).
- `QUÉ` es lo que buscas. Solo significa **buscar dentro del contenido de
  los archivos**.

Prueba estos (cambia `~/docs` por una carpeta que realmente tengas):

```bash
# ¿Qué archivos mencionan "annual report"? (ambas palabras, en cualquier lugar del archivo)
./searcher ~/docs "annual report"

# Buscar en la carpeta actual, en su lugar
./searcher . "annual report"

# Solo contar las coincidencias (útil en scripts)
./searcher ~/docs "annual report" --count
```

Mientras se ejecuta, verás una línea de estado: primero cuántos archivos se
encontraron, luego una barra de progreso con el archivo que se está leyendo
y, al final, un resumen como `searcher: 3 match(es) in 120 file(s) (0.8s)`.
Esto va a un canal separado (stderr), así que nunca ensucia la salida usada
en tuberías (pipes). Agrega `-q` (o `--quiet`) para apagarlo, ej.:
`./searcher ~/docs "x" --count -q`.

## 5. Buscar por el nombre del archivo

Agrega `-n` (o `--name`) para comparar con los nombres de los archivos en vez
del contenido:

```bash
# Archivos cuyo NOMBRE contiene "report" o "invoice"
./searcher ~/docs --name "report OR invoice"

# Los nombres siguen la misma lógica de palabras que el contenido (ver la próxima sección)
./searcher ~/docs --name "2024 budget"
```

Hasta puedes combinar ambos: el nombre debe coincidir con X **y** el contenido
debe coincidir con Y:

```bash
# Archivos llamados como "invoice" cuyo contenido menciona "paid" u "overdue"
./searcher ~/docs -n invoice -s "paid OR overdue"
```

(`-s` / `--content` deja explícito que es búsqueda de contenido.)

## 6. El lenguaje de búsqueda: AND, OR y frases exactas

Este es el corazón de `searcher`. Las mismas reglas valen para búsquedas por
nombre y por contenido. La comparación ignora mayúsculas/minúsculas, a menos
que pases `--case-sensitive`.

| Escribes | Obtienes |
|---|---|
| `apple` | archivos que contienen `apple` |
| `apple banana` | archivos que contienen `apple` **Y** `banana` (en cualquier lugar, en cualquier orden) |
| `apple OR banana` | archivos que contienen `apple` **u** `banana` (al menos uno) |
| `"apple pie"` | archivos que contienen la frase exacta `apple pie` |
| `"apple pie" "banana bread"` | archivos que contienen **ambas** frases exactas |
| `"apple pie" OR "banana bread"` | archivos que contienen **al menos una** de las frases |

### Comillas en la terminal (¡importante!)

La terminal (shell) también usa comillas, así que debes envolver tu búsqueda
entre comillas para que llegue completa al programa. Reglas prácticas:

- **Envuelve siempre la búsqueda en comillas dobles**: `"annual report"`.
- **Las frases exactas necesitan comillas simples fuera y dobles dentro**:
  `'"quarterly results"'`. (Comillas dobles fuera harían que el shell se
  "coma" las de dentro.)
- La palabra `OR` debe estar en mayúsculas para funcionar como "o". Para
  buscar la palabra literal "or", ponla entre comillas: `'"or"'`.

Ejemplos para copiar:

```bash
./searcher ~/docs "budget forecast"                        # AND (Y)
./searcher ~/docs "budget OR forecast"                     # OR (O)
./searcher ~/docs '"quarterly results"'                    # frase exacta
./searcher ~/docs '"quarterly results" "annual summary"'   # ambas frases
./searcher ~/docs '"quarterly results" OR "annual summary"' # cualquiera de las frases
```

## 7. Limitar qué archivos se buscan

Por defecto, la búsqueda de contenido mira archivos tipo documento (`txt`,
`md`, `pdf`, `docx`, `odt`, `rtf`, `epub`, `pptx`, `xlsx`, …) y omite el
código fuente. Puedes cambiarlo:

```bash
# Solo archivos PDF
./searcher ~/docs "contract" --ext pdf

# Varios formatos (separados por comas, con o sin punto)
./searcher ~/docs "contract" --ext pdf,docx,txt

# Incluir también código fuente y scripts
./searcher ~/projects "TODO" --all-text

# Ver la lista completa de extensiones admitidas
./searcher --list-exts
```

Otras opciones útiles de alcance:

```bash
./searcher ~/docs "x" --no-recursive   # solo la carpeta actual, sin subcarpetas
./searcher ~/docs "x" --hidden         # incluir también archivos/carpetas ocultos
./searcher ~/docs "x" --max-size 10    # omitir archivos de más de 10 MB
./searcher ~/docs "X" --case-sensitive # "X" ya no coincide con "x"
./searcher ~/docs "x" --absolute       # mostrar rutas completas en vez de cortas
```

## 8. Leer los resultados

Cada acierto muestra el archivo más hasta 2 líneas de vista previa, con las
palabras encontradas marcadas `*así*`:

```bash
./searcher ~/docs "warranty" --lines 5    # muestra 5 líneas de vista previa por archivo
./searcher ~/docs "warranty" --lines 0    # sin vistas previas, solo nombres de archivos
./searcher ~/docs "warranty" --limit 5    # muestra solo los 5 primeros aciertos
./searcher ~/docs "warranty" --count      # muestra solo el número, ej.: 14
```

Códigos de salida (útiles al combinar con otros comandos): `0` = encontró
algo, `1` = nada encontrado, `2` = error (ej.: la carpeta no existe).

## 9. Las herramientas clásicas, lado a lado

Las mismas tareas, con herramientas distintas. Ejecútalas para sentir la
diferencia (usando una carpeta `~/docs` con un archivo `notes.txt` que
contiene "call Ana about the warranty", es decir, "llama a Ana por la
garantía"):

```bash
# --- por NOMBRE ---
find ~/docs -type f -name "*warranty*"     # recorrido en vivo, patrón glob
locate warranty                            # instantáneo, pero puede perder archivos nuevos
./searcher ~/docs --name warranty          # recorrido en vivo, lógica de palabras

# --- por CONTENIDO (texto simple) ---
grep -ri "warranty" ~/docs                 # clásico, muestra cada línea coincidente
grep -rli "warranty" ~/docs                # -l: solo nombres de archivos
grep -rn -C 2 "warranty" ~/docs            # -n: números de línea, -C 2: 2 líneas de contexto
./searcher ~/docs "warranty"               # nombres de archivos + vistas previas cortas

# --- por CONTENIDO, dos palabras en cualquier lugar (AND/Y) ---
grep -ril "warranty" ~/docs | xargs grep -li "ana"   # forma torpe en dos pasos
./searcher ~/docs "warranty ana"                     # lo mismo, directo

# --- dentro de PDF: solo searcher funciona ---
grep -ri "warranty" ~/docs                 # "Binary file report.pdf matches" — inútil
./searcher ~/docs "warranty" --ext pdf     # lee de verdad el texto del PDF
```

Una nota sobre `locate`: si nunca encuentra tus archivos nuevos, su base de
datos está desactualizada. Actualizarla (`sudo updatedb`) exige permisos de
administrador — un motivo más por el que la búsqueda en vivo de `searcher`
es conveniente.

## 10. Recetas: tareas comunes

```bash
# Todos los PDF que mencionan "LinkedIn"
./searcher ~/Documents "LinkedIn" --ext pdf

# Facturas (por nombre) que mencionan "overdue" (vencido)
./searcher ~/docs -n invoice -s overdue

# ¿Cuántas notas de reunión mencionan "budget" o "forecast"?
./searcher ~/notes "budget OR forecast" --count

# Mensaje exacto de error en varios manuales (cualquier formato)
./searcher ~/manuals '"paper jam in tray 2"'

# Todo lo que menciona al cliente, solo los 10 primeros (limitando el alcance)
./searcher ~/clients/acme "acme" --ext pdf,docx --limit 10

# Buscar en un solo archivo
./searcher ~/docs/handbook.pdf "parental leave"

# Ver POR QUÉ se omitieron algunos archivos
./searcher ~/docs "x" --errors
```

## 11. Solución de problemas

**`command not found`** — olvidaste el `./` (ejecuta `./searcher`, no
`searcher`), a menos que lo hayas instalado en tu `PATH` (ver la sección 3).

**`Permission denied`** — ejecuta `chmod +x searcher` una vez.

**Sin resultados, pero el archivo seguro contiene la palabra** — revisa tres
cosas:
1. ¿La extensión del archivo está en el conjunto buscado?
   (`./searcher --list-exts`). El código fuente necesita `--all-text`;
   cualquier otra cosa necesita `--ext`.
2. ¿Es un archivo `.doc` antiguo (no `.docx`)? Esos se omiten con un aviso —
   conviértelo a `.docx` primero (ej.: con LibreOffice).
3. ¿Es un PDF *escaneado* (fotos de páginas, sin texto real)? Ninguna
   herramienta de búsqueda de texto los lee sin un programa de OCR.

**Avisos extraños de PDF** — no deberías ver ninguno; `searcher` silencia la
cháchara del analizador de PDF y omite archivos corruptos en silencio. Agrega
`--errors` para listar qué archivos se omitieron y por qué.

**"PDF fallback extractor in use"** — instala `pypdf` una vez
(`pip install pypdf`) para una extracción de texto de PDF mucho mejor.

**Búsqueda lenta** — reduce el alcance: `--ext pdf,docx`, `--max-size 20`,
`--no-recursive`, o apunta a una carpeta más pequeña. Los árboles muy grandes
se recorren archivo por archivo (de uno en uno), como con `grep -r`.

## 12. Hoja de referencia rápida

```bash
./searcher . "words"                    # contenido, carpeta actual, recursivo
./searcher ~/docs "a b"                 # AND (Y)
./searcher ~/docs "a OR b"              # OR (O)
./searcher ~/docs '"exact phrase"'      # frase exacta
./searcher ~/docs --name "invoice"      # por el nombre del archivo
./searcher ~/docs -n inv -s "paid"      # nombre Y contenido
./searcher ~/docs "x" --ext pdf,docx    # solo estos formatos
./searcher ~/docs "x" --count           # solo el número
./searcher ~/docs "x" -q                # sin barra de estado
./searcher --help                       # manual completo
```

¡Felices búsquedas!
