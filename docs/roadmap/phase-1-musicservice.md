# Fase 1 — Estrazione `MusicService`

- **Tipo:** refactor (Spotify invariato)
- **Rischio:** basso
- **Effort:** M
- **Dipende da:** — (Fase 0 opzionale)
- **Stato:** Da iniziare

## Obiettivo

Introdurre l'interfaccia `MusicService` e farci passare **tutto** il codice Spotify
attuale come `SpotifyService`, **senza cambiare comportamento**. È il ~70% del valore
architetturale a rischio quasi zero.

## Interfaccia (bozza)

```
struct AudioStream {
    std::shared_ptr<AudioSource> source;   // già astratto (TrackStream / ...)
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

## Lavoro (checklist)

- [ ] Spostare `LibraryItem`, `AlbumRef`, `ArtistPage`, `SearchResult`, `TrackInfo`
      in un header neutro condiviso (sono già quasi service-agnostici).
- [ ] Definire `MusicService` + `AudioStream`.
- [ ] `SpotifyService : MusicService` che incapsula `Session` + `Library` + `Search`.
- [ ] `OpenStream()` = attuale catena `MercuryGet + PickVorbis + RequestAudioKey +
      new TrackStream` → ritorna `AudioStream{ source=TrackStream, format=Vorbis }`.
- [ ] Il `Player` chiama `fService->...` invece di `fSession`/`fLibrary` diretti.
- [ ] Le finestre (Library/Artist) ricevono un `MusicService*` invece di `Library*`.

## Definition of Done

- L'app si comporta **identica** a oggi: login, ricerca, libreria, pagine
  artista/album, playback, seek, coda persistente, VU, qualità audio. Nessuna
  regressione osservabile.

## Rischi e mitigazioni

- *Regressione durante lo spostamento*: procedere a piccoli passi, ricompilando e
  rilanciando spesso; mantenere la firma dei metodi identica dove possibile.

## Note

`AudioSource` è già un'astrazione (lo implementa `TrackStream`), quindi `OpenStream`
non richiede nuovi concetti lato Player — solo di restituire l'oggetto giusto.
