# Flipper

Gioco arcade di flipper realizzato in **Unreal Engine 5.7**, con una direzione visiva synthwave e un focus sulla risposta fisica della pallina. Il progetto combina palette controllate da tastiera, un lanciatore caricabile, respingenti, elementi di rallentamento, muri distruttibili e un'interfaccia per il punteggio e la fine della partita.

La relazione *Flipper Physics* indica come obiettivo una fisica credibile e reattiva, vicina alla sensazione dei cabinati arcade. Questa intenzione si traduce nella combinazione di materiali fisici, impulsi e interazioni dedicate ai diversi oggetti del tavolo.

## Esperienza di gioco

Il giocatore carica il lanciatore e controlla due palette per mantenere la pallina in gioco. Gli incontri con gli elementi del tavolo modificano il suo movimento e alimentano il sistema di punteggio. L'HUD mostra il totale e i comandi; è inoltre presente una schermata di Game Over con le indicazioni per ricominciare o uscire.

Il ciclo di gioco descritto dal materiale disponibile è:

1. Caricare il lanciatore per mettere in movimento la pallina.
2. Azionare le palette per rilanciarla nell'area di gioco.
3. Interagire con respingenti, rallentatori e muri, seguendo il punteggio nell'HUD.
4. Alla fine della partita, ricominciare oppure chiudere il gioco.

Non sono documentati un numero di vite, una condizione di vittoria, moltiplicatori, una classifica o una progressione fra livelli. Questi elementi non vengono quindi attribuiti al progetto.

### Controlli

| Tasto | Azione |
| --- | --- |
| **O** | Aziona la paletta sinistra. |
| **P** | Aziona la paletta destra. |
| **Spazio - Hold** | Carica il lanciatore. |
| **Q** | Ricarica la partita. |
| **Esc** | Chiude il gioco. |

Le indicazioni dell'HUD trovano riscontro negli eventi di input serializzati negli asset. Il lanciatore contiene eventi di pressione e rilascio di Spazio; la relazione esatta fra durata della pressione, potenza e momento del lancio va confermata in Unreal. Non è stato verificato un sistema di rimappatura né un supporto giocabile al controller.

## Elementi e meccaniche

### Pallina

La pallina è il principale oggetto fisico con cui il giocatore interagisce indirettamente. La relazione attribuisce particolare importanza al rapporto fra massa e forza dei respingenti: una risposta troppo debole può far perdere rapidamente inerzia, mentre una troppo intensa può portare la pallina oltre i limiti del tavolo.

Nell'asset sono presenti proprietà relative a simulazione fisica, massa, collisioni e CCD. I valori effettivi delle istanze in scena, incluse eventuali modifiche rispetto al Blueprint di base, richiedono un controllo nell'editor.

### Palette

Le due palette sono controllabili separatamente. Nei rispettivi Blueprint sono presenti una Timeline e operazioni di rotazione relativa: questo riscontro indica una soluzione basata sul controllo della rotazione, da verificare nei collegamenti del grafo e in gioco.

Non è stata confermata una soluzione basata su motori di giunti fisici. La descrizione della relazione secondo cui le palette producono impulsi intensi va quindi intesa come descrizione dell'effetto sulla pallina, senza attribuire alle palette una specifica chiamata di applicazione dell'impulso.

### Lanciatore

Il lanciatore introduce una fase di carica prima dell'ingresso della pallina nel gioco. L'asset contiene una zona di rilevamento, un riferimento alla pallina, una Timeline di potenza, il movimento di un pistone e un'operazione di impulso.

Questi elementi supportano la presenza di un lanciatore caricabile; non sono qui fissati valori di forza, tempi massimi o formule di carica, perché non è stata ricostruita integralmente la sequenza di esecuzione.

### Respingenti

I bumper aggiungono una risposta attiva agli incontri con la pallina. La relazione descrive una repulsione fisica personalizzata. Nell'asset sono presenti un evento di sovrapposizione, operazioni su posizione e vettori, una normalizzazione, un impulso e un riferimento al sistema di assegnazione punti.

La scelta distingue la repulsione aggiuntiva dal solo rimbalzo passivo delle superfici. Intensità e punti assegnati devono essere verificati anche sulle singole istanze del tavolo.

### Rallentatori

Gli Slower modificano il movimento della pallina. Il PDF parla di variazione del momento lineare; l'asset contiene invece, più precisamente, operazioni di lettura e impostazione della velocità lineare.

Dal punto di vista del giocatore, il comportamento descritto è un rallentamento. Il fattore applicato e le condizioni di attivazione rimangono da verificare nel grafo: un valore isolato nell'asset non basta a stabilire l'effetto finale.

### Muri distruttibili

La relazione presenta muri di vetro fucsia che reagiscono al contatto con la pallina. Il Blueprint esaminato contiene riferimenti a un evento di sovrapposizione, all'assegnazione di punti e alla distruzione di un attore.

**La simulazione di frammenti descritta nel PDF non è confermata.** Non sono stati individuati asset Geometry Collection nel contenuto inventariato. Il riscontro disponibile supporta una logica di rimozione dell'oggetto, ma non dimostra una frantumazione fisica del vetro.

### Punteggio e fine partita

È presente una gestione centrale del punteggio e un widget HUD che ne legge il valore e lo converte in testo. Bumper e muri contengono riferimenti alla funzione di aggiunta dei punti.

Un attore dedicato al Game Over contiene riferimenti al riconoscimento della pallina, alla distruzione di un attore e alla creazione della schermata finale. È inoltre presente una variabile per l'ultimo punteggio nella Game Instance. Questo non dimostra l'esistenza di un record salvato su disco o di una classifica persistente.

Non vengono indicati valori dei premi, regole di combo o un aggiornamento dell'HUD tramite dispatcher: quest'ultimo mostra evidenze di un binding del testo.

## Scelte progettuali e motivazioni

### Risposta fisica e comportamento arcade

La relazione motiva l'uso dei materiali fisici con la possibilità di ottenere rimbalzi elastici attraverso il motore, riducendo il bisogno di calcolare manualmente i vettori di riflessione. Nel progetto sono presenti materiali fisici distinti e interazioni che applicano impulsi o modificano la velocità.

Il risultato ricercato è quindi una risposta fisica regolabile per tipo di elemento. Le valutazioni positive sulla qualità del rimbalzo appartengono alla relazione e non costituiscono risultati di un playtest svolto per questo README.

### Stabilità alle alte velocità

Il PDF identifica il *tunneling*, cioè l'attraversamento delle superfici fra due aggiornamenti della simulazione, come una delle principali difficoltà tecniche. Riporta l'uso del **Continuous Collision Detection** per contrastarlo.

La configurazione conferma inoltre il **substepping fisico**, con un intervallo massimo per sottopasso di **0,01 secondi** e un massimo di **16 sottopassi**. Questa scelta aumenta la granularità temporale disponibile per la simulazione. Non equivale a garantire l'assenza di attraversamenti o una frequenza di rendering di 100 FPS.

La presenza delle proprietà CCD negli asset è coerente con la relazione; la loro attivazione effettiva su ciascun componente va verificata nell'editor.

### Separazione degli oggetti

La relazione indica la separazione di pallina, palette e ostacoli in Blueprint distinti come una scelta che ha reso lo sviluppo più ordinato e scalabile. La struttura degli asset conferma questa suddivisione.

Le interazioni sono distribuite fra oggetti dedicati, mentre punteggio e interfaccia hanno componenti separati. Questo è il riscontro organizzativo disponibile; non vengono attribuiti contributi specifici a membri del gruppo.

### Identità visiva

Il PDF descrive un'estetica synthwave, con vetro fucsia e un'impostazione arcade. L'icona del progetto utilizza la scritta “PINBALL”, colori luminosi e richiami al neon. Il contenuto include materiali distinti per superfici e oggetti del tavolo.

Le impostazioni grafiche privilegiano un profilo desktop di qualità massima e includono illuminazione e riflessi dinamici. Non sono disponibili misurazioni che permettano di indicare requisiti minimi, FPS attesi o il costo effettivo delle singole opzioni.

## Organizzazione tecnica

La copia esaminata è un **progetto Blueprint**: non contiene una cartella di sorgenti C++ e il descrittore non dichiara moduli applicativi. Il PDF lo definisce invece ibrido C++/Blueprint; questa affermazione non è supportata dalla cartella disponibile.

| Area | Contenuto e responsabilità riscontrati |
| --- | --- |
| Livello principale | Tavolo, riferimenti agli oggetti di gioco, camera e comandi di ricarica/uscita. |
| Blueprint degli oggetti | Pallina, palette, lanciatore, bumper, rallentatore e muro. |
| Gestione della partita | Game Mode con punteggio e creazione dell'HUD; attore dedicato al Game Over. |
| Game Instance | Variabile per l'ultimo punteggio; persistenza su disco non confermata. |
| Interfaccia | Widget HUD e Game Over. |
| Materiali | Materiali visivi e materiali fisici. |
| Geometrie | Mesh del tavolo e degli elementi, con un file FBX presente nel contenuto. |
| Configurazione | Avvio, input, grafica, fisica e packaging. |

## Requisiti e avvio

1. Installare **Unreal Engine 5.7**. La versione **5.7.4** è indicata dal PDF e confermata nei log storici della copia esaminata.
2. Se il progetto viene recuperato da Git, installare Git LFS e scaricare i contenuti LFS prima di aprirlo: il repository configura questo sistema per asset Unreal e altri file binari.
3. Aprire il file `.uproject` dalla cartella del progetto.
4. Attendere il caricamento e l'eventuale compilazione degli shader.
5. Avviare il livello predefinito tramite **Play**, quindi dare il focus alla finestra di gioco.
6. Utilizzare i comandi indicati nella tabella.

Il livello iniziale è già configurato sia per l'editor sia per il gioco. Per la copia analizzata non è documentato un passaggio di compilazione di codice C++ applicativo.

Il target Windows usa DirectX 12 come impostazione predefinita. Le impostazioni presenti per Linux e macOS non costituiscono una verifica di compatibilità su quelle piattaforme.

### Build

La configurazione di packaging seleziona **Shipping**. Un log del 19 settembre 2026 registra il completamento di un packaging Windows con codice di uscita 0.

Questo conferma un esito storico, non una nuova build verificata durante la stesura. Per riprodurre la distribuzione occorre eseguire il packaging Windows dall'editor con i componenti di piattaforma richiesti installati e provare l'eseguibile risultante.

### Gestione del repository

Sono configurati Git LFS per asset e media e l'esclusione delle principali cartelle generate, delle cache e dei file degli IDE. Il PDF dichiara inoltre l'uso di Git Flow e branch separati per oggetto; la cronologia e i branch non sono stati verificati.

Non è stato individuato un file di licenza nel materiale di progetto inventariato: non si attribuisce quindi una licenza di riuso al progetto.

## Differenze rispetto alla relazione e verifiche aperte

| Tema | Esito dell'analisi |
| --- | --- |
| C++ e Blueprint | Il PDF dichiara un'architettura ibrida; la copia disponibile non contiene sorgenti o moduli C++ applicativi. |
| Materiali fisici | I nomi esemplificati nel PDF differiscono dai nomi effettivi. Associazioni e override vanno controllati per componente. |
| Distruzione del vetro | La rimozione dell'attore ha riscontri negli asset; la simulazione dei frammenti non è confermata. |
| CCD | Dichiarato nella relazione; presenti proprietà corrispondenti, ma attivazione e copertura non verificate in editor. |
| Input | I comandi sono presenti nei Blueprint e nei testi UI; le impostazioni generiche di Enhanced Input non dimostrano un sistema completo di Input Action. |
| Git Flow | Dichiarato dal PDF; non verificato nella cronologia. |

Prima di presentare il progetto come interamente validato, restano da controllare in Unreal la compilazione dei Blueprint, i collegamenti dei grafi, i valori delle istanze, la carica del lanciatore, il conteggio dei punti, il Game Over e il riavvio. Sono inoltre necessarie prove ad alta velocità per valutare il contenimento della pallina e una prova della build distribuita.

Alcuni asset conservano stringhe diagnostiche relative a pin o collegamenti: possono essere residui di precedenti modifiche e non dimostrano un errore attuale. Vanno risolte tramite una nuova compilazione nell'editor, senza confonderle con l'esito positivo del packaging storico.
