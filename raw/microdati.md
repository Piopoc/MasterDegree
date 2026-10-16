Per garantire la privacy nel rilascio dei **microdati** (dati relativi ai singoli individui), le fonti presentano una serie di tecniche strutturate che bilanciano la protezione dell'identità degli utenti e il mantenimento delle proprietà statistiche del dataset.

La prima operazione preliminare è la **sanitizzazione (sanitizing)**, che consiste nella rimozione degli identificatori diretti ed espliciti come nomi, indirizzi, codici fiscali o numeri di telefono. Per proteggere i dati dai rischi di re-identificazione legati ai quasi-identificatori o agli attributi sensibili, si utilizzano due macro-categorie principali: il **mascheramento (masking)** e la **generazione di dati sintetici**.

---

### 1. Mascheramento Non Perturbativo (Non-perturbative Masking)

Queste tecniche riducono il dettaglio o eliminano osservazioni dai microdati originali senza alterare i valori esistenti con dati falsi o modificati:

- **Sampling (Campionamento)**: rimuove alcune osservazioni dal dataset secondo una politica detta _"Just a few"_, riducendo la probabilità che un individuo specifico sia presente nel campione rilasciato.
- **Local Suppression (Soppressione locale)**: sostituisce con un valore mancante (_"missing value"_) le celle o gli attributi specifici che aumentano il rischio di divulgazione.
- **Global Recoding (Ricodifica globale)**: suddivide il dominio di un attributo in intervalli disgiunti e sostituisce i valori puntuali con l'etichetta dell'intervallo corrispondente.
    - **Top-coding**: variante che sostituisce tutti i valori superiori a una determinata soglia con un valore limite (ad esempio, sostituendo le altezze superiori a 1.9m con l'etichetta `">1.9m"`).
    - **Bottom-coding**: variante speculare che converte i valori inferiori a una soglia (ad esempio, altezze inferiori a 1.7m con `"<1.7m"`).
- **Generalization (Generalizzazione)**: sfrutta una gerarchia di concetti per sostituire gli attributi con una rappresentazione più generale (ad esempio, indicando solo l'anno o il mese di nascita anziché la data completa, oppure solo una parte del CAP).

---

### 2. Mascheramento Perturbativo (Perturbative Masking)

Queste tecniche alterano o modificano i dati originali introducendo rumore, combinazioni differenti o variazioni nei valori:

- **Rounding (Arrotondamento)**: a differenza della ricodifica globale, definisce degli intervalli basati su un passo \(b\) e punti \(p_i\), e sostituisce il valore con un punto specifico contenuto all'interno dell'intervallo.
- **Resampling (Ricampionamento)**: effettua un ricampionamento con reinserimento della colonna da mascherare per più volte, ordina i valori all'interno di ciascun campione e li riassocia alle tuple.
- **Lossy Compression (Compressione con perdita)**: tratta la tabella dei dati come se fosse un'immagine e le applica un algoritmo di compressione con perdita (come JPEG); il tasso di compressione definisce il parametro di offuscamento.
- **PRAM (Post RAndomized Method)**: cambia in modo probabilistico le etichette degli attributi categoriali sulla base di una matrice di distribuzione di probabilità appositamente definita.
- **Random Noise (Rumore casuale)**: aggiunge rumore additivo estratto da una distribuzione; può essere _decorrelato_ oppure _correlato_ (tenendo conto della matrice di covarianza degli attributi).
- **Swapping (Scambio)**: scambia in modo casuale i valori degli attributi sensibili tra le tuple.
- **Rank Swapping**: variante dello swapping che limita lo scambio ai soli valori compresi entro una distanza massima \(p\) definita dal parametro di offuscamento.
- **Micro-aggregation / Blurring (Micro-aggregazione)**: raggruppa le tuple in cluster basati sulla massima similarità, calcola la media dell'attributo sensibile per ciascun cluster e sostituisce il valore originale di ogni tupla con la media calcolata.

---

### 3. Generazione di Dati Sintetici (Synthetic Data Generation)

Invece di limitarsi a mascherare i dati reali, questa categoria crea dati del tutto o parzialmente nuovi:

- **Fully Synthetic (Completamente sintetico)**: genera un dataset del tutto nuovo che rispecchia la struttura originale.
    - **Decomposizione di Cholesky**: utilizzata per dati continui, calcola la decomposizione di Cholesky della matrice di covarianza \(C = U^T \times U\) e moltiplica \(U\) per una matrice \(R\) di valori casuali (con matrice di covarianza pari all'identità), creando un dataset sintetico che preserva la covarianza originale.
- **Perturbative / Partially Synthetic (Parzialmente sintetico)**: genera dati sintetici solo per alcuni attributi o tuple e li combina con i dati reali.
    - **Random Response (Risposta casuale)**: tecnica impiegata nel framework della Differential Privacy dove si chiede al soggetto di rispondere a una domanda tra un insieme di domande (molte non sensibili e una sensibile) senza indicare quale ha scelto, permettendo di stimare la distribuzione complessiva.
    - **Blank and Impute (Oscuramento e Imputazione)**: cancella gli attributi sensibili per un sottoinsieme casuale di tuple e riempie le celle svuotate tramite imputazione (ad esempio inserendo la media o la categoria più frequente delle tuple più simili).