# Fase 2 — Estrazione `AudioDecoder`

- **Tipo:** refactor (Spotify invariato)
- **Rischio:** basso
- **Effort:** S
- **Dipende da:** Fase 1
- **Stato:** Da iniziare

## Obiettivo

Separare la **decodifica** dal **formato**, così la pipeline può accogliere FLAC/AAC
(Tidal) oltre a Vorbis (Spotify). Il sink audio (`AudioOutput` / BSoundPlayer / nodo
Cortex) è già condiviso e **non si tocca**.

## Interfaccia (bozza)

```
class AudioDecoder {
    virtual bool  Open(std::shared_ptr<AudioSource> src) = 0;
    virtual size_t ReadBytes(void* buf, size_t n) = 0;   // int16 interleaved PCM
    virtual bool  SeekFrame(int64 frame) = 0;
    virtual int64 TotalFrames() = 0;
    virtual int   SampleRate() = 0;
    virtual int   Channels() = 0;
};
```

`VorbisDecoder` ha **già** questa forma → basta farlo derivare dall'interfaccia.

## Lavoro (checklist)

- [ ] Definire `AudioDecoder`.
- [ ] `VorbisDecoder : AudioDecoder` (rinomini minimi).
- [ ] Il `Player` sceglie il decoder in base a `AudioStream::format`
      (factory: `Vorbis → VorbisDecoder`).
- [ ] `AudioOutput`/`AudioPlayer` accettano un `AudioDecoder*` generico (già
      lavorano su PCM int16, quindi cambia solo il tipo).

## Definition of Done

- Spotify suona **identico** a oggi, incluso seek-once-downloaded e lo swap del
  decoder, ma ora attraverso l'interfaccia generica.

## Note

Dopo questa fase hai un **core riusabile completo**, ancora mono-servizio: buon punto
di rilascio intermedio (nessuna feature nuova visibile, ma base pronta).
