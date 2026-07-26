# Decisioni (Fase 0)

Registro delle decisioni di fondo. Ognuna è una **raccomandazione motivata** e
resta **rivedibile** nelle fasi successive (in particolare il packaging in Fase 5).

## D1 — Naming

- **Brube2000** resta il nome dell'app **Spotify**.
- Il **core condiviso** viene estratto come libreria statica, nome di lavoro
  `libmusiccore` (namespace `brube::core`).
- Il futuro backend **Tidal** avrà la propria app (coerente con l'orientamento a due
  app, vedi D2); nome provvisorio **da definire**.

**Motivo:** "Brube2000" ha già identità e release proprie legate a Spotify; forzarlo
a nome generico diluirebbe il brand. Meglio un core neutro + app con brand distinto.

_Stato: raccomandazione — confermare._
