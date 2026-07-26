# Architettura — core condiviso & backend per servizio

Documento di riferimento per l'evoluzione da client Spotify a **core riusabile +
backend per servizio** (vedi [roadmap](roadmap/README.md)).

## Strati

```
  ┌─────────────────────────────────────────────┐
  │  GUI  (Interface Kit)                         │  service-agnostic
  ├─────────────────────────────────────────────┤
  │  Player  (coda, stato, auto-advance, seek)    │  service-agnostic
  ├───────────────┬─────────────────────────────┤
  │ MusicService  │  AudioDecoder                 │  ← le due cuciture
  │ (per servizio)│  (per formato)                │
  ├───────────────┴─────────────────────────────┤
  │  AudioOutput  (BSoundPlayer / nodo Cortex)    │  service-agnostic
  └─────────────────────────────────────────────┘
```

Gran parte del codice è **già** neutra rispetto al servizio (GUI, Player, coda,
`AudioOutput`) e `AudioSource` è **già** un'astrazione (lo implementa `TrackStream`).
Il lavoro consiste nell'introdurre due cuciture e spostare dietro di esse il codice
specifico del servizio.

## Cucitura 1 — `MusicService`

Interfaccia dietro cui vive tutto ciò che è specifico del servizio: auth, ricerca,
libreria, metadati, apertura dello stream audio.

```
struct AudioStream {
    std::shared_ptr<AudioSource> source;   // già astratto
    enum Format { Vorbis, Flac, Aac, Pcm } format;
    int   sampleRateHint, channelsHint;
    int64 totalBytes;                       // per buffer/seek (0 se ignoto)
};

class MusicService {
    virtual bool Available() = 0;
    virtual bool Login(std::string& err) = 0;
    virtual std::string Name() = 0;

    virtual std::vector<SearchResult> Search(txt, limit, err) = 0;
    virtual TrackInfo ResolveTrack(id, err) = 0;

    virtual std::vector<LibraryItem> LikedTracks(err) = 0;
    virtual std::vector<LibraryItem> Playlists(err) = 0;
    virtual std::vector<LibraryItem> PlaylistTracks(id, err) = 0;
    virtual ArtistPage GetArtist(id) = 0;
    virtual std::vector<LibraryItem> AlbumTracks(id, err) = 0;

    virtual std::unique_ptr<AudioStream> OpenStream(id, quality, err) = 0;
};
```

- `SpotifyService` implementa `OpenStream` con l'attuale catena
  `MercuryGet + PickVorbis + RequestAudioKey + new TrackStream` → `format=Vorbis`.
- `TidalService` la implementa con `playbackinfo → manifest → fetcher a segmenti`
  → `format=Flac/Aac`.

Il **Player** cambia pochissimo: chiama `service->OpenStream()` e riceve un
`AudioStream`. Coda, auto-advance, seek-once-downloaded, VU, persistenza restano uguali.

## Cucitura 2 — `AudioDecoder`

Separa la decodifica dal formato. Il sink (`AudioOutput`) è già condiviso.

```
class AudioDecoder {
    virtual bool  Open(std::shared_ptr<AudioSource> src) = 0;
    virtual size_t ReadBytes(void* buf, size_t n) = 0;   // int16 interleaved
    virtual bool  SeekFrame(int64 frame) = 0;
    virtual int64 TotalFrames() = 0;
    virtual int   SampleRate() = 0;
    virtual int   Channels() = 0;
};
```

`VorbisDecoder` ha già questa forma. Il Player sceglie il decoder da
`AudioStream::format` (factory: `Vorbis→VorbisDecoder`, `Flac→FlacDecoder`, ...).

## Tipi neutri (già esistenti)

`LibraryItem`, `AlbumRef`, `ArtistPage`, `SearchResult`, `TrackInfo`, `QueueEntry`
sono già quasi indipendenti dal servizio → vanno spostati in un header condiviso.
Il campo `uri`/`id` diventa un identificatore opaco interpretato dal backend.

## Packaging

"Una app con selettore" vs "due app su libreria statica condivisa" è una scelta di
**Fase 5**, a costo basso una volta che il core è una libreria. Orientamento attuale
in [decisions.md](decisions.md).

## Non-goal (confine valori)

- Solo **streaming** di contenuti riproducibili in chiaro. Nessun salvataggio su
  file / ripping, nessun aggiramento di DRM.
- **MQA** e **Dolby Atmos** (proprietari) esclusi.
