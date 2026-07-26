# Fase 0 — Preparazione (doc + decisioni)

- **Tipo:** documentazione / decisioni
- **Rischio:** nullo
- **Effort:** ~mezza sessione (opzionale)
- **Dipende da:** —
- **Stato:** Da iniziare

## Obiettivo

Fissare le decisioni e la mappa prima di toccare codice, così le fasi successive non
si bloccano su scelte di fondo.

## Lavoro (checklist)

- [ ] `docs/architecture.md`: schema a strati (GUI → Player → `MusicService` /
      `AudioDecoder` / `AudioOutput`) con i confini chiari.
- [ ] Decidere il **naming**: Brube2000 resta il brand Spotify? Il core condiviso
      come si chiama (es. `libbrube` / `MediaCore`)?
- [ ] Decisione preliminare **app-unica-con-selettore** vs **due-app-su-libreria**
      (rivedibile in Fase 5, ma orienta il refactor).
- [ ] Confermare lo scope: MQA / Dolby Atmos / DRM forte **esclusi**.

## Definition of Done

- Esiste un documento di riferimento condiviso e le tre decisioni (naming, packaging
  preliminare, scope) sono scritte.

## Note

Fase saltabile: se hai già le idee chiare, si può partire dalla Fase 1 e scrivere il
doc man mano.
