# Fase 0 — Preparazione (doc + decisioni)

- **Tipo:** documentazione / decisioni
- **Rischio:** nullo
- **Effort:** ~mezza sessione (opzionale)
- **Dipende da:** —
- **Stato:** ✅ Fatto

## Obiettivo

Fissare le decisioni e la mappa prima di toccare codice, così le fasi successive non
si bloccano su scelte di fondo.

## Lavoro (checklist)

- [x] `docs/architecture.md`: schema a strati (GUI → Player → `MusicService` /
      `AudioDecoder` / `AudioOutput`) con i confini chiari.
- [x] Decidere il **naming** → `docs/decisions.md` D1.
- [x] Decisione preliminare **app-unica** vs **due-app** → `docs/decisions.md` D2.
- [x] Confermare lo scope (MQA / Atmos / DRM esclusi) → `docs/decisions.md` D3.

## Definition of Done

- Esiste un documento di riferimento condiviso e le tre decisioni (naming, packaging
  preliminare, scope) sono scritte.

## Note

Fase saltabile: se hai già le idee chiare, si può partire dalla Fase 1 e scrivere il
doc man mano.
