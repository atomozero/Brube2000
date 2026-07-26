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

## D2 — Packaging (preliminare)

- Orientamento: **due app sottili su una libreria statica condivisa** (Brube2000 per
  Spotify, una seconda app per Tidal).
- Alternativa: **app unica con selettore servizio**, se la visione diventa "il player
  musicale universale di Haiku".

**Motivo:** entrambi i backend sono reverse-engineering fragili e c'è un solo
sviluppatore. Due app isolano il *blast-radius* (se Tidal si rompe, Spotify no) e
permettono brand e cadenza di rilascio indipendenti. Si perde la coda cross-servizio,
ritenuta meno importante dell'isolamento.

**Decisione definitiva rimandata alla Fase 5** (a costo basso una volta che il core è
una libreria).

_Stato: raccomandazione — confermare._
