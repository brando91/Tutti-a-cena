---
name: settimana
description: "Generate the weekly meal plan (14 meals: pranzo e cena for 7 days) for the upcoming week, picking recipes from this repo's ricette/ folder and saving it as a new file in menu/ named after the week's date range. Use whenever the user invokes /settimana or asks for a 'menu settimanale', 'piano dei pasti', or to plan what to cook for the coming week. Not for a single recipe request — that's the /ricetta skill."
---

## Cosa fa questa skill

Genera il piano dei pasti per la settimana successiva a quella corrente (lunedì–domenica, 14 pasti: pranzo e cena), pescando le ricette dalla cartella `ricette/` di questo repository e rispettando vincoli su tempo di preparazione, varietà, stagionalità e frequenza degli ingredienti principali. Salva il piano come nuovo file Markdown in `menu/`, con il nome basato sull'intervallo di date della settimana (vedi Passo 5) così che i file risultino ordinabili alfabeticamente in ordine cronologico.

## Passo 1 — Fai sempre queste domande prima di generare il piano

Non generare mai il piano senza aver prima chiesto:

1. **In quali giorni della prossima settimana sarai in ufficio?** Serve per sapere in quali sere applicare il limite di tempo di preparazione più stretto (vedi Passo 4).
2. **Ci sono pasti per cui non è necessario pianificare una ricetta** (es. perché si mangerà fuori)? Chiedi giorno e pasto specifico (pranzo/cena).

Non chiedere altro oltre a queste due: l'abbinamento dei contorni (vedi Passo 4) è già deciso in modo permanente e non va richiesto di nuovo.

## Passo 2 — Raccogli i dati delle ricette

Usa Grep con il pattern `^# |^\*\*Tipologia:|^\*\*Tempo di preparazione:|^\*\*Ultima volta mangiato:` sulla cartella `ricette/` (ricorsivo, con i numeri di riga) per estrarre in un colpo solo, per ogni ricetta: emoji+titolo, tipologia, tempo di preparazione in minuti, e data dell'ultima volta mangiato (o `_(da compilare)_`). Non serve leggere il corpo di ogni ricetta (ingredienti/procedimento) — il piano si basa solo su questi metadati più il percorso del file per il link. Fai eccezione solo per le ricette che stai valutando di includere e il cui titolo/tipologia suggerisce la presenza di legumi (ceci, fagioli, lenticchie, piselli, fagiolini...): in quel caso apri il file per verificare se sono nell'elenco ingredienti — non serve controllare come sono descritti (lessati, in scatola, surgelati, ecc.), vedi Passo 4.

Tieni traccia della sottocartella di ciascuna ricetta (`primi`/`secondi`/`contorni`), perché insieme alla "Tipologia" serve per applicare i vincoli di frequenza.

## Passo 3 — Determina le date della settimana

La settimana da pianificare è quella successiva a quella corrente. Calcola le date esatte con il comando `date` (es. `date -d "oggi +N days" +"%A %d/%m/%Y"`) partendo dalla data odierna — non dedurle a mente, verificale sempre così.

## Passo 4 — Applica i vincoli, in quest'ordine

1. **Pasti da non pianificare**: per i pasti indicati dall'utente al Passo 1, non assegnare nessuna ricetta — segnalali semplicemente come tali nel piano.
2. **Venerdì sera è sempre pizza 🍕**: non scegliere una ricetta dal repository per la cena di venerdì, salvo che l'utente non abbia indicato al Passo 1 che quel pasto specifico va escluso per quella settimana (es. si mangia fuori). Scrivi semplicemente `- **Cena:** 🍕 Pizza`, senza tempo di preparazione né link. Questo pasto non entra nei conteggi di frequenza degli ingredienti principali né nei vincoli di tempo.
3. **Contorni**: quando un pasto è un primo (da `ricette/primi/`), la ricetta principale basta da sola. Quando un pasto è un secondo (da `ricette/secondi/`), abbina sempre anche una ricetta da `ricette/contorni/` come accompagnamento — scegli un contorno coerente con il secondo e non ripetuto rispetto agli altri pasti della settimana.
4. **Frequenza degli ingredienti principali**:
   - al massimo 1 ricetta a settimana con tipologia carne bianca
   - al massimo 1 ricetta a settimana con tipologia pesce
   - al massimo 1 ricetta a settimana con tipologia uova
   - al massimo 1 ricetta ogni 3 settimane con tipologia carne rossa — controlla gli ultimi 2 file presenti in `menu/` (le due settimane precedenti, se esistono): se in uno dei due compare già una ricetta di carne rossa, non includerne un'altra questa settimana; altrimenti puoi includerne al massimo una. Se `menu/` è vuota o ha meno di 2 file, il vincolo è considerato soddisfatto e puoi includerne una.
