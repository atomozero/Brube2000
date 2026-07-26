# Roadmap — Multi-service (Tidal) & core condiviso

Piano per trasformare Brube2000 da client Spotify a **core riusabile + backend per servizio**,
e aggiungere **Tidal** come secondo backend.

## Idea di fondo

Gran parte di Brube2000 è già neutra rispetto al servizio (GUI, Player, coda,
`AudioOutput`/BSoundPlayer/nodo Cortex) e `AudioSource` è già un'astrazione.
Servono due "cuciture":

1. **`MusicService`** — un'interfaccia dietro cui vive tutto il codice specifico del
   servizio (auth, ricerca, libreria, metadati, apertura stream).
2. **`AudioDecoder`** — un'interfaccia che separa la decodifica dal formato
   (Vorbis per Spotify; FLAC/AAC per Tidal).

Poi "una app o due" è una scelta di **packaging** secondaria (vedi Fase 5).

## Le fasi

| Fase | Titolo | Tipo | Rischio | Stato |
|---|---|---|---|---|
| [0](phase-0-preparation.md) | Preparazione (doc + decisioni) | doc | nullo | Da iniziare |
| [1](phase-1-musicservice.md) | Estrazione `MusicService` | refactor | basso | Da iniziare |
| [2](phase-2-audiodecoder.md) | Estrazione `AudioDecoder` | refactor | basso | Da iniziare |
| [3](phase-3-tidal-readonly.md) | Backend Tidal read-only | nuovo | medio | Da iniziare |
| [4](phase-4-tidal-playback.md) | Playback Tidal | nuovo | **alto** | Da iniziare |
| [5](phase-5-packaging.md) | Packaging & rifinitura | prodotto | basso | Da iniziare |

## Principi

- **Spotify non si rompe mai**: ogni fase è verificabile e non distruttiva; da Fase 2
  in poi ogni fase è un potenziale punto di rilascio.
- **Confine valori invariato**: streaming sì, ripping / aggirare DRM no. MQA/Atmos
  fuori scope iniziale.
- Se si accorpa: **1+2** vanno insieme (refactor puro), **3+4** sono la "feature Tidal".

## Nota sul versionamento

Il repo pubblico è attualmente *solo documentazione* con un `.gitignore` a whitelist
(README + screenshots). Per pubblicare anche questa roadmap serve aggiungere
`!/docs/` alla whitelist; altrimenti resta locale (utile se la si vuole tenere privata).
