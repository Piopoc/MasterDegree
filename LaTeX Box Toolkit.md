---
type: reference
date: 2026-10-10
tags: [reference, latex, mathjax, obsidian, formatting]
ai-first: true
---

# LaTeX Box Toolkit

> **For future agent:** reference note explaining how to build framed "boxes with a caption" inside
> Obsidian notes using MathJax (obsidian-latex-suite, already installed in this vault). It documents
> the `\bbox` pattern: a colored frame with custom left/right header labels and a centered body,
> where the border style, width and color are real CSS and therefore freely modifiable. Read this
> BEFORE attempting any framed-box LaTeX in this vault. MathJax is not LaTeX: `\fcolorbox` and
> `\colorbox` are BROKEN in this version, `\newcommand` cannot wrap `\bbox`, and `\multicolumn` does
> not exist. Section 1 is the fast answer to "why doesn't my box work".

---

## 1. Cosa funziona e cosa no

Obsidian renderizza la matematica con **MathJax 3**, non con LaTeX. Non ci sono pacchetti da
installare. La tabella e' stata verificata renderizzando ogni costrutto, non presa da documentazione.

| Cosa | Funziona? | Note |
|---|---|---|
| `\require{bbox}` poi `\bbox[stili]{...}` | **si** | e' il costrutto da usare per i box |
| `border:2px solid #2F6FEB` nello stile | si | stile CSS: colore esadecimale diretto, nessun `\definecolor` |
| `dashed`, `dotted`, `double`, `groove` | si | il "motivo" della cornice, vero |
| `padding`, `color` | si | padding interno e colore del testo |
| `border-radius` | **NO** | viene ignorato: gli angoli restano squarati |
| `\begin{array}{ccc}` e array annidati | si | e' il modo in cui si costruisce il layout interno |
| `\hline`, `\hdashline` | si | separatori a tutta larghezza dell'array |
| `\phantom{...}` | si | serve a centrare bene il corpo |
| `\text{}`, `\textbf{}`, `\textit{}` | si | **tutto il testo va dentro `\text{}`** |
| `\fcolorbox`, `\colorbox` | **NO** | renderizzano il sorgente come testo letterale |
| `\newcommand` che avvolge `\bbox` | **NO** | "Math input error", anche senza `#` |
| `#` (hash) dentro una `\newcommand` | **NO** | "Illegal macro parameter reference" |
| `\multicolumn` | **NO** | mai implementato in MathJax |
| wrapping automatico del testo | **NO** | ogni riga va chiusa a mano con `\\` |
| `\definecolor{x}{HTML}{2F6FEB}` | **NO** | il modello colore `HTML` non esiste: usa `RGB` |
| miscele di colore `green!50!black` | **NO** | rendono **nero** senza avvisare |
| `minipage`, `parbox`, `tcolorbox`, `mdframed` | **NO** | pacchetti TeX puri, non caricabili |
| un dollaro dentro un blocco display | **NO** | chiuderebbe il blocco; per formule annidate si resta in math mode |

Le tre estensioni utili sono gia' precaricate da Obsidian (verificato nel binario
`obsidian.asar`: `MathJax.loader.preLoad("[tex]/bbox", ...)`), quindi `\bbox` e `\hdashline`
funzionano senza configurare nulla. Se per caso non fossero caricate, `\require{bbox}` all'inizio
del blocco le carica.

Conseguenza pratica: **non esiste il wrapping**, quindi un box con molto testo e' scomodo da
scrivere. Meglio un box corto (una definizione, un lemma, un avvertimento) che un paragrafo.

---

## 2. Il pattern

```latex
$$
\bbox[border:2px solid #2F6FEB; background:#F4F8FF; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Definizione 1.1} && \textit{Macrodata} \\
\hline
& \begin{array}{c} \text{riga uno} \\ \text{riga due} \end{array} &
\end{array}
}
$$
```

Ogni box e' **un blocco autonomo**: nessuna macro, nessun preambolo da ricordarsi. Si copia,
si incolla e si cambiano tre cose: gli stili, le due etichette, il corpo.

Risultato:

$$
\bbox[border:2px solid #2F6FEB; padding:10px]{
\begin{array}{ccc}
\textbf{Definizione 1.1} && \textit{Macrodata} \\
\hline
& \begin{array}{c} \text{Dati aggregati relativi a intere popolazioni,} \\ \text{non riferibili a singoli individui.} \end{array} &
\end{array}
}
$$

### Anatomia

L'idea e' un array a **tre colonne**. La prima riga occupa le colonne 1 e 3 (le due etichette
si agganciano ai bordi sinistro e destro), il `\hline` attraversa tutta la larghezza, e il corpo
va nella colonna centrale, che e' centrata rispetto al box.

```
+-------------------------------------------------+
|  etichetta sinistra            etichetta destra  |   <- riga 1, colonne 1 e 3
+-------------------------------------------------+   <- \hline
|                                                 |
|                    corpo                       |   <- riga 2, colonna 2
|                                                 |
+-------------------------------------------------+
```