5. **Cena nei giorni di ufficio**: solo ricette con tempo di preparazione ≤ 25 minuti per la ricetta principale. Se il pasto è un secondo e quindi include un contorno (vedi punto 3), anche il contorno deve avere un tempo di preparazione ≤ 20 minuti.
6. **Pranzo dal lunedì al venerdì**: preferisci ricette con tempo di preparazione ≤ 15 minuti. Se per varietà scegli una ricetta più lunga, va bene lo stesso, ma aggiungi una nota che suggerisce di prepararne una dose abbondante la sera prima, così a pranzo si tratta solo di riscaldare.
7. **Legumi da ammollare**: se una ricetta scelta (principale o contorno) include legumi tra gli ingredienti, considerali *sempre* legumi secchi da ammollare — non fare eccezioni anche se la ricetta li descrive come lessati, in scatola, surgelati o già pronti. Aggiungi sempre una nota nel giorno *precedente* per ricordare di metterli in ammollo.
8. **Varietà, stagionalità e ricette non preparate da tempo**: tra le ricette che rispettano i vincoli sopra, privilegia in quest'ordine di importanza:
   - varietà: evita di ripetere a distanza ravvicinata la stessa tipologia/ingrediente principale (es. non pasta a ogni pasto, non due contorni di verdure uguali in pochi giorni)
   - stagionalità: preferisci ricette con ingredienti di stagione rispetto al periodo dell'anno della settimana pianificata
   - tempo dall'ultima volta: a parità delle altre condizioni, preferisci le ricette con la data in "Ultima volta mangiato" più lontana nel passato, o non ancora compilata, rispetto a quelle mangiate di recente

## Passo 5 — Scrivi il file del piano

Il nome del file è l'intervallo di date della settimana in formato ISO, ordinabile alfabeticamente: `{lunedì AAAA-MM-GG}_{domenica AAAA-MM-GG}.md` (es. `2026-08-03_2026-08-09.md`). Non usare più numerazione incrementale.

Usa questo template per ogni giorno:

```markdown
# Menu settimanale — dal {lunedì GG/MM/AAAA} al {domenica GG/MM/AAAA}

## Lunedì {GG/MM}

- **Pranzo:** {emoji}{titolo} — {tempo} minuti — [ricetta](../ricette/{cartella}/{slug}.md)
- **Cena:** {emoji}{titolo} — {tempo} minuti — [ricetta](../ricette/{cartella}/{slug}.md)
  (se il pasto è un secondo, aggiungi su una riga subito sotto: `  + {emoji}{titolo contorno} — {tempo} minuti — [ricetta](../ricette/contorni/{slug}.md)`)

## Martedì {GG/MM}

...
```

(ripeti la struttura per tutti e 7 i giorni, nell'ordine lunedì → domenica; per venerdì la cena è `- **Cena:** 🍕 Pizza`, salvo diversa indicazione dell'utente per quella settimana)

Per un pasto non pianificato scrivi `- **Pranzo:** fuori` (o il testo indicato dall'utente) al posto della ricetta.

Subito sotto il pasto interessato, quando applicabile, aggiungi le note richieste su una riga a parte:
- `  > Nota: mettere in ammollo {legume} per la ricetta di domani.`
- `  > Nota: prepara una dose abbondante stasera, così domani a pranzo scaldi e basta.`

## Passo 6 — Salva e conferma

Salva il file in `menu/{lunedì AAAA-MM-GG}_{domenica AAAA-MM-GG}.md`. Se stai lavorando su un clone locale (caso più comune in questa sessione):
```
git add menu/{lunedì AAAA-MM-GG}_{domenica AAAA-MM-GG}.md
git commit -m "Aggiunge menu settimanale dal {lunedì GG/MM/AAAA} al {domenica GG/MM/AAAA}"
git push -u origin <branch corrente>
```
Fai sempre un `git fetch`/controllo dello stato del branch remoto prima di committare (altri file potrebbero essere stati aggiornati nel frattempo), e non forzare mai il push.

Al termine, riepiloga all'utente il piano generato (in breve, non ripetere tutto il file) e conferma il percorso dove è stato salvato.
