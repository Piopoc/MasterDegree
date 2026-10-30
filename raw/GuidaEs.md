# Guida Procedurale e Metodologica alla Risoluzione di Esercizi di Teoria dei Giochi

### 1. Quadro Metodologico Generale per i Giochi Statici a Informazione Completa

Un gioco statico a informazione completa in forma normale (o strategica) rappresenta un modello matematico di interazione simultanea tra agenti razionali. Formalmente, un gioco statico a n giocatori è definito dalla tripla:

G = \left\langle N, (S_i)_{i \in N}, (u_i)_{i \in N} \right\rangle

dove:

- **Giocatori (****N****)**: L'insieme finito degli agenti decisionali razionali, indicato con N = \{1, 2, \dots, n\}. Nei giochi a due giocatori, gli attori sono convenzionalmente denotati come A (Giocatore 1, riga) e B (Giocatore 2, colonna).
- **Strategie Pure (****S_i****)**: L'insieme discreto delle azioni deterministiche a disposizione del giocatore i \in N. Un profilo di strategie pure è una n-upla s = (s_1, s_2, \dots, s_n) \in S, dove S = \prod_{i \in N} S_i rappresenta lo spazio delle strategie congiunte.
- **Strategie Miste (****\Delta(S_i)****)**: Il simplesso di probabilità definito sull'insieme delle strategie pure S_i. Una strategia mista per il giocatore i è una distribuzione di probabilità m_i \in \Delta(S_i) tale che \sum_{s_i \in S_i} m_i(s_i) = 1 e m_i(s_i) \ge 0. Il supporto di m_i, denotato con \text{supp}(m_i), è l'insieme delle strategie pure a cui m_i assegna una probabilità strettamente positiva.
- **Funzioni di Utilità e Payoff Atteso (****u_i, U_i****)**: La funzione u_i: S \to \mathbb{R} associa a ogni profilo di strategie pure s \in S un valore numerico reale rappresentativo del benessere dell'agente i. In presenza di strategie miste m = (m_1, \dots, m_n), l'utilità attesa del giocatore i è data da: U_i(m) = \sum_{s \in S} \left( \prod_{j \in N} m_j(s_j) \right) u_i(s)
- **Risposta Ottima (Best Response -** **\text{BR}_i****)**: Per un dato profilo di strategie degli avversari s_{-i}, la risposta ottima del giocatore i è la strategia (pura o mista) che massimizza la sua utilità: \text{BR}_i(s_{-i}) = \arg\max_{s_i \in S_i} u_i(s_i, s_{-i})
- **Equilibrio di Nash in Strategie Pure**: Un profilo di strategie pure s^* = (s_1^*, \dots, s_n^*) \in S costituisce un Equilibrio di Nash se ciascuna componente è una risposta ottima alle scelte degli altri giocatori: \forall i \in N, \quad u_i(s_i^*, s_{-i}^*) \ge u_i(s_i, s_{-i}^*), \quad \forall s_i \in S_i
- **Equilibrio di Nash in Strategie Miste e Principio di Indifferenza**: Un profilo di strategie miste m^* \in \Delta(S) è un Equilibrio di Nash se nessun giocatore può aumentare la propria utilità attesa deviando unilateralmente. Nel caso di equilibri misti interni (con supporto non singoletto), si applica il **Principio di Indifferenza**: se una strategia mista m_i^* assegna probabilità strettamente positiva a due o più strategie pure, tali strategie pure devono garantire al giocatore i il medesimo payoff atteso dato m_{-i}^*.
- **Allocazione Pareto-Ottimale**: Un profilo di strategie s \in S è Pareto-ottimale (o non dominato ai sensi di Pareto) se non esiste alcun altro profilo s' \in S tale per cui u_i(s') \ge u_i(s) per tutti gli i \in N, con almeno un'uguaglianza stretta u_j(s') > u_j(s) per un giocatore j.

### 2. Risoluzione dei Giochi Statici 2x2 in Forma Normale (Esercizio 1)

#### 2.1 Metodologia di Analisi per Matrici 2x2

Per la determinazione sistematica degli Equilibri di Nash in matrici di payoff 2 \times 2, l'algoritmo procedurale si articola come segue:

1. **Analisi delle Risposte Ottime del Giocatore A (Righe)**:
    - Fissare la strategia del Giocatore B sulla prima colonna (f) e identificare la riga che massimizza il payoff di A.
    - Fissare la strategia del Giocatore B sulla seconda colonna (g) e identificare la riga che massimizza il payoff di A.
