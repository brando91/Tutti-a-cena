---
name: ricetta
description: "Import a recipe from a URL into this repo's ricette/ folder as a clean, standardized Markdown file scaled to 4 servings. Use whenever the user invokes /ricetta <url>, pastes a recipe link and asks to save/import it, or asks to 'add this recipe to the repo'. Not for general recipe questions or cooking advice with no URL involved."
---

## Cosa fa questa skill

Data l'URL di una pagina ricetta, produce un file Markdown pulito e standardizzato, con le dosi riportate a 4 persone, e lo salva nella sottocartella corretta di `ricette/` in questo repository.

## Passo 1 — Recupero e pulizia del contenuto

Usa lo strumento di fetch web (WebFetch, o `curl` se WebFetch non è disponibile) per scaricare la pagina all'URL fornito.

Estrai **solo** questi quattro elementi, scartando tutto il resto (pubblicità, biografia dell'autore, commenti, valori nutrizionali, articoli correlati, video incorporati, ecc.):

- **Titolo** della ricetta
- **Lista ingredienti**, con quantità e unità di misura originali
- **Procedimento**, passo per passo
- **Immagine principale**: di solito il tag `og:image` nel `<head>`, oppure la prima immagine "hero" associata alla ricetta

Cerca anche, se presente, il numero di porzioni originario (es. "per 2 persone", "serves 6", "dosi per 8"): serve per il passo successivo. Se la pagina non lo dichiara, annotalo come non specificato — non inventarlo.

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

Questa classificazione determina anche la cartella di destinazione (vedi Passo 5).

## Passo 4 — Scrivi il file Markdown

Usa esattamente questo template:

```markdown
# {Titolo}

![{Titolo}]({URL immagine principale})

**Tipologia:** {categoria dal Passo 3}

**Fonte:** {URL originale}

## Ingredienti (per 4 persone)

- {ingrediente 1 con quantità scalata}
- {ingrediente 2 con quantità scalata}
- ...

## Procedimento

1. {passo 1}
2. {passo 2}
...
```

L'immagine viene collegata tramite l'URL originale (non viene riscaricata né ri-ospitata nel repo), per rispettare i diritti di chi l'ha pubblicata e mantenere il repo leggero.

Se hai dovuto assumere qualcosa (porzioni originali mancanti, ingrediente principale ambiguo), aggiungi una riga di nota subito sotto il titolo.

## Passo 5 — Determina la cartella di destinazione e salva

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
