[# AIwatermarkRemover

**Rileva e rimuove i caratteri Unicode invisibili dai testi generati dalle AI. Una pagina HTML, nessuna dipendenza, nessun dato in rete.**

🇮🇹 [Italiano](#italiano) · 🇬🇧 [English](#english)

---

<a name="italiano"></a>

## Italiano

### Cos'è

Una singola pagina HTML che fa due cose sullo stesso testo:

1. **Verifica** — scandaglia il testo carattere per carattere e dice esattamente quali codepoint Unicode contiene che non si vedono: spazi a larghezza zero, marcatori bidirezionali, selettori di variazione, caratteri tag, omoglifi cirillici e greci, punteggiatura tipografica.
2. **Trasforma** — applica il ciclo UTF-8 → ASCII → UTF-8 nella stessa casella: rimuove ciò che è invisibile, riporta all'equivalente ASCII ciò che è tipografico, lascia intatti gli accenti.

Il ciclo è pensato per essere ripetuto: verifichi, trasformi, verifichi di nuovo e vedi i contatori andare a zero.

Non c'è build, non c'è npm, non c'è backend. Apri il file col browser e funziona, anche offline (senza rete perdi solo i font, non le funzioni).

### Perché esiste

I testi generati dai modelli linguistici, e più in generale i testi che passano da editor moderni e CMS, portano spesso caratteri Unicode che non si vedono ma occupano byte. Sono una seccatura concreta:

- rompono i diff di Git e i confronti stringa
- fanno fallire i parser di CSV e JSON
- sporcano il codice incollato dentro un IDE
- passano invisibili nei sistemi legacy che si aspettano ASCII
- nel caso dei **caratteri tag** (`U+E0000`–`U+E007F`) possono trasportare un messaggio intero dentro una frase apparentemente normale

Questo strumento li mostra e li toglie.

### Cosa segnala il referto

| Classe | Cosa contiene | Come interpretarla |
|---|---|---|
| **Nascosti** | Larghezza zero, unione parole, marcatori e isolamenti bidirezionali, selettori di variazione, caratteri tag, BOM, trattino morbido, controlli C0/C1 | In un testo digitato a mano non compaiono quasi mai. I caratteri tag in particolare non hanno usi legittimi nel testo corrente |
| **Omoglifi** | Lettere cirilliche e greche graficamente identiche alle latine (`а`, `е`, `о`, `р`, `с`, `ο`, `ν`…) | Anomale in un testo italiano, ma capitano anche col copia-incolla dal web |
| **Tipografici** | Virgolette curve, caporali, lineette en/em, puntini di sospensione, spazi unificatori e tipografici di ogni larghezza | Normalissimi in qualunque testo passato da Word o da un CMS: da soli non indicano nulla |
| **Non ASCII ok** | Accenti, lettere latine estese, simboli legittimi | Nessun sospetto. Restano intatti se non attivi la modalità ASCII rigorosa |

### Cosa NON fa

Questa sezione conta quanto le altre, perché il nome del repository promette più di quanto qualunque strumento possa mantenere.

- **Non stabilisce se un testo sia stato scritto da una persona o da un modello.** Quel giudizio oggi è statistico e sbaglia in entrambe le direzioni. Qui si contano caratteri, che o ci sono o non ci sono.
- **Non interroga nessun servizio esterno.** Non esiste, per quanto ne so, un endpoint pubblico di verifica filigrana a cui inviare il testo. I rilevatori commerciali di "testo AI" non verificano watermark: stimano lo stile.
- **Non rimuove filigrane che non siano fatte di caratteri.** Se un fornitore adottasse una filigrana statistica sulla scelta dei token — cioè codificata nel *quali parole vengono scelte*, non nei byte — questa pagina non la vedrebbe e non potrebbe toglierla. Non risulta che Anthropic ne applichi una al testo di Claude, ma è un limite di cui essere consapevoli.
- **Non rende un testo indistinguibile da uno scritto a mano.** Lo stile resta quello che è. Qui si lavora sui byte.

### Come si usa

Apri `index.html` nel browser. Oppure, se hai attivato GitHub Pages sul repository:

```
https://prinzimaker.github.io/AIwatermarkRemover/
```

1. Incolla il testo nella casella (o trascina un file `.txt`, `.md`, `.csv`, `.json`)
2. **Verifica testo AI** — o `Ctrl+Invio` — produce il referto
3. **Trasforma** — riscrive il testo pulito nella stessa casella
4. **Verifica testo AI** di nuovo — i contatori devono andare a zero

Ci sono anche **Annulla** (fino a 20 passi indietro), **Copia**, **Scarica .txt**, **Apri file**, **Svuota**.

### Opzioni

Ogni trasformazione è disattivabile singolarmente e incide anche su cosa il referto dichiara di voler fare.

| Opzione | Predefinito | Effetto |
|---|---|---|
| Rimuovi i caratteri invisibili | attiva | Elimina l'intera classe "Nascosti" |
| Normalizza gli spazi | attiva | `U+00A0`, `U+2000`–`U+200A`, `U+202F`, `U+205F`, `U+3000` → spazio normale |
| Punteggiatura tipografica → ASCII | attiva | `“ ” ‘ ’ « » — – … ′ ″` → `" ' - -- ...` |
| Correggi gli omoglifi | attiva | Lettere cirilliche/greche → latine. **Disattivala se il testo è davvero in russo o greco** |
| Normalizzazione NFKC | attiva | Scioglie legature, larghezza piena, apici e indici |
| Ripulisci le righe | attiva | Fine riga a LF, spazi rimossi in coda, righe vuote multiple compattate |
| Solo ASCII, senza eccezioni | **spenta** | Toglie anche gli accenti (`perché` → `perche`). In italiano di solito non lo vuoi |
| Rimuovi le emoji | **spenta** | Elimina pittogrammi e simboli |

### Privacy

Tutto avviene nel browser. Nessuna chiamata di rete, nessun log, nessuna telemetria, nessun `localStorage`. L'unica risorsa esterna è il foglio di stile dei font Google, e la pagina resta pienamente funzionante se non è raggiungibile.

Per un uso in ambiente chiuso, basta scaricare i font in locale e sostituire il `<link>` nell'`<head>`.

### Estendere

Le due strutture da toccare sono in cima allo script, il resto del motore non va modificato:

```js
const HOMO = new Map([
  [0x0430,"a"], [0x0435,"e"], [0x043E,"o"], // …aggiungi qui gli omoglifi
]);

const PUNCT = new Map([
  [0x2014,"--"], [0x2026,"..."],            // …aggiungi qui le sostituzioni ASCII
]);
```

Per aggiungere un carattere alla classe "Nascosti" basta inserirne il codepoint nel `Set` `INVISIBLE`, oppure un intervallo nella funzione `isInvisible()`.

### Compatibilità

Browser moderni con supporto a `String.prototype.normalize`, alle proprietà Unicode nelle espressioni regolari (`\p{M}` con flag `u`) e alla Clipboard API. In pratica: Chrome, Edge, Firefox e Safari recenti. Se la lettura degli appunti è negata dal browser, si incolla a mano con `Ctrl+V`.

### Licenza

MIT — vedi [LICENSE](LICENSE).

### Autore

**Aldo Prinzi** — [aldo.prinzi.it](https://aldo.prinzi.it)

---

<a name="english"></a>

## English

### What it is

A single HTML page that does two things to the same text:

1. **Inspect** — walks the text codepoint by codepoint and reports exactly which invisible Unicode characters it contains: zero-width characters, bidirectional marks, variation selectors, tag characters, Cyrillic and Greek homoglyphs, typographic punctuation.
2. **Transform** — applies a UTF-8 → ASCII → UTF-8 pass in place: strips what is invisible, folds typographic characters to their ASCII equivalent, leaves accented letters alone.

The loop is meant to be repeated: inspect, transform, inspect again, and watch the counters drop to zero.

No build step, no npm, no backend. Open the file in a browser and it works, offline included (without a connection you only lose the webfonts, not the functionality).

### Why it exists

Text produced by language models — and more generally any text that passes through a modern editor or CMS — often carries Unicode characters that are invisible but occupy bytes. They cause real problems:

- they break Git diffs and string comparisons
- they make CSV and JSON parsers fail
- they pollute code pasted into an IDE
- they slip unnoticed into legacy systems that expect ASCII
- in the case of **tag characters** (`U+E0000`–`U+E007F`) they can carry an entire hidden message inside an ordinary-looking sentence

This tool shows them and removes them.

### What the report tells you

| Class | What it covers | How to read it |
|---|---|---|
| **Hidden** | Zero-width, word joiner, bidi marks and isolates, variation selectors, tag characters, BOM, soft hyphen, C0/C1 controls | Almost never present in hand-typed text. Tag characters in particular have no legitimate use in running prose |
| **Homoglyphs** | Cyrillic and Greek letters visually identical to Latin ones (`а`, `е`, `о`, `р`, `с`, `ο`, `ν`…) | Anomalous in Western-language text, but also a common artefact of copy-pasting from the web |
| **Typographic** | Curly quotes, guillemets, en/em dashes, ellipses, non-breaking and typographic spaces | Perfectly normal in anything that passed through Word or a CMS: on their own they mean nothing |
| **Legitimate non-ASCII** | Accents, extended Latin letters, valid symbols | Nothing suspicious. Left untouched unless strict ASCII mode is on |

### What it does NOT do

This section matters as much as the others, because the repository name promises more than any tool can deliver.

- **It does not determine whether a text was written by a human or a model.** That judgement is statistical today and errs in both directions. Here we count characters, which are either present or absent.
- **It does not call any external service.** As far as I know, no public watermark-verification endpoint exists to send text to. Commercial "AI text detectors" do not verify watermarks: they estimate style.
- **It cannot remove a watermark that is not made of characters.** If a provider adopted a statistical watermark over token selection — encoded in *which words are chosen*, not in the bytes — this page would neither see it nor be able to strip it. Anthropic is not known to apply one to Claude's text, but the limitation is worth stating.
- **It does not make text indistinguishable from human writing.** Style stays what it is. This works on bytes.

### Usage

Open `index.html` in a browser, or, with GitHub Pages enabled:

```
https://prinzimaker.github.io/AIwatermarkRemover/
```

1. Paste your text (or drop a `.txt`, `.md`, `.csv`, `.json` file on the page)
2. **Verifica testo AI** — or `Ctrl+Enter` — produces the report
3. **Trasforma** — rewrites the cleaned text in the same box
4. **Verifica testo AI** again — the counters should read zero

There are also **Undo** (up to 20 steps), **Copy**, **Download .txt**, **Open file** and **Clear**.

> **Note:** the interface is currently in Italian. Contributions adding an English UI are welcome.

### Options

Every transformation can be toggled individually, and the setting also changes what the report says it would do.

| Option | Default | Effect |
|---|---|---|
| Remove invisible characters | on | Strips the entire "Hidden" class |
| Normalise spaces | on | `U+00A0`, `U+2000`–`U+200A`, `U+202F`, `U+205F`, `U+3000` → plain space |
| Typographic punctuation → ASCII | on | `“ ” ‘ ’ « » — – … ′ ″` → `" ' - -- ...` |
| Fix homoglyphs | on | Cyrillic/Greek → Latin. **Turn it off if the text really is Russian or Greek** |
| NFKC normalisation | on | Resolves ligatures, fullwidth forms, superscripts and subscripts |
| Clean up lines | on | LF line endings, trailing whitespace removed, multiple blank lines collapsed |
| Strict ASCII | **off** | Also strips accents (`perché` → `perche`) |
| Remove emoji | **off** | Strips pictographs and symbols |

### Privacy

Everything runs in the browser. No network calls, no logging, no telemetry, no `localStorage`. The only external resource is the Google Fonts stylesheet, and the page remains fully functional when it is unreachable.

For air-gapped use, download the fonts locally and replace the `<link>` in the `<head>`.

### Extending

The two structures to edit sit at the top of the script; the rest of the engine needs no changes:

```js
const HOMO = new Map([
  [0x0430,"a"], [0x0435,"e"], [0x043E,"o"], // …add homoglyphs here
]);

const PUNCT = new Map([
  [0x2014,"--"], [0x2026,"..."],            // …add ASCII substitutions here
]);
```

To add a character to the "Hidden" class, put its codepoint in the `INVISIBLE` set, or add a range to `isInvisible()`.

### Compatibility

Modern browsers with `String.prototype.normalize`, Unicode property escapes in regular expressions (`\p{M}` with the `u` flag) and the Clipboard API. In practice: recent Chrome, Edge, Firefox and Safari. If clipboard reading is denied, paste manually with `Ctrl+V`.

### License

MIT — see [LICENSE](LICENSE).

### Author

**Aldo Prinzi** — [aldo.prinzi.it](https://aldo.prinzi.it)
](https://github.com/prinzimaker/AIwatermarkRemover/blob/main/README.md)
