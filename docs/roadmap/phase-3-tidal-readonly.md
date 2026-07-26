# Fase 3 — Backend Tidal read-only

- **Tipo:** nuovo (primo codice davvero nuovo)
- **Rischio:** medio (API non ufficiale) — ma **isolato**: se rompe, Spotify non ne risente
- **Effort:** M
- **Dipende da:** Fasi 1–2
- **Stato:** Da iniziare

## Obiettivo

Dimostrare che l'astrazione `MusicService` regge con un servizio davvero diverso,
**senza playback**. Login + navigazione nella stessa GUI.

## Lavoro (checklist)

- [ ] `TidalService : MusicService`.
- [ ] **Auth**: OAuth2 device-flow; salvataggio token (riusa `TokenStore`/`HttpsClient`).
- [ ] **Search**: `GET /search` → riempi `SearchResult`.
- [ ] **Metadati**: `ResolveTrack` da `GET /tracks/{id}` → `TrackInfo`
      (nome/artista/album/anno/cover/durata).
- [ ] **Libreria**: liked, playlist, playlist tracks, artista, album → riempi
      `LibraryItem` / `ArtistPage` / `AlbumRef` (REST + JSON via `Json.cpp`).
- [ ] **Cover art**: URL immagine Tidal al posto di `i.scdn.co`.
- [ ] `OpenStream()` ritorna un errore "not implemented" (playback in Fase 4).

## Definition of Done

- Login Tidal riuscito; puoi cercare, aprire la libreria, sfogliare artisti/album
  nella GUI esistente. Il tasto Play mostra un messaggio "non ancora disponibile".

## Rischi e mitigazioni

- *API non ufficiale/instabile*: incapsulare tutte le chiamate in `TidalService`;
  degradare con messaggi chiari invece di crashare.
- *Client id/token*: usare un flusso di auth pulito (device-flow), gestione errori
  esplicita.

## Note

Questa è la fase-prova dell'architettura: se i tipi neutri reggono un secondo
servizio senza modifiche, il refactor delle Fasi 1–2 è validato.