L'array annidato `\begin{array}{c} ... \end{array}` serve solo a dare al corpo piu' righe: e' un
array dentro la cella centrale. Se il corpo e' una riga sola, si puo' scrivere direttamente
`& \text{corpo} &` e risparmiare un livello di annidamento.

---

## 3. Palette

I colori si scrivono **in esadecimale dentro gli stili**, quindi non serve alcun `\definecolor`.
Coppia cornice/fondo da usare nei preset.

| Preset | `border` | `background` | Uso tipico |
|---|---|---|---|
| Definizione | `2px solid #2F6FEB` | `#F4F8FF` | concetto da memorizzare |
| Lemma | `2px dashed #15803D` | `#F2FBF5` | teorema con dimostrazione breve |
| Teorema | `3px double #7C3AED` | `#F8F5FF` | risultato forte |
| Avvertenza | `2px dotted #B45309` | `#FFF8EC` | trappola d'esame, fonte da rileggere |
| Attenzione | `2px solid #B91C1C` | `#FEF3F3` | errore frequente, sbaglio da non fare |
| Esempio | `2px solid #475569` | `#F6F7F9` | caso concreto, controesempio |
| Neutro | `background:#F6F7F9` | (nessun bordo) | box generico, solo sfondo |

Il parametro `color:#111827` imposta il colore del testo. Serve in **tema scuro**: senza, il
testo eredita il colore chiaro del tema e diventa illeggibile su uno sfondo chiaro. In tema
chiaro puo' essere omesso.

---

## 4. I preset pronti

### 4.1 Definizione

$$
\bbox[border:2px solid #2F6FEB; background:#F4F8FF; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Definizione 2.1} && \textit{Microdata} \\
\hline
& \begin{array}{c} \text{Record individuali rilasciati cosi' com'è, quindi}\\\text{identificabili direttamente.} \end{array} &
\end{array}
}
$$

### 4.2 Lemma

