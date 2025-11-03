# Reazione a Catena — Progetto di Sistemi Operativi 2023/24


Questo repository contiene l'implementazione della simulazione di **reazione a catena** richiesta per il progetto di _Sistemi Operativi_ (a.a. 2023/24). 
L'applicazione modella i processi:
- **master**: coordina la simulazione, raccoglie statistiche e preleva energia;
- **atomo**: esegue la scissione (fork) generando nuovi atomi ed energia;
- **attivatore**: ordina agli atomi quando scindersi;
- **alimentazione**: immette periodicamente nuovi atomi.

(Opzionale): **inibitore**, che assorbe energia e limita le scissioni in modo probabilistico.

> Nota: tutti i parametri sono caricati **a runtime** (da file o variabili d'ambiente), senza necessità di ricompilare.


## Struttura del progetto

```
project/
└── Sistemi-Operativi-main
    ├── alimentazione.c
    ├── atomo.c
    ├── attivatore.c
    ├── inibitore.c
    ├── library.c
    ├── library.h
    ├── makefile
    ├── master.c
    ├── parameters.txt
    ├── README.md
    └── Relazione.pdf
```


### File principali individuati
- Makefile: Sistemi-Operativi-main/makefile
- Sorgenti: 7 file (C/C++). Esempi: Sistemi-Operativi-main/alimentazione.c, Sistemi-Operativi-main/atomo.c, Sistemi-Operativi-main/attivatore.c, Sistemi-Operativi-main/inibitore.c, Sistemi-Operativi-main/library.c…
- Configurazione: Sistemi-Operativi-main/parameters.txt
- Token/concetti rilevati nel codice: ENERGY, EXPLODE, MIN_N_ATOMICO, N_ATOM_MAX, N_NUOVI_ATOMI, SIM_DURATION, _GNU_SOURCE, alimentazione, atomo, attivatore, inibitore, master, msg, semaforo, shm, signal


## Compilazione
È fornito un `Makefile`. Comandi tipici:
```bash
make            # compila i target principali
make clean      # rimuove i binari/artefatti
```
Compilazione consigliata con `gcc -Wvla -Wextra -Werror` e macro `_GNU_SOURCE` abilitate.


## Esecuzione
Eseguibili/target principali attesi:

- `alimentazione`
- `atomo`
- `attivatore`
- `inibitore`
- `master`

### Configurazione a runtime
I parametri possono essere impostati tramite **variabili d'ambiente** o **file di configurazione** (se presente). Esempio di variabili comunemente usate:
- `SIM_DURATION`: durata massima della simulazione (secondi);
- `ENERGY_DEMAND`: energia prelevata dal master ogni secondo;
- `ENERGY_EXPLODE_THRESHOLD`: soglia per terminazione `explode`;
- `MIN_N_ATOMICO`, `N_ATOMI_INIT`, `N_ATOM_MAX`, `N_NUOVI_ATOMI`, `STEP`: parametri del modello;
- `INIBITORE=1`: abilita il processo inibitore (se implementato).

Esempio di esecuzione:
```bash
export SIM_DURATION=30
export ENERGY_DEMAND=10
export ENERGY_EXPLODE_THRESHOLD=10000
export N_ATOMI_INIT=4
export N_ATOM_MAX=16
export MIN_N_ATOMICO=1
export N_NUOVI_ATOMI=2
export STEP=1000000
# make && ./master     # oppure il binario principale definito nel Makefile
```


## Sincronizzazione e IPC
Il progetto utilizza meccanismi **POSIX/System V** per:
- **memoria condivisa** per stato/statistiche globali;
- **semafori** per mutua esclusione e sincronizzazione;
- **code di messaggi** e/o **pipe** per i comandi dell’attivatore verso gli atomi;
- **segnali** per start/stop e terminazione controllata.

Tutte le risorse IPC vengono deallocate al termine della simulazione o in caso di errore.


## Casi di terminazione
La simulazione termina per uno dei seguenti motivi:
- `timeout`: raggiunta `SIM_DURATION`;
- `explode`: energia netta > `ENERGY_EXPLODE_THRESHOLD`;
- `blackout`: domanda di energia > energia disponibile;
- `meltdown`: errore nelle `fork()` di qualunque processo;
- (con inibitore attivo) le condizioni di `explode`/`meltdown` dovrebbero essere rese improbabili.

All'uscita vengono stampate **statistiche aggregate** e la **causa di terminazione**.


## Testing rapido
Sono suggerite configurazioni di prova per verificare tutti i casi di terminazione. Esempio:
```bash
# explode
export SIM_DURATION=120 ENERGY_DEMAND=1 ENERGY_EXPLODE_THRESHOLD=2000
# blackout
export SIM_DURATION=60 ENERGY_DEMAND=999999
# timeout
export SIM_DURATION=5
# meltdown (forzare fallimenti di fork con limiti di risorse del sistema o simulazioni nel codice)
```
Adattare i parametri alla propria implementazione.


## Linee guida di sviluppo
- Progettazione modulare: processi lanciati con `execve(...)` da eseguibili distinti.
- Massimizzare la concorrenza tra processi.
- Evitare l'attesa attiva (busy-wait).
- Log periodico **ogni secondo**: attivazioni, scissioni, energia prodotta/consumata, scorie, (ed energia assorbita/inibizione se presente).
- Compatibile con sistemi multi-core.


## Struttura del repository
- `src/` o cartelle equivalenti: moduli di processo (master/atomo/attivatore/alimentazione[/inibitore]).
- `include/`: header comuni (se presenti).
- `Makefile`: regole di build.
- `config/` o file `.ini/.json`: parametri di simulazione (opzionale).
- `scripts/`: utility per test o lancio.