2. **Analisi delle Risposte Ottime del Giocatore B (Colonne)**:
    - Fissare la strategia del Giocatore A sulla prima riga (F) e identificare la colonna che massimizza il payoff di B.
    - Fissare la strategia del Giocatore A sulla seconda riga (G) e identificare la colonna che massimizza il payoff di B.
3. **Identificazione degli Equilibri Puri**:
    - Le celle in cui si verifica la coincidenza delle risposte ottime di entrambi i giocatori rappresentano gli Equilibri di Nash in strategie pure.
4. **Calcolo dell'Equilibrio Misto**:
    - Posto m_A = (p, 1-p) con p = P(A = F) e m_B = (q, 1-q) con q = P(B = f), si impostano le equazioni di indifferenza U_B(F, m_B) = U_B(G, m_B) per determinare q^*, e U_A(m_A, f) = U_A(m_A, g) per determinare p^*. Se i valori ottenuti soddisfano il vincolo p^*, q^* \in (0, 1), il profilo (m_A, m_B) costituisce un equilibrio misto interno.

#### 2.2 Applicazione e Risoluzione dei Casi Studio (a, b, c, d)

##### Caso (a)

**Matrice dei Payoff:**

|   |   |   |
|---|---|---|
|A \ B|f|g|
|**F**|2, 4|0, 1|
|**G**|1, 6|3, 5|

**Iter logico di risoluzione:**

1. _Risposte ottime di A_:
    - Dato B = f: u_A(F, f) = 2 > 1 = u_A(G, f) \implies \text{BR}_A(f) = F.
    - Dato B = g: u_A(G, g) = 3 > 0 = u_A(F, g) \implies \text{BR}_A(g) = G.
2. _Risposte ottime di B_:
    - Dato A = F: u_B(F, f) = 4 > 1 = u_B(F, g) \implies \text{BR}_B(F) = f.
    - Dato A = G: u_B(G, f) = 6 > 5 = u_B(G, g) \implies \text{BR}_B(G) = f. _(Nota: La strategia_ _f_ _domina strettamente la strategia_ _g_ _per il Giocatore B)._
3. _Incrocio delle Risposte Ottime_:
    - Cella (F, f): F \in \text{BR}_A(f) e f \in \text{BR}_B(F).
4. _Verifica dell'assenza di Equilibri Misti interni_: Sia m_A = (p, 1-p) la strategia mista di A. Impostando l'uguaglianza dei payoff attesi per le due strategie pure di B: U_B(m_A, f) = U_B(m_A, g) \implies 4p + 6(1-p) = 1p + 5(1-p) \implies 6 - 2p = 5 - 4p \implies 2p = -1 \implies p^* = -\frac{1}{2} Poiché p^* = -1/2 \notin [0, 1], la condizione necessaria dell'indifferenza non può essere soddisfatta da alcuna distribuzione di probabilità ammissibile. Ciò dimostra rigorosamente che non esiste alcun equilibrio di Nash in strategie miste a supporto completo.

- **Insieme degli Equilibri di Nash**: NE = \{(F, f)\} con payoff (2, 4).

##### Caso (b)

**Matrice dei Payoff:**

|   |   |   |
|---|---|---|
|A \ B|f|g|
|**F**|0, 4|3, 0|
|**G**|6, 0|0, 5|

**Iter logico di risoluzione:**

1. _Risposte ottime di A_:
    - Dato B = f: u_A(G, f) = 6 > 0 = u_A(F, f) \implies \text{BR}_A(f) = G.
    - Dato B = g: u_A(F, g) = 3 > 0 = u_A(G, g) \implies \text{BR}_A(g) = F.
2. _Risposte ottime di B_:
    - Dato A = F: u_B(F, f) = 4 > 0 = u_B(F, g) \implies \text{BR}_B(F) = f.
    - Dato A = G: u_B(G, g) = 5 > 0 = u_B(G, f) \implies \text{BR}_B(G) = g.
3. _Incrocio delle Risposte Ottime_:
    - Nessuna cella soddisfa la condizione di reciprocità della risposta ottima. Non esistono equilibri di Nash in strategie pure.
4. _Calcolo dell'Equilibrio in Strategie Miste_:
    - Siano m_A = (p, 1-p) e m_B = (q, 1-q).
    - _Indifferenza del Giocatore A_: U_A(F, m_B) = U_A(G, m_B) 0 \cdot q + 3(1-q) = 6 \cdot q + 0(1-q) \implies 3 - 3q = 6q \implies 9q = 3 \implies q^* = \frac{1}{3}
    - _Indifferenza del Giocatore B_: U_B(m_A, f) = U_B(m_A, g) 4 \cdot p + 0(1-p) = 0 \cdot p + 5(1-p) \implies 4p = 5 - 5p \implies 9p = 5 \implies p^* = \frac{5}{9}

