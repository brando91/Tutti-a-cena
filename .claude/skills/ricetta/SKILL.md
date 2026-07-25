---
name: ricetta
description: "Import a recipe the user pastes into chat (title, ingredients, instructions — copied from a webpage or app) into this repo's ricette/ folder as a clean, standardized Markdown file scaled to 4 servings. Use whenever the user invokes /ricetta, pastes recipe text/content and asks to save/import it, or asks to 'add this recipe to the repo'. Not for general recipe questions or cooking advice with no recipe content to import."
---

## Cosa fa questa skill

L'utente incolla in chat il contenuto di una ricetta (copiato da una pagina web, un'app o scritto a mano) invece di passare un URL da recuperare — niente fetch web, quindi funziona anche in sessioni/ambienti senza accesso a internet. La skill pulisce quel testo, lo trasforma in un file Markdown standardizzato con le dosi riportate a 4 persone, e lo salva nella sottocartella corretta di `ricette/` in questo repository.

## Passo 1 — Pulizia del contenuto incollato

Il contenuto grezzo arriva incollato nel messaggio dell'utente (spesso è tutto il testo di una pagina web: menu, pubblicità, articoli correlati, recensioni prodotto...). Non serve fare fetch di nessuna URL: lavora direttamente su quel testo.

Estrai **solo** questi tre elementi, scartando tutto il resto:

- **Titolo** della ricetta
- **Lista ingredienti**, con quantità e unità di misura originali
- **Procedimento**, passo per passo

Non è necessaria un'immagine: la ricetta viene identificata con un paio di emoji rappresentative (vedi Passo 4), non con una foto.

Se nel testo incollato compare anche l'URL/nome della pagina di origine, tienilo da parte per il campo "Fonte" del Passo 5 — ma non è un dato indispensabile, e se l'utente non lo fornisce va semplicemente omesso (non chiederlo esplicitamente, non inventarlo).

Cerca anche, se presente, il numero di porzioni originario (es. "per 2 persone", "serves 6", "dosi per 8"): serve per il passo successivo. Se il testo non lo dichiara, annotalo come non specificato — non inventarlo.

## Passo 2 — Riporta le dosi a 4 persone

Se la ricetta originale indica un numero di porzioni diverso da 4, calcola il fattore di scala (4 / porzioni originali) e moltiplica ogni quantità.

Linee guida per un risultato che abbia senso in cucina, non solo aritmeticamente corretto:
- Arrotonda a valori pratici (es. 233 g → "circa 230 g", non "233,33 g").
- Per ingredienti indivisibili (uova, spicchi d'aglio, foglie di alloro, bustine…) arrotonda all'intero più sensato e usa il buon senso — es. 1,5 uova diventano 2, non "1,5 uova".
- Se le porzioni originali non erano specificate, non forzare uno scaling: riporta le quantità così come sono e aggiungi una nota nel file ("dosi originali non specificate, si assume fossero già per circa 4 persone").

## Passo 3 — Classifica l'ingrediente principale

Determina l'ingrediente/base predominante della ricetta e assegnalo a una categoria come: `pasta`, `riso`, `bulgur`, `cous cous`, `carne bianca`, `carne rossa`, `pesce`, `uova`, `legumi`, ...

Regole di classificazione:
- **carne bianca**: pollo, tacchino, coniglio e altre carni bianche.
- **carne rossa**: manzo, vitello, maiale, agnello, selvaggina.
- **pesce**: pesce e frutti di mare/molluschi/crostacei.
- Se la ricetta è un contorno usa come tipologia il nome reale dell'ingrediente principale (es. "verdure", "patate", "legumi") — questo è normale per i contorni.

Questa classificazione determina anche la cartella di destinazione (vedi Passo 6).

## Passo 4 — Scegli 2-3 emoji rappresentative

Ogni ricetta viene identificata da 2-3 emoji invece che da un'immagine: niente URL da recuperare, niente file da generare, e restano comunque distintive perché variano da ricetta a ricetta.

Componi la sequenza così:
1. **Un'emoji di categoria**, coerente con la tipologia del Passo 3 — usa questa tabella come riferimento (non è esaustiva, scegli l'emoji più sensata se manca una voce):

   | Tipologia | Emoji |
   |---|---|
   | pasta | 🍝 |
   | riso | 🍚 |
   | bulgur / cous cous | 🌾 |
   | carne bianca | 🍗 |
   | carne rossa | 🥩 |
   | pesce | 🐟 |
   | verdure | 🥦 |
   | patate | 🥔 |
   | legumi | 🫘 |
   | uova | 🥚 |
   | funghi | 🍄 |

2. **1-2 emoji degli ingredienti caratterizzanti** di quella specifica ricetta (es. formaggio 🧀, frutta secca 🌰, limone 🍋, pomodoro 🍅, vino 🍷, aglio 🧄, peperoncino 🌶️, basilico 🌿, panna 🥛...) — sono queste a rendere l'emoji-set diverso da un'altra ricetta della stessa categoria.

Esempio: un risotto al gorgonzola e noci → 🍚🧀🌰. Una tagliata di manzo al rosmarino → 🥩🌿.

## Passo 5 — Scrivi il file Markdown

Usa esattamente questo template:

```markdown
# {emoji di categoria}{emoji ingredienti} {Titolo}

**Tipologia:** {categoria dal Passo 3}

**Ultima volta mangiato:** _(da compilare)_

## Ingredienti (per 4 persone)

- {ingrediente 1 con quantità scalata}
- {ingrediente 2 con quantità scalata}
- ...

## Procedimento

1. {passo 1}
2. {passo 2}
...
```

Il campo "Ultima volta mangiato" va sempre incluso ma mai valorizzato: lascialo esattamente come `_(da compilare)_`, anche se dal contesto potresti dedurre una data plausibile — è un campo che l'utente aggiorna a mano ogni volta che rifà il piatto, non qualcosa da stimare o dedurre.

Se l'utente ha fornito anche l'URL/nome della pagina di origine, aggiungi una riga `**Fonte:** {...}` subito sotto "Tipologia" — altrimenti ometti del tutto quel campo, non lasciare placeholder vuoti.

Se hai dovuto assumere qualcosa (porzioni originali mancanti, ingrediente principale ambiguo), aggiungi una riga di nota subito sotto il titolo.

## Passo 6 — Determina la cartella di destinazione e salva

Mappa la categoria del Passo 3 sulla sottocartella:

| Categoria | Cartella |
|---|---|
| cereali: pasta, riso, bulgur, cous cous, ... | `ricette/primi/` |
| carne bianca, carne rossa, pesce, uova | `ricette/secondi/` |
| qualsiasi altra cosa (verdure, patate, legumi...) | `ricette/contorni/` |

Nome del file: slug del titolo in minuscolo, spazi sostituiti da trattini, senza accenti/caratteri speciali, estensione `.md` (es. "Spaghetti alla Carbonara" → `spaghetti-alla-carbonara.md`).

Salva il file nel repository:

1. Se stai lavorando su un clone locale del repo (caso più comune in questa sessione), scrivi il file con lo strumento di scrittura file nella cartella corretta, poi:
   ```
   git add ricette/<sottocartella>/<slug>.md
   git commit -m "Aggiunge ricetta: <Titolo>"
   git push -u origin <branch corrente>
   ```
   Usa il branch attualmente attivo, a meno che l'utente non chieda esplicitamente un branch diverso. Non forzare push né sovrascrivere altri file.
2. Se non hai un clone locale (es. sessione senza repo checked out), usa gli strumenti MCP di GitHub (`create_or_update_file` o equivalente) per creare il file direttamente sul branch di lavoro.

Al termine, conferma all'utente titolo, cartella di destinazione e percorso del file salvato.
