# Fase 4 — Playback Tidal

- **Tipo:** nuovo (la fase più pesante)
- **Rischio:** **alto** (decoder + fetcher + eventuale cifratura)
- **Effort:** M/L
- **Dipende da:** Fase 3
- **Stato:** Da iniziare

## Obiettivo

Far **suonare** Tidal: dalla richiesta di riproduzione allo streaming reale, con seek
e VU come su Spotify.

## Lavoro (checklist)

- [x] **Migrazione streaming Spotify** (fatta in anticipo): `OpenStream()` +
      `AudioStream` aggiunti a `MusicService`; la catena di streaming è dentro
      `SpotifyService::OpenStream` (sessione di playback propria) e il Player
      consuma un `AudioStream` — è **backend-agnostic**. Verificato: login,
      riproduzione, seek, coda invariati. Restano solo i pezzi Tidal-specifici
      qui sotto.
- [ ] **`OpenStream`** (Tidal): `playbackinfo` per la traccia alla qualità scelta →
      ottieni URL/manifest (HLS o DASH).
- [ ] **Fetcher a segmenti**: un `AudioSource` che scarica i segmenti in sequenza in
      un buffer che cresce (stesso pattern di `TrackStream`, ma HTTP invece di chunk
      AP). Supporta "suona subito, scarica in background".
- [ ] **`FlacDecoder : AudioDecoder`** con `libFLAC` (presente su Haiku).
- [ ] *(opzionale)* **`AacDecoder`** per i tier lossy (via Media Kit codec o ffmpeg).
- [ ] Seek: riusa la logica "seekable once downloaded" già esistente, adattata al
      contenitore FLAC/AAC.
- [ ] Gestione qualità Tidal mappata sul selettore qualità esistente.

## Definition of Done

- Una traccia Tidal (FLAC) parte in streaming in pochi secondi, con **seek**, **VU**,
  volume e auto-advance funzionanti come su Spotify.

## Fuori scope (confine valori)

- **MQA** e **Dolby Atmos** (proprietari) → esclusi.
- Tracce con **DRM forte** → non aggirare. Solo streaming di ciò che è riproducibile
  in chiaro. Nessun salvataggio su file / ripping.

## Rischi e mitigazioni

- *Formati manifest e cifratura variabili*: iniziare dal caso più semplice (FLAC
  diretto), poi HLS/DASH; isolare tutto in `TidalService`/decoder.
- *Timing/underrun dei segmenti*: riusare il pattern di prebuffer/ring già collaudato.

## Note

È la fase dove concentrare la cura e i test. Il trasporto Tidal è HTTP/JSON (più
semplice dello stack cripto Spotify); la difficoltà vera sono i **decoder** e il
**fetcher a segmenti**.