- **Insieme degli Equilibri di Nash**: NE = \left\{ \left( m_A = \left(\frac{5}{9}, \frac{4}{9}\right), m_B = \left(\frac{1}{3}, \frac{2}{3}\right) \right) \right\}.

##### Caso (c)

**Matrice dei Payoff:**

|   |   |   |
|---|---|---|
|A \ B|f|g|
|**F**|9, 3|2, 2|
|**G**|0, 0|3, 9|

**Iter logico di risoluzione:**

1. _Risposte ottime di A_:
    - Dato B = f: \text{BR}_A(f) = F (9 > 0).
    - Dato B = g: \text{BR}_A(g) = G (3 > 2).
2. _Risposte ottime di B_:
    - Dato A = F: \text{BR}_B(F) = f (3 > 2).
    - Dato A = G: \text{BR}_B(G) = g (9 > 0).
3. _Incrocio delle Risposte Ottime_:
    - Le celle (F, f) e (G, g) costituiscono due equilibri in strategie pure.
4. _Calcolo dell'Equilibrio in Strategie Miste_:
    - Siano m_A = (p, 1-p) e m_B = (q, 1-q).
    - _Indifferenza per A_: U_A(F, m_B) = U_A(G, m_B) \implies 9q + 2(1-q) = 0q + 3(1-q) \implies 7q + 2 = 3 - 3q \implies 10q = 1 \implies q^* = \frac{1}{10}.
    - _Indifferenza per B_: U_B(m_A, f) = U_B(m_A, g) \implies 3p + 0(1-p) = 2p + 9(1-p) \implies 3p = 9 - 7p \implies 10p = 9 \implies p^* = \frac{9}{10}.

- **Insieme degli Equilibri di Nash**: NE = \left\{ (F, f), (G, g), \left( m_A = \left(\frac{9}{10}, \frac{1}{10}\right), m_B = \left(\frac{1}{10}, \frac{9}{10}\right) \right) \right\}.

##### Caso (d)

**Matrice dei Payoff:**

|   |   |   |
|---|---|---|
|A \ B|f|g|
|**F**|2, 2|0, 6|
|**G**|6, 0|1, 1|

**Iter logico di risoluzione:**

1. _Risposte ottime di A_:
    - Dato B = f: \text{BR}_A(f) = G (6 > 2).
    - Dato B = g: \text{BR}_A(g) = G (1 > 0). _(La strategia_ _G_ _domina strettamente la strategia_ _F_ _per l'attore A)._
2. _Risposte ottime di B_:
    - Dato A = F: \text{BR}_B(F) = g (6 > 2).
    - Dato A = G: \text{BR}_B(G) = g (1 > 0). _(La strategia_ _g_ _domina strettamente la strategia_ _f_ _per l'attore B)._
3. _Incrocio delle Risposte Ottime_:
    - L'unico profilo in cui le strategie sono reciprocamente risposte ottime è (G, g).
    - L'esistenza di strategie strettamente dominate esclude la presenza di equilibri misti interni a supporto completo.

- **Insieme degli Equilibri di Nash**: NE = \{(G, g)\} con payoff (1, 1).

### 3. Analisi di Giochi Estesi 3x4 e Concetti Avanzati (Esercizio 2)

Si consideri la matrice dei payoff relativa a un gioco statico 3 \times 4 in cui S_A = \{X, Y, Z\} e S_B = \{J, K, L, M\}:

|   |   |   |   |   |
|---|---|---|---|---|
|A \ B|J|K|L|M|
|**X**|6, 7|5, 5|3, 8|8, 1|
|**Y**|4, 9|9, 2|0, 4|2, 3|
|**Z**|8, 4|2, 8|4, 2|3, 6|

#### 3.1 Dimostrazione dell'Assenza di Equilibri di Nash in Strategie Pure

Si calcolano formalmente gli insiemi di risposta ottima per ogni azione dell'avversario:

- **Risposte Ottime del Giocatore A (****\text{BR}_A****)**:
    - \text{BR}_A(J) = \arg\max \{u_A(X,J)=6, \, u_A(Y,J)=4, \, u_A(Z,J)=8\} = \{Z\}
    - \text{BR}_A(K) = \arg\max \{u_A(X,K)=5, \, u_A(Y,K)=9, \, u_A(Z,K)=2\} = \{Y\}
    - \text{BR}_A(L) = \arg\max \{u_A(X,L)=3, \, u_A(Y,L)=0, \, u_A(Z,L)=4\} = \{Z\}
    - \text{BR}_A(M) = \arg\max \{u_A(X,M)=8, \, u_A(Y,M)=2, \, u_A(Z,M)=3\} = \{X\}
- **Risposte Ottime del Giocatore B (****\text{BR}_B****)**:
    - \text{BR}_B(X) = \arg\max \{u_B(X,J)=7, \, u_B(X,K)=5, \, u_B(X,L)=8, \, u_B(X,M)=1\} = \{L\}
    - \text{BR}_B(Y) = \arg\max \{u_B(Y,J)=9, \, u_B(Y,K)=2, \, u_B(Y,L)=4, \, u_B(Y,M)=3\} = \{J\}
    - \text{BR}_B(Z) = \arg\max \{u_B(Z,J)=4, \, u_B(Z,K)=8, \, u_B(Z,L)=2, \, u_B(Z,M)=6\} = \{K\}

**Verifica della condizione di reciprocità (****s_i \in \text{BR}_i(s_{-i})****)**:

- In (X, L): \text{BR}_B(X) = L, ma \text{BR}_A(L) = Z \neq X.
- In (Y, J): \text{BR}_B(Y) = J, ma \text{BR}_A(J) = Z \neq Y.
- In (Z, K): \text{BR}_B(Z) = K, ma \text{BR}_A(K) = Y \neq Z.
- In (X, M): \text{BR}_A(M) = X, ma \text{BR}_B(X) = L \neq M.

Nessuna delle 12 coppie di strategie pure appartiene al grafico cartesiano delle risposte ottime incrociate. Pertanto, **non esiste alcun Equilibrio di Nash in strategie pure**.

#### 3.2 Verifica degli Equilibri di Nash in Strategie Miste

Un profilo di strategie miste (m_A, m_B) costituisce un Equilibrio di Nash se e solo se:

1. Per ogni s_i, s_i' \in \text{supp}(m_i), U_i(s_i, m_{-i}) = U_i(s_i', m_{-i}) = U_i(m_i, m_{-i}).
2. Per ogni s_i'' \notin \text{supp}(m_i), U_i(s_i'', m_{-i}) \le U_i(m_i, m_{-i}).

##### Dimostrazione per il Profilo 1:

m_A = \left(\frac{2}{3}, 0, \frac{1}{3}\right), \quad m_B = \left(\frac{5}{11}, \frac{4}{11}, \frac{2}{11}, 0\right)

- **Verifica per il Giocatore A** (dato m_B):
    - U_A(X, m_B) = 6\left(\frac{5}{11}\right) + 5\left(\frac{4}{11}\right) + 3\left(\frac{2}{11}\right) + 8(0) = \frac{30 + 20 + 6}{11} = \frac{56}{11}
    - U_A(Y, m_B) = 4\left(\frac{5}{11}\right) + 9\left(\frac{4}{11}\right) + 0\left(\frac{2}{11}\right) + 2(0) = \frac{20 + 36 + 0}{11} = \frac{56}{11}
    - U_A(Z, m_B) = 8\left(\frac{5}{11}\right) + 2\left(\frac{4}{11}\right) + 4\left(\frac{2}{11}\right) + 3(0) = \frac{40 + 8 + 8}{11} = \frac{56}{11}

Tutte le strategie pure di A (sia quelle nel supporto \{X, Z\}, sia la strategia Y fuori dal supporto) erogano la medesima utilità attesa \frac{56}{11}. La condizione di best response per A è pienamente verificata.

- **Verifica per il Giocatore B** (dato m_A):
    - U_B(m_A, J) = 7\left(\frac{2}{3}\right) + 9(0) + 4\left(\frac{1}{3}\right) = \frac{14 + 4}{3} = 6
    - U_B(m_A, K) = 5\left(\frac{2}{3}\right) + 2(0) + 8\left(\frac{1}{3}\right) = \frac{10 + 8}{3} = 6
    - U_B(m_A, L) = 8\left(\frac{2}{3}\right) + 4(0) + 2\left(\frac{1}{3}\right) = \frac{16 + 2}{3} = 6
    - U_B(m_A, M) = 1\left(\frac{2}{3}\right) + 3(0) + 6\left(\frac{1}{3}\right) = \frac{2 + 6}{3} = \frac{8}{3}

Per le strategie nel supporto \text{supp}(m_B) = \{J, K, L\}, l'utilità è uniformemente pari a 6. Per la strategia fuori dal supporto M, l'utilità attesa è 8/3 < 6. Di conseguenza, m_B è la risposta ottima a m_A.

Il profilo (m_A, m_B) è un **Equilibrio di Nash in strategie miste**.

##### Dimostrazione per il Profilo 2:

m_A = \left(0, \frac{4}{11}, \frac{7}{11}\right), \quad m_B = \left(\frac{7}{11}, \frac{4}{11}, 0, 0\right)

- **Verifica per il Giocatore A** (dato m_B):
    - U_A(X, m_B) = 6\left(\frac{7}{11}\right) + 5\left(\frac{4}{11}\right) + 3(0) + 8(0) = \frac{42 + 20}{11} = \frac{62}{11}
    - U_A(Y, m_B) = 4\left(\frac{7}{11}\right) + 9\left(\frac{4}{11}\right) + 0(0) + 2(0) = \frac{28 + 36}{11} = \frac{64}{11}
    - U_A(Z, m_B) = 8\left(\frac{7}{11}\right) + 2\left(\frac{4}{11}\right) + 4(0) + 3(0) = \frac{56 + 8}{11} = \frac{64}{11}

Le strategie nel supporto \{Y, Z\} garantiscono un payoff atteso pari a \frac{64}{11}, mentre la strategia fuori dal supporto X genera un payoff strettamente inferiore (\frac{62}{11} < \frac{64}{11}). Pertanto, m_A è risposta ottima a m_B.

- **Verifica per il Giocatore B** (dato m_A):
    - U_B(m_A, J) = 7(0) + 9\left(\frac{4}{11}\right) + 4\left(\frac{7}{11}\right) = \frac{36 + 28}{11} = \frac{64}{11}
    - U_B(m_A, K) = 5(0) + 2\left(\frac{4}{11}\right) + 8\left(\frac{7}{11}\right) = \frac{8 + 56}{11} = \frac{64}{11}
    - U_B(m_A, L) = 8(0) + 4\left(\frac{4}{11}\right) + 2\left(\frac{7}{11}\right) = \frac{16 + 14}{11} = \frac{30}{11}
    - U_B(m_A, M) = 1(0) + 3\left(\frac{4}{11}\right) + 6\left(\frac{7}{11}\right) = \frac{12 + 42}{11} = \frac{54}{11}

Le strategie nel supporto \{J, K\} garantiscono un payoff atteso di \frac{64}{11}, superiore a quello delle strategie non giocate L (\frac{30}{11}) e M (\frac{54}{11}). Dunque m_B è risposta ottima a m_A.

Il profilo costituisce un **secondo Equilibrio di Nash in strategie miste**.

#### 3.3 Identificazione delle Strategie Congiunte Pure Pareto-Ottimali

Si valuta la dominanza ai sensi di Pareto per ciascuna delle 12 combinazioni di strategie pure:

1. **(X, J)** con payoff **(6, 7)**: Non dominato da alcuna allocazione. \rightarrow **Pareto-Ottimale**
2. **(X, K)** con payoff **(5, 5)**: Strettamente dominato da (X, J) poiché 6 > 5 e 7 > 5.
3. **(X, L)** con payoff **(3, 8)**: Strettamente dominato da (Y, J) poiché 4 > 3 e 9 > 8.
4. **(X, M)** con payoff **(8, 1)**: Strettamente dominato da (Z, J) poiché 8 \ge 8 e 4 > 1.
5. **(Y, J)** con payoff **(4, 9)**: Non dominato (rappresenta il massimo payoff assoluto per B). \rightarrow **Pareto-Ottimale**
6. **(Y, K)** con payoff **(9, 2)**: Non dominato (rappresenta il massimo payoff assoluto per A). \rightarrow **Pareto-Ottimale**
7. **(Y, L)** con payoff **(0, 4)**: Strettamente dominato da (Z, J) poiché 8 > 0 e 4 \ge 4.
8. **(Y, M)** con payoff **(2, 3)**: Strettamente dominato da (X, J) poiché 6 > 2 e 7 > 3.
9. **(Z, J)** con payoff **(8, 4)**: Non dominato da alcuna altra allocazione. \rightarrow **Pareto-Ottimale**
10. **(Z, K)** con payoff **(2, 8)**: Strettamente dominato da (Y, J) poiché 4 > 2 e 9 > 8.
11. **(Z, L)** con payoff **(4, 2)**: Strettamente dominato da (Z, J) poiché 8 > 4 e 4 > 2.
12. **(Z, M)** con payoff **(3, 6)**: Strettamente dominato da (X, J) poiché 6 > 3 e 7 > 6.

**Elenco completo delle strategie congiunte pure Pareto-ottimali:**

- **(X, J)** con payoff **(6, 7)**
- **(Y, J)** con payoff **(4, 9)**
- **(Y, K)** con payoff **(9, 2)**
- **(Z, J)** con payoff **(8, 4)**

### 4. Traduzione e Formalizzazione di Problemi in Prosa

#### 4.1 Modello della Scelta del Supervisore della Tesi (Esercizio 3)

##### Step 1: Estrazione Parametri e Regole

- **Giocatori**: Charlotte (C) e Daniel (D).
- **Insieme delle strategie**: Scelta autonoma del relatore S_C = S_D = \{X, Y, Z\}, dove X = \text{Xavier}, Y = \text{Yuan}, Z = \text{Zingberry}.
- **Utilità nominale della supervisione esclusiva**: v(X) = 40, \quad v(Y) = 60, \quad v(Z) = 50
- **Regola di riduzione dell'utilità in caso di congestione**: In presenza di scelta coincidente dello stesso docente, la disponibilità del supervisore dimezza l'attenzione individuale, garantendo una ritenzione dell'utilità pari al 70\% del valore base (0.70 \cdot v(\cdot)).

##### Step 1b: Calcolo Dettagliato delle Utilità Individuali per le 9 Combinazioni

1. **Profilo (X, X)**: Entrambi scelgono Xavier.
    - u_C(X, X) = 40 \times 0.70 = 28
    - u_D(X, X) = 40 \times 0.70 = 28
    - Payoff: **(28, 28)**
2. **Profilo (X, Y)**: Charlotte sceglie Xavier, Daniel sceglie Yuan.
    - u_C(X, Y) = 40
    - u_D(X, Y) = 60
    - Payoff: **(40, 60)**
3. **Profilo (X, Z)**: Charlotte sceglie Xavier, Daniel sceglie Zingberry.
    - u_C(X, Z) = 40
    - u_D(X, Z) = 50
    - Payoff: **(40, 50)**
4. **Profilo (Y, X)**: Charlotte sceglie Yuan, Daniel sceglie Xavier.
    - u_C(Y, X) = 60
    - u_D(Y, X) = 40
    - Payoff: **(60, 40)**
5. **Profilo (Y, Y)**: Entrambi scelgono Yuan.
    - u_C(Y, Y) = 60 \times 0.70 = 42
    - u_D(Y, Y) = 60 \times 0.70 = 42
    - Payoff: **(42, 42)**
6. **Profilo (Y, Z)**: Charlotte sceglie Yuan, Daniel sceglie Zingberry.
    - u_C(Y, Z) = 60
    - u_D(Y, Z) = 50
    - Payoff: **(60, 50)**
7. **Profilo (Z, X)**: Charlotte sceglie Zingberry, Daniel sceglie Xavier.
    - u_C(Z, X) = 50
    - u_D(Z, X) = 40
    - Payoff: **(50, 40)**
8. **Profilo (Z, Y)**: Charlotte sceglie Zingberry, Daniel sceglie Yuan.
    - u_C(Z, Y) = 50
    - u_D(Z, Y) = 60
    - Payoff: **(50, 60)**
9. **Profilo (Z, Z)**: Entrambi scelgono Zingberry.
    - u_C(Z, Z) = 50 \times 0.70 = 35
    - u_D(Z, Z) = 50 \times 0.70 = 35
    - Payoff: **(35, 35)**

##### Step 2: Costruzione della Forma Normale

La matrice di payoff 3 \times 3 risultante è:

|   |   |   |   |
|---|---|---|---|
|C \ D|Xavier (X)|Yuan (Y)|Zingberry (Z)|
|**Xavier (X)**|28, 28|40, 60|40, 50|
|**Yuan (Y)**|60, 40|42, 42|60, 50|
|**Zingberry (Z)**|50, 40|50, 60|35, 35|

##### Step 3: Ricerca degli Equilibri di Nash Puri

1. **Risposte ottime di Charlotte (****C****)**:
    - Se D = X \implies \text{BR}_C(X) = Y (payoff 60 > 50 > 28).
    - Se D = Y \implies \text{BR}_C(Y) = Z (payoff 50 > 42 > 40).
    - Se D = Z \implies \text{BR}_C(Z) = Y (payoff 60 > 40 > 35).
2. **Risposte ottime di Daniel (****D****)**:
    - Se C = X \implies \text{BR}_D(X) = Y (payoff 60 > 50 > 28).
    - Se C = Y \implies \text{BR}_D(Y) = Z (payoff 50 > 42 > 40).
    - Se C = Z \implies \text{BR}_D(Z) = Y (payoff 60 > 40 > 35).

Intersecando i risultati, gli **Equilibri di Nash in strategie pure** sono:

- **(Y, Z)** con payoff **(60, 50)**
- **(Z, Y)** con payoff **(50, 60)**

##### Step 4: Determinazione dell'Insieme Completo degli Equilibri

Si osserva che per entrambi i giocatori la strategia X (Xavier) è **strettamente dominata** dalla strategia Y (Yuan). Infatti:

- u_C(Y, s_D) > u_C(X, s_D) \quad \forall s_D \in \{X, Y, Z\} \quad (60 > 28, \, 42 > 40, \, 60 > 40).

In base al principio dell'Eliminazione Iterata di Strategie Strettamente Dominate (IESTDS), la strategia X non può essere giocata con probabilità positiva in alcun equilibrio di Nash. Eliminando la riga e la colonna corrispondenti a X, il gioco si riduce a una matrice 2 \times 2 sulle strategie \{Y, Z\}:

|   |   |   |
|---|---|---|
|C \ D|Yuan (Y)|Zingberry (Z)|
|**Yuan (Y)**|42, 42|60, 50|
|**Zingberry (Z)**|50, 60|35, 35|

Applicando il Principio di Indifferenza per il calcolo dell'equilibrio misto: Sia m_C = (0, q, 1-q) e m_D = (0, p, 1-p), dove q e p indicano la probabilità di scegliere Y:

- _Indifferenza di_ _C_: U_C(Y, m_D) = U_C(Z, m_D) 42p + 60(1-p) = 50p + 35(1-p) \implies 60 - 18p = 35 + 15p \implies 33p = 25 \implies p^* = \frac{25}{33}
- _Indifferenza di_ _D_: Per simmetria strutturale della matrice, q^* = \frac{25}{33}.

**Insieme completo degli Equilibri di Nash del gioco**:

1. Equilibrio puro 1: **(Y, Z)** con payoff (60, 50)
2. Equilibrio puro 2: **(Z, Y)** con payoff (50, 60)
3. Equilibrio misto: \left( m_C = \left(0, \frac{25}{33}, \frac{8}{33}\right), \, m_D = \left(0, \frac{25}{33}, \frac{8}{33}\right) \right).

#### 4.2 Modello di Evasione Fiscale e Ispezione (Esercizio 4)

##### Step 1: Formalizzazione Simbolica

- **Giocatori**: Contribuente T (Taxpayer) e Ispettore Fiscale I (Inspector).
- **Insieme delle Strategie Pure**:
    - S_T = \{H, P\} dove H = Nascondere/Evadere quota di tasse (Hide), P = Pagare integralmente le imposte (Pay full).
    - S_I = \{C, D\} dove C = Svolgere un controllo/ispezione (Check), D = Non ispezionare (Do not check).
- **Parametri del Modello**:
    - R: Reddito netto al netto delle imposte regolarmente corrisposte (R > 0).
    - S: Ammontare totale delle imposte dovute (S > 0).
    - L: Incremento patrimoniale illecito ottenuto evadendo le imposte (0 < L \le S).
    - E: Costo operativo sostenuto dall'Ispettore per condurre un'ispezione (E > 0).
    - F: Sanzione pecuniaria inflitta al contribuente e incassata dall'Ispettore in caso di accertamento dell'evasione (F > 0).
    - p: Probabilità dell'Ispettore di individuare e dimostrare l'evasione a seguito di un controllo (0 < p \le 1).

##### Step 2: Derivazione Assiomatica delle Funzioni di Payoff

- **Profilo (P, D)** [Adempimento, Nessun Controllo]:
    - T paga le imposte S ottenendo reddito netto R. Payoff u_T(P, D) = R.
    - I incassa S senza sostenere costi. Payoff u_I(P, D) = S.
- **Profilo (P, C)** [Adempimento, Controllo]:
    - T è in regola e conserva il reddito netto R. Payoff u_T(P, C) = R.
    - I incassa S ma sostiene il costo d'ispezione E. Payoff u_I(P, C) = S - E.
- **Profilo (H, D)** [Evasione, Nessun Controllo]:
    - T trattiene L, ottenendo un reddito incrementato R + L. Payoff u_T(H, D) = R + L.
    - I incassa solo il gettito non evaso S - L. Payoff u_I(H, D) = S - L.
- **Profilo (H, C)** [Evasione, Controllo]:
    - Per il Contribuente T: con probabilità p l'evasione viene accertata (restituzione imposta, sanzione F, reddito residuo R - F); con probabilità (1-p) l'evasione non viene rilevata (reddito R + L). u_T(H, C) = p(R - F) + (1-p)(R + L) = R + (1-p)L - pF
    - Per l'Ispettore I: con probabilità p recupera le imposte S e incassa la multa F; con probabilità (1-p) incassa solo S - L. In ogni caso sostiene il costo d'ispezione E. u_I(H, C) = p(S + F) + (1-p)(S - L) - E = S - (1-p)L + pF - E

##### Step 3: Costruzione della Matrice 2x2 Parametrica

|   |   |   |
|---|---|---|
|T \ I|Check (C)|Do not check (D)|
|**Hide (H)**|R + (1-p)L - pF, \quad S - (1-p)L + pF - E|R + L, \quad S - L|
|**Pay full (P)**|R, \quad S - E|R, \quad S|

##### Step 4: Analisi degli Equilibri di Nash e Comparazione Statica

1. **Analisi delle Risposte Ottime dell'Ispettore (****I****)**:
    - Se T = P: S > S - E (poiché E > 0), quindi la risposta ottima è strettamente **D**.
    - Se T = H: I sceglie C rispetto a D se e solo se: S - (1-p)L + pF - E > S - L \iff -E + p(L + F) > 0 \iff p(L + F) > E
2. **Analisi delle Risposte Ottime del Contribuente (****T****)**:
    - Se I = D: R + L > R (poiché L > 0), quindi la risposta ottima è strettamente **H**.
    - Se I = C: T sceglie P rispetto a H se e solo se: R > R + (1-p)L - pF \iff pF > (1-p)L
3. **Determinazione dei Regimi di Equilibrio**:
    - **Regime A: Costo di Ispezione Inefficiente (****p(L + F) \le E****)**: Se il costo dell'ispezione supera il beneficio atteso dall'accertamento, la strategia D è strettamente dominante per l'Ispettore. Data la scelta D, la risposta ottima del Contribuente è H. \rightarrow **Equilibrio di Nash puro unico**: **(H, D)** con payoff (R + L, \, S - L).
    - **Regime B: Deterrenza Strategica e Audit Attivo (****p(L + F) > E** **e** **pF > (1-p)L****)**: Non esistono equilibri in strategie pure a causa della presenza di cicli di risposte ottime ((H, C) \to (P, C) \to (P, D) \to (H, D) \to (H, C)). Si determina l'**Equilibrio di Nash in strategie miste**:
        - Sia q = P(T = H) la probabilità di evasione del Contribuente.
        - Sia r = P(I = C) la probabilità di ispezione dell'Ispettore.

_Derivazione di_ _q^*_ _(Indifferenza per l'Ispettore)_: U_I(C, q) = U_I(D, q) q \left[ S - (1-p)L + pF - E \right] + (1-q)[S - E] = q(S - L) + (1-q)S Sviluppando e semplificando i termini identici S - qL presente ad entrambi i membri: S - E - qL + q p(L + F) = S - qL \implies -E + q p(L + F) = 0 q^* = \frac{E}{p(L + F)} _(Nota di ammissibilità: la condizione del Regime B_ _p(L + F) > E_ _garantisce_ _q^* \in (0, 1)__)._

_Derivazione di_ _r^*_ _(Indifferenza per il Contribuente)_: U_T(H, r) = U_T(P, r) r \left[ R + (1-p)L - pF \right] + (1-r)[R + L] = R R + L - r \left[ L - (1-p)L + pF \right] = R \implies L - r p(L + F) = 0 r^* = \frac{L}{p(L + F)}

##### Interpretazione Economica e Statica Comparata dell'Equilibrio Misto

L'equilibrio misto derivato:

\left( m_T^* = \left(\frac{E}{p(L + F)}, \, 1 - \frac{E}{p(L + F)}\right), \quad m_I^* = \left(\frac{L}{p(L + F)}, \, 1 - \frac{L}{p(L + F)}\right) \right)

evidenzia la classica proprietà contraintuitiva dei giochi di controllo in strategie miste:

- **Tasso di Evasione Fiscale in Equilibrio (****q^*****)**: La probabilità di evasione del contribuente dipende esclusivamente dai parametri organizzativi ed economici dell'Ispettore (E, p, F) e non dall'onere fiscale S o dal beneficio immediato dell'evasione L. Un aumento della sanzione F o un incremento dell'efficacia dell'ispezione p **non** riduce direttamente la propensione all'evasione del contribuente a parità di altre condizioni, ma riduce la frequenza di controllo richiesta dall'ispettore.
- **Frequenza di Audit dell'Ispettore (****r^*****)**: La probabilità di controllo r^* è direttamente proporzionale al guadagno illecito L ed inversamente proporzionale alla severità della multa F e all'efficienza p. Pertanto, la sanzione F e la probabilità di scoperta p agiscono come strumenti di deterrenza indiretta: incrementare F consente allo Stato di mantenere lo stesso livello di adempimento fiscale riducendo la frequenza dei controlli costosi r^*, minimizzando così la spesa pubblica d'ispezione E.