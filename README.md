# Bowflex ↔ Ferro equivalente

Mini web-app mobile-first per stimare il **ferro equivalente** di un setup su **Bowflex Xtreme SE 2**.

L’app non tratta le libbre nominali stampate sulle Power Rods come peso direttamente confrontabile con bilancieri, manubri o cavi tradizionali. Parte invece dalla composizione delle aste, dal routing dei cavi, dalla maniglia, dall’esercizio, dall’eventuale accessorio **BC52** e dalla temperatura delle Power Rods per ottenere una stima operativa in **lb/kg di ferro equivalente**.

> **Stato attuale:** v0.10 — modello inferito e prudenziale, pensato per rendere coerente il diario allenamenti. Non è una misura certificata della resistenza effettiva all’impugnatura.

## App online

Apri l’app da iPhone, Android o desktop:

**https://cirocorvino.github.io/bowflex-lbkg-converter/**

Non richiede installazione, account o backend.

---

## Obiettivo del progetto

Le Power Rods Bowflex hanno una curva di resistenza diversa dal ferro tradizionale. A parità di libbre nominali, la difficoltà può cambiare in funzione di:

- tipo e quantità di Power Rods collegate;
- utilizzo su una linea (`x1`) oppure due cavi/lati (`x2`);
- esercizio e linea di tiro;
- posizione dei pulley;
- ramo medio attivo: `LMI` oppure `LMS`;
- maniglia o accessorio;
- collegamento standard, BC52 centralizzatrice o BC52 impugnata direttamente;
- temperatura effettiva delle Power Rods.

L’app traduce questi elementi in un valore unico di **ferro equivalente**, utile per confrontare sedute diverse e registrare il carico in un diario di allenamento.

---

## Come funziona il modello

Il modello usa questa catena:

```text
Power Rods nominali
→ ferro base Power Rods
× coefficiente esercizio
× coefficiente routing/precarico
× coefficiente maniglia
× coefficiente accessorio/BC52
× coefficiente temperatura
= ferro equivalente
```

### 1. Power Rods nominali

Il valore nominale è la somma delle aste selezionate. È informativo, ma non è il carico finale equivalente.

Esempio:

```text
L 5 + L 30 + M 50 + H 50 = 135 lb nominali Bowflex
```

### 2. Ferro base Power Rods

Ogni tipo di asta usa un fattore prudenziale interno al modello:

| Power Rod | Max per linea/lato | Nominale | Fattore | Ferro base per asta, x1 |
|---|---:|---:|---:|---:|
| L 5 lb | 1 | 5 lb | 0,36 | 1,8 lb |
| L 10 lb | 2 | 10 lb | 0,36 | 3,6 lb |
| L 30 lb | 1 | 30 lb | 0,36 | 10,8 lb |
| M 50 lb | 1 | 50 lb | 0,48 | 24,0 lb |
| H 50 lb | 2 | 50 lb | 0,65 | 32,5 lb |

Le quantità delle aste sono sempre riferite a **una sola linea/lato**. Se selezioni `x2`, il modello moltiplica le aste attive per due cavi/lati.

### 3. Coefficiente esercizio

Il coefficiente esercizio rappresenta la differenza tra il ferro base delle aste e il carico equivalente medio percepito nel gesto.

| Esercizio | Coefficiente |
|---|---:|
| Rematore / Cable Row | 0,72 |
| Lat machine / Pulldown | 0,78 |
| Pushdown tricipiti | 0,68 |
| Face pull | 0,58 |
| Croci ai cavi | 0,62 |
| High row / Rear-delt row | 0,65 |
| Custom | 0,70 |

Il coefficiente `0,65` dell’high row è una stima inferenziale intermedia tra rematore e face pull. Il relativo preset è destinato alla BC52 impugnata direttamente e non va confrontato come se fosse uno dei due esercizi originari.

### 4. Routing, pulley e precarico

Il routing della Bowflex è composto da tre livelli:

```text
LS + un solo ramo LM + LI
```

Il livello medio è mutuamente esclusivo:

```text
LMI oppure LMS
```

Non vengono mai sommati LMI e LMS, perché operano sullo stesso gruppo di pulley.

Il riferimento geometrico è:

```text
LS1 + LMI2 + LI2 = coefficiente routing 1,00
```

Le posizioni più esterne e soprattutto `LMS↑` sono trattate come configurazioni con maggiore precarico/linea di tiro più impegnativa. Il modello usa coefficienti inferiti dalle caratteristiche geometriche e dalle sensazioni di carico rilevate sulla macchina.

| Livello | Posizione | Coefficiente relativo |
|---|---|---:|
| LS | LS1 | 1,00 |
| LS | LS2 | 1,03 |
| LMI | LMI1 | 0,93 |
| LMI | LMI2 | 1,00 |
| LMI | LMI3 | 1,07 |
| LMI | LMI4 | 1,14 |
| LMS | LMS↓ | 1,20 |
| LMS | LMS↑ | 1,32 |
| LI | LI1 | 0,95 |
| LI | LI2 | 1,00 |
| LI | LI3 | 1,05 |
| LI | LI4 | 1,10 |

Il routing totale si ottiene sommando gli scostamenti rispetto alla configurazione di riferimento. L’app conserva inoltre un intervallo di sicurezza interno per evitare risultati estremi.

Esempi indicativi:

```text
LS1 + LMI2 + LI2 = 1,00
LS1 + LMS↑ + LI2 = 1,32
LS2 + LMS↑ + LI4 = 1,45
```

### 5. Accessorio BC52

La **BC52** è una barra centralizzatrice con anelli laterali distanti 52 cm e gancio centrale. Può essere usata in due modi meccanicamente differenti.

#### A. BC52 centralizzatrice

I due cavi vengono collegati agli anelli laterali; la maniglia viene sospesa al centro mediante due moschettoni.

```text
pulley → BC52 → due moschettoni centrali → maniglia → mani
```

La maggiore distanza fra la BC52 e il punto impugnato riduce parte della corsa utile dei cavi e quindi della flessione delle Power Rods. La centralizzazione recupera una quota diversa di resistenza secondo la direzione del gesto.

| Esercizio | Maniglia | Routing/preset | Coefficiente BC52 |
|---|---|---|---:|
| Pushdown | Corda | LS1 + LMS↓ + LI2 | 0,92 |
| Pushdown | Neutra stretta | LS1 + LMS↓ + LI2 | 0,95 |
| Rematore | Neutra stretta | LS1 + LMI2 + LI2 | 0,97 |
| Lat machine stretta | Neutra stretta | LS1 + LMS↓ + LI2 | 0,95 |
| Face pull centralizzato | Corda | LS1 + LMI2 + LI2 | 0,96 |
| Custom | Maniglia scelta | modificabile | 1,00 |

Il face pull con BC52 è una variante condizionata: `LMI2` è circa 30 cm più basso del normale `LMS↓`. Occorre arretrare il corpo finché la linea di tiro risulta soltanto lievemente ascendente, indicativamente non oltre circa 15°. Non è direttamente confrontabile con il face pull standard.

La lat machine con BC52 centralizzatrice è ammessa soltanto con **presa neutra stretta**. La barra lat originale larga/media possiede due agganci laterali allineati a `LS2`, non ha un gancio centrale ed è già un setup dedicato: non può essere combinata con BC52.

#### B. BC52 impugnata direttamente

I cavi vengono collegati agli anelli laterali e le mani afferrano direttamente la barra.

```text
pulley → BC52 → mani
```

Non esistono la maniglia sospesa né i due moschettoni centrali nella catena utile. Il coefficiente accessorio è quindi neutro:

```text
BC52 come barra dritta = 1,00
```

Sono disponibili i seguenti preset:

| Esercizio | Variante |
|---|---|
| Pushdown | Barra dritta BC52 |
| Rematore | Mini-vogatore con barra dritta BC52 |
| Lat machine | Barra corta/media BC52 su LS1 |
| High row / Rear-delt row | Barra dritta BC52 su LMI2 |
| Custom | BC52 come barra dritta |