$$
\bbox[border:2px dashed #15803D; background:#F2FBF5; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Lemma 2.2} && \textit{Cap. 3} \\
\hline
& \begin{array}{c} \text{Se la domanda e' aggregata e il rilascio e' non ambiguo,}\\\text{allora il rilascio e' sicuro.} \end{array} &
\end{array}
}
$$

### 4.3 Teorema con formula

$$
\bbox[border:3px double #7C3AED; background:#F8F5FF; color:#111827; padding:12px]{
\begin{array}{ccc}
\textbf{Formato} && \textit{n. 1.4} \\
\hline
& \begin{array}{c} P_{n,k} = \dfrac{n!}{k!\,(n-k)!} \\ \text{con } 0 \le k \le n \end{array} &
\end{array}
}
$$

### 4.4 Avvertenza

$$
\bbox[border:2px dotted #B45309; background:#FFF8EC; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Attenzione} && \textit{Da sapere} \\
\hdashline
& \begin{array}{c} \text{Macrodata e microdata hanno regole di protezione}\\\text{diverse: non sono la stessa cosa.} \end{array} &
\end{array}
}
$$

### 4.5 Attenzione (rosso, per gli errori da non fare)

$$
\bbox[border:2px solid #B91C1C; background:#FEF3F3; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Errore classico} && \textit{Esame 2024} \\
\hline
& \begin{array}{c} \text{Confondere } (n,k)\text{-anonymity con } k\text{-anonymity:}\\\text{la } (n,k)\text{-richiede che ogni classe ne contenga almeno } n. \end{array} &
\end{array}
}
$$

---

## 5. Gli stili della cornice

La parte modificabile per eccellenza: tutto quello che sta dentro `\bbox[...]` e' CSS puro.

| Proprieta' | Valori utili | Effetto |
|---|---|---|
| `border` | `2px solid`, `2px dashed`, `2px dotted`, `3px double`, `4px groove` | il "motivo" della cornice |
| spessore | `1px` sottile, `2px` normale, `3px` marcato |Cornice piu' o meno pesante |
| colore bordo | `#RRGGBB` | il colore della cornice |
| `background` | `#RRGGBB` | il fondo del box |
| `color` | `#111827` | colore del testo (obbligatorio in tema scuro) |
| `padding` | `8px` stretto, `12px` ampio | spaziatura interna |
| `border-radius` | non ha effetto | ignorato dal renderer, angoli sempre squarati |

Attenzione a **usare il punto e virgola `;` per separare le proprieta'**. La virgola `,` separa
gli argomenti di `\bbox`, quindi `border:2px solid red, background:#fff` funziona, ma
`border:2px solid #2F6FEB; background:#F4F8FF; padding:10px` e' la forma piu' comoda perche' ti
permette di scrivere anche i valori RGB con virgola senza conflitti.

### 5.1 Stili di bordo a confronto

Bordo continuo, 2px:

$$
\bbox[border:2px solid #2F6FEB; background:#F4F8FF; color:#111827; padding:8px]{
\begin{array}{ccc}\textbf{solid} && \textit{2px}\\\hline&\text{cornice continua}&\end{array}
}
$$

Bordo tratteggiato, 2px:

$$
\bbox[border:2px dashed #15803D; background:#F2FBF5; color:#111827; padding:8px]{
\begin{array}{ccc}\textbf{dashed} && \textit{2px}\\\hline&\text{cornice tratteggiata}&\end{array}
}
$$

Bordo punteggiato, 2px:

$$
\bbox[border:2px dotted #B45309; background:#FFF8EC; color:#111827; padding:8px]{
\begin{array}{ccc}\textbf{dotted} && \textit{2px}\\\hline&\text{cornice punteggiata}&\end{array}
}
$$

Bordo doppio, 3px:

$$
\bbox[border:3px double #7C3AED; background:#F8F5FF; color:#111827; padding:8px]{
\begin{array}{ccc}\textbf{double} && \textit{3px}\\\hline&\text{cornice doppia}&\end{array}
}
$$

Solo sfondo, nessun bordo:

$$
\bbox[background:#F6F7F9; color:#111827; padding:8px]{
\begin{array}{ccc}\textbf{neutro} && \textit{nessun bordo}\\\hline&\text{solo sfondo}&\end{array}
}
$$

---

## 6. Il corpo del box

### 6.1 Multilinea

Ogni riga del corpo corrisponde a una riga dell'array annidato, chiusa con `\\`.

```latex
& \begin{array}{c} \text{riga uno} \\ \text{riga due} \\ \text{riga tre} \end{array} &
```

### 6.2 Formule dentro il corpo

Il corpo e' gia' in modalita' matematica, quindi le formule si scrivono nude. Il testo intorno
va avvolto in `\text{}`. Le frazioni usano `\dfrac` per non diventare minuscole.

```latex
& \begin{array}{c} P_{n,k} = \dfrac{n!}{k!\,(n-k)!} \\ \text{con } 0 \le k \le n \end{array} &
```

### 6.3 Elenchi

MathJax non fa le liste. Si simula con un prefisso `\text{- }` e un `\\` per voce.

```latex
& \begin{array}{c} \text{- Definire lo schema relazionale} \\ \text{- Scegliere le chiavi primarie} \\ \text{- Verificare le dipendenze} \end{array} &
```

### 6.4 Centrare bene il corpo

La colonna centrale e' larga quanto il corpo, ma e' centrata **fra le due colonne delle
etichette**, quindi se le etichette hanno lunghezze diverse il corpo risulta leggermente
spostato. Si corregge con `\phantom{}`, che occupa spazio senza mostrare nulla: si mette nel
lato piu' corto una stringa invisibile lunga quanto l'altra etichetta.

```latex
\textbf{Passi}\phantom{\text{xxxxxxxxxx}} && \textit{v. 2.1} \\
```

### 6.5 Larghezza minima

Per evitare un box stretto attorno a una parola, si aggiunge al corpo una riga invisibile di
larghezza fissa: `\rule{20em}{0pt}` ha larghezza ma altezza zero.

```latex
& \begin{array}{c} \rule{20em}{0pt} \\ \text{Questo e' un esempio piuttosto breve.} \end{array} &
```

---

## 7. Ricette pronte per il vault

### Nota di studio, con box definizione

```latex
$$
\bbox[border:2px solid #2F6FEB; background:#F4F8FF; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Definizione} && \textit{voce di esempio} \\
\hline
& \begin{array}{c} \text{corpo della definizione, righe spezzate} \\ \text{con doppio backslash} \end{array} &
\end{array}
}
$$
```

### Box che collega a un'altra nota

```latex
$$
\bbox[border:2px solid #475569; background:#F6F7F9; color:#111827; padding:10px]{
\begin{array}{ccc}
\textbf{Cross-link} && \textit{vedi anche} \\
\hline
& \text{Questo argomento tocca anche la nota gemella.} &
\end{array}
}
$$
```

La fonte della nota **non va nel box**: va una sola volta nel footer `## Sources` della nota
(regole del vault 2026-10-15). Nel corpo di un box i wikilink `[[...]]` **non diventano cliccabili**: dentro `\text{}` vengono
resi come testo semplice. Se serve un link cliccabile, mettilo **fuori** dal box, subito sotto.

---

## 8. Quando il box LaTeX e' la scelta sbagliata

Il box serve per contenuti brevi (una definizione, un lemma, un avvertimento). Non usarlo per:

- paragrafi lunghi: non c'e' wrapping, il testo esce dal box
- testo con wikilink cliccabili
- tabelle: la sintassi markdown nativa e' migliore
- note interattive: per callout `> [!note]` o liste di task usa la sintassi Obsidian
- testo che deve adattarsi al tema: i colori qui sono fissi e non cambiano con il tema

Se serve un box con paragrafo, testo che va a capo da solo e link cliccabili, il percorso giusto
non e' LaTeX ma un **callout Obsidian** con CSS personalizzato.