# Emo-Clone 🤖

> Robot da scrivania emotivo e interattivo, ispirato a Emo (Living.ai), sviluppato come progetto personale — **work in progress**.

Emo-Clone è un compagno robotico da scrivania che reagisce, ascolta, parla, riconosce i volti e sviluppa una piccola "personalità" nel tempo. Il progetto nasce come esercizio pratico di elettronica, programmazione embedded, computer vision e sviluppo Android, in preparazione al percorso di studi in Informatica e all'obiettivo futuro di Robotica al Politecnico di Milano.

⚠️ **Stato del progetto:** in fase di progettazione/sviluppo iniziale. Non tutte le funzioni elencate sono ancora implementate — questo README descrive sia lo stato attuale sia la roadmap completa.

---

## 📋 Indice

- [Panoramica](#panoramica)
- [Funzionalità](#funzionalità)
- [Hardware](#hardware)
- [Software e stack tecnologico](#software-e-stack-tecnologico)
- [Architettura del sistema](#architettura-del-sistema)
- [Macchina a stati](#macchina-a-stati)
- [Setup e installazione](#setup-e-installazione)
- [App companion Android](#app-companion-android)
- [Roadmap](#roadmap)
- [Struttura del repository](#struttura-del-repository)
- [Autore](#autore)

---

## Panoramica

Emo-Clone è composto da:
1. **Il robot fisico** — testa mobile con schermo per occhi animati, camera, microfono/i, altoparlante, base con motori per il movimento pan-tilt (e in futuro braccetti laterali).
2. **Il "cervello"** — un board (es. Raspberry Pi / microcontrollore, da definire in base al carico computazionale) che gestisce percezione, stato emotivo e risposte.
3. **L'app companion Android** — realizzata in Kotlin/Jetpack Compose, comunica col robot per configurazione, notifiche e controllo remoto.

---

## Funzionalità

### ✅ Core (fase 1)
- Occhi animati sullo schermo con **espressioni multiple** (felice, curioso, annoiato, sorpreso, ecc.)
- **Testa pan-tilt** coordinata con lo sguardo/espressione
- **Riconoscimento volto** via camera
- **Ascolto e risposta vocale offline** (Vosk per lo speech-to-text, pyttsx3/espeak-ng per il text-to-speech)
- **Macchina a stati** che gestisce il comportamento: `IDLE`, `ASCOLTO`, `PARLA`, `GUARDA_VOLTO`, `ANNOIATO`
- Sensore **PIR** opzionale per rilevare presenza/movimento nella stanza

### 🎯 Estensioni pianificate (fase 2)
- **Localizzazione della sorgente sonora** con array di microfoni (es. ReSpeaker), per girare la testa verso chi parla ancora prima di vederlo con la camera
- Integrazione opzionale con **API AI** (es. Claude) per risposte più intelligenti e naturali
- Uso del robot come **altoparlante** tramite il microfono del telefono
- **Sincronizzazione selettiva delle notifiche** del telefono (l'utente sceglie quali app notificare tramite il robot)
- **Due braccetti laterali** controllabili dall'app

### 🚀 Funzioni avanzate (da documento MVP)
- **Wake word** dedicata (attivazione vocale)
- Sensore **touch** sulla testa
- Sensore di **prossimità/distanza** (ToF o laser)
- Sensore di **luce ambientale**
- **IMU** (accelerometro + giroscopio)
- Sensori **anti-caduta / edge detection** sulla base
- **Movimento autonomo** sulla scrivania (spostamenti, passeggiate, routine animate)
- **Riconoscimento e memoria di più utenti/volti**
- **Memoria personale**: nome, preferenze, eventi ricorrenti, date speciali (es. compleanno)
- **Personalità evolutiva** con variabili interne: energia, umore, curiosità, sonno, socialità
- Funzioni pratiche vocali: ora, sveglie, meteo
- **Base di ricarica dock** (idealmente wireless, con LED di stato)
- **Aggiornamenti OTA** via app

---

## Hardware

> ⚠️ Lista indicativa — da aggiornare man mano che i componenti vengono scelti e testati.

| Componente | Funzione | Stato |
|---|---|---|
| Board di controllo (Raspberry Pi / MCU) | "Cervello" del robot | Da definire |
| Camera | Riconoscimento volto, visione | Pianificato |
| Microfono / array di microfoni (es. ReSpeaker) | Ascolto vocale, localizzazione sorgente sonora | Pianificato |
| Altoparlante | Output vocale | Pianificato |
| Schermo (occhi animati) | Espressioni facciali | Pianificato |
| Servomotori (pan-tilt testa) | Movimento testa | Pianificato |
| Servomotori (braccetti laterali) | Gestualità | Fase 2 |
| Sensore PIR | Rilevamento presenza | Opzionale |
| Sensore ToF/laser | Distanza/prossimità | Fase avanzata |
| Sensore di luce ambientale | Adattamento comportamento | Fase avanzata |
| IMU (accelerometro + giroscopio) | Orientamento, stabilità | Fase avanzata |
| Sensori edge/anti-caduta | Sicurezza sulla base mobile | Fase avanzata |
| Base di ricarica (dock) | Ricarica wireless + LED di stato | Fase avanzata |

---

## Software e stack tecnologico

**Firmware / logica robot:**
- Python (gestione stati, sensori, AI locale)
- Vosk — speech-to-text offline
- pyttsx3 / espeak-ng — text-to-speech offline
- OpenCV (o libreria equivalente) — riconoscimento volto
- Flask — API locale per la comunicazione con l'app

**App companion Android:**
- Kotlin
- Jetpack Compose (UI)
- Retrofit (comunicazione HTTP con il robot via Flask)

**Linguaggi già in uso dall'autore:** C/C++ (Arduino), HTML, CSS, JavaScript.

---

## Architettura del sistema

```
┌─────────────────────┐         ┌─────────────────────┐
│   Robot (board)      │◄──────►│   App Android         │
│                       │  HTTP   │   (Kotlin/Compose)    │
│  - Camera             │ Flask/  │                        │
│  - Microfono/i        │Retrofit │  - Configurazione      │
│  - Altoparlante       │         │  - Notifiche selettive │
│  - Schermo (occhi)    │         │  - Controllo braccetti │
│  - Motori pan-tilt    │         │  - Aggiornamenti OTA   │
│  - Sensori            │         │                        │
└─────────────────────┘         └─────────────────────┘
```

---

## Macchina a stati

Il comportamento del robot è gestito da una macchina a stati finiti (FSM):

```
IDLE ──► ASCOLTO ──► PARLA ──► IDLE
  │                              ▲
  ▼                              │
GUARDA_VOLTO ────────────────────┘
  │
  ▼
ANNOIATO ──► IDLE (dopo timeout senza interazioni)
```

- **IDLE**: stato di riposo, nessuna interazione in corso
- **ASCOLTO**: rilevata wake word o input vocale, il robot ascolta
- **PARLA**: il robot sta rispondendo (TTS)
- **GUARDA_VOLTO**: un volto è stato riconosciuto, il robot lo segue con lo sguardo
- **ANNOIATO**: nessuna interazione per un certo periodo, il robot mostra comportamenti "annoiati"

---

## Setup e installazione

> Sezione da completare man mano che il progetto avanza.

```bash
# Clona il repository
git clone https://github.com/<tuo-username>/emo-clone.git
cd emo-clone

# Crea un ambiente virtuale Python
python -m venv venv
source venv/bin/activate  # su Windows: venv\Scripts\activate

# Installa le dipendenze
pip install -r requirements.txt

# Avvia il robot
python main.py
```

### Requisiti
- Python 3.x
- Modello Vosk per la lingua italiana (da scaricare separatamente)
- Board/hardware assemblato secondo lo schema in `/hardware`

---

## App companion Android

L'app permette di:
- Configurare il robot (Wi-Fi, nome, preferenze)
- Scegliere quali notifiche del telefono sincronizzare
- Controllare manualmente i braccetti laterali
- Gestire gli aggiornamenti OTA del firmware

Repository/cartella dedicata: `/app-android` *(da creare)*

---

## Roadmap

- [ ] Fase 1 — Core: occhi animati, pan-tilt, riconoscimento volto, ascolto/risposta vocale, FSM base
- [ ] Fase 2 — Localizzazione sorgente sonora, integrazione API AI, altoparlante da telefono, notifiche selettive, braccetti laterali
- [ ] Fase 3 — Sensoristica avanzata (touch, ToF, luce, IMU, anti-caduta), movimento autonomo
- [ ] Fase 4 — Memoria multi-utente, personalità evolutiva, funzioni vocali pratiche, dock di ricarica, OTA

---

## Struttura del repository

```
emo-clone/
├── firmware/           # Codice Python del robot (stati, sensori, AI)
├── app-android/        # App companion Kotlin/Jetpack Compose
├── hardware/           # Schemi, distinta componenti, modelli 3D
├── docs/               # Documentazione tecnica e MVP
└── README.md
```

---

## Autore

**MaiSbgaliare**

---

*Questo README verrà aggiornato progressivamente man mano che il progetto avanza.*