#### Incompatibilità

L’app impedisce o disabilita le combinazioni seguenti:

- croci ai cavi con qualsiasi modalità BC52;
- face pull con BC52 impugnata direttamente;
- face pull BC52 collegato a `LMS↓`, perché i pulley distano circa 25 cm e la barra li allargherebbe invece di allinearli;
- BC52 combinata con la barra lat originale larga/media;
- high row nel collegamento standard o con maniglia sospesa: il preset è definito per BC52 diretta.

Quando si seleziona un preset BC52 definito:

- `x2` è obbligatorio;
- la composizione delle Power Rods è simmetrica sui due lati;
- routing e maniglia vengono impostati e bloccati sulla combinazione compatibile;
- in `Custom`, routing e maniglia restano modificabili e il coefficiente BC52 è neutro.

### 6. Temperatura delle Power Rods

La temperatura influisce sulla risposta delle aste: in ambiente freddo possono risultare più rigide, mentre con caldo elevato possono apparire meno resistenti.

L’app richiede la **temperatura della stanza in cui le Power Rods sono rimaste per almeno 30–60 minuti**. Non usare la temperatura esterna salvo che sia un buon proxy della temperatura reale della macchina, per esempio in garage o cantina non climatizzati.

Riferimento del modello:

```text
22 °C = coefficiente 1,000
```

Variazione usata:

```text
circa 0,4% per ogni °C rispetto a 22 °C
```

| Temperatura | Coefficiente termico | Effetto stimato |
|---:|---:|---:|
| 15 °C | 1,028 | +2,8% più duro |
| 18 °C | 1,016 | +1,6% |
| 22 °C | 1,000 | riferimento |
| 25 °C | 0,988 | −1,2% |
| 30 °C | 0,968 | −3,2% |
| 35 °C | 0,948 | −5,2% più morbido |

Il coefficiente termico è una stima pratica: serve a mantenere confrontabili le sedute tra stagione fredda e calda, non a descrivere con precisione di laboratorio il comportamento del materiale.

---

## Modalità di utilizzo

### A. Power Rods → ferro equivalente

Usa questa modalità quando conosci il setup Bowflex.

1. Seleziona esercizio e collegamento/accessorio.
2. Scegli la maniglia fra quelle compatibili.
3. Imposta routing LS, LM e LI; nei preset BC52 definiti viene caricato automaticamente.
4. Seleziona `x1`/`x2`; con BC52 l’app impone `x2`.
5. Inserisci la temperatura delle Power Rods.
6. Componi le aste con i pulsanti `+` e `−`.
7. Leggi il valore di **ferro equivalente** in lb e kg.
8. Copia il riepilogo o la riga diario.

### B. Ferro equivalente → Power Rods

Usa questa modalità quando vuoi raggiungere un target di ferro equivalente.

1. Imposta esercizio, accessorio, routing, maniglia, temperatura e `x1`/`x2`.
2. Inserisci il target in lb.
3. L’app cerca la combinazione Power Rod più alta che non superi il target.
4. Verifica il riepilogo proposto e collega le aste indicate.

---

## Riepilogo e riga diario

L’app produce due testi copiabili.

### Riepilogo composizione

Contiene il dettaglio tecnico completo:

- input iniziale;
- esercizio e variante;
- Power Rods attive;
- `x1` o `x2`;
- Bowflex nominale;
- routing completo;
- ramo LM attivo;
- collegamento/accessorio e coefficiente BC52;
- maniglia e relativo coefficiente;
- temperatura delle Power Rods;
- ferro base;
- ferro equivalente finale.

### Riga diario

È più pulita e pensata per il log dell’allenamento. Non contiene ferro base o il dettaglio completo dei coefficienti.

Esempio:

```text
Pushdown tricipiti — BC52 centralizzatrice + corda — routing LS1 + LMS↓ + LI2 — Corda — BC52 centralizzatrice — maniglia al centro con doppio moschettone — rods L 5 + M 50 per linea x2 — 30°C — Bowflex nominale 110 lb — ferro equivalente 38,6 lb / 17,5 kg
```

---

## Struttura del repository

```text
.
├── index.html   # App completa: HTML, CSS e JavaScript senza dipendenze
└── README.md    # Documentazione del progetto
```

L’app è volutamente contenuta in un singolo file per:

- semplicità di manutenzione;
- compatibilità immediata con GitHub Pages;
- ottimo utilizzo su iPhone/Safari;
- assenza di build step, pacchetti o server.

---

## Esecuzione locale

È sufficiente aprire `index.html` nel browser.

Per testarla da un server locale, ad esempio con Python:

```bash
python3 -m http.server 8000
```

Poi apri:

```text
http://localhost:8000
```

---

## Limiti importanti

Questo progetto fornisce una **stima di ferro equivalente**, non una misura diretta della resistenza effettiva all’impugnatura.

Il valore reale può cambiare in funzione di:

- punto del ROM;
- stato e invecchiamento delle Power Rods;
- temperatura effettiva del materiale;
- elasticità del cavo;
- attrito e manutenzione dei pulley;
- posizione del corpo;
- angolo di trazione;
- impugnatura e accessori;
- distanza aggiunta da moschettoni e maniglie;
- velocità di esecuzione.

I coefficienti BC52 sono inferenze operative basate su geometria, corsa utile, tipo di esercizio e prime sensazioni sul campo. Non derivano da una misurazione dinamometrica.

La conversione è quindi utile soprattutto per registrare sedute in modo coerente nel tempo, non per sostenere che una configurazione Bowflex equivalga in modo assoluto a un preciso carico di bilanciere.

---

## Roadmap

- [x] Input Power Rods → ferro equivalente
- [x] Input ferro equivalente → composizione Power Rods
- [x] Routing LS + LMI/LMS + LI
- [x] Esclusione reciproca LMI/LMS
- [x] Coefficienti esercizio-specifici
- [x] Coefficiente maniglia
- [x] Coefficiente temperatura 15–35 °C
- [x] Output leggibile e copiabile su mobile
- [x] Preset specifici per esercizio
- [x] BC52 centralizzatrice con coefficienti per esercizio/maniglia
- [x] BC52 impugnata direttamente come barra dritta
- [x] Regole di compatibilità per lat machine, face pull e croci
- [ ] Salvataggio automatico dell’ultimo setup nel browser
- [ ] Storico/calibrazione da diario allenamenti
- [ ] Modalità calibrazione con dinamometro o bilancia a gancio
- [ ] Profilo di resistenza per inizio / metà / fine ROM
- [ ] v1.0 con coefficienti misurati sulla singola macchina

---

## Calibrazione futura consigliata

La versione più accurata richiederà prove dirette sulla macchina.

Protocollo minimo suggerito:

1. Usa una sola configurazione Power Rod alla volta.
2. Mantieni uguali esercizio, maniglia, distanza del corpo e temperatura.
3. Distingui collegamento standard, BC52 centralizzatrice e BC52 impugnata direttamente.
4. Collega una bilancia a gancio o un dinamometro al punto di trazione.
5. Misura almeno in tre punti del ROM: inizio, metà e fine.
6. Ripeti per routing interno, riferimento e routing esterno.
7. Ripeti per `LMS↓`, `LMS↑` e `LMI2` quando applicabili.
8. Aggiorna i coefficienti con i rapporti misurati rispetto al rispettivo setup di riferimento a 22 °C.

Con alcune misure ben controllate sarà possibile trasformare l’attuale modello inferito in una mappa calibrata sulla specifica Bowflex.

---

## Disclaimer

Questa app è un progetto personale di stima e tracciamento dell’allenamento. Non sostituisce indicazioni mediche, fisioterapiche o professionali. Usa carichi, tecnica e progressioni adatti alla tua esperienza e alle condizioni della tua attrezzatura.
