# Fase 5 — Packaging & rifinitura

- **Tipo:** prodotto
- **Rischio:** basso
- **Effort:** S/M
- **Dipende da:** Fase 4
- **Stato:** Da iniziare

## Obiettivo

Decidere la forma finale di distribuzione e sistemare i contorni.

## Le due opzioni

### A) App unica con selettore servizio
- ➕ UX unificata (coda/cronologia/preferiti condivisi), una sola release.
- ➖ Binario più grande, config più complessa, branding "generico".

### B) Due app sottili su libreria statica condivisa
- ➕ Isolamento del blast-radius (Tidal si rompe → Spotify no), branding e release
  indipendenti, app piccole e focalizzate.
- ➖ Due shell/packaging da mantenere; niente coda cross-servizio.

**Orientamento attuale:** B (due app su core condiviso), a meno che la visione sia
esplicitamente "il player musicale universale di Haiku" → allora A.

## Lavoro (checklist)

- [ ] Estrarre il core come **libreria statica** condivisa.
- [ ] Selettore servizio (opzione A) **oppure** secondo target app (opzione B).
- [ ] Persistenza **per-servizio**: token, coda, qualità separati.
- [ ] Icone / branding per ciascun target.
- [ ] `rdef` + `make-hpkg.sh` per ogni app; release GitHub.

## Definition of Done

- Pacchetto/i `.hpkg` distribuibili, installabili con `pkgman`, con icona e metadati
  corretti; release pubblicata.

## Note

Decisione a costo basso ormai: una volta che il core è una libreria, passare da
un'app a due (o viceversa) è quasi solo lavoro di shell/packaging.

## Aggiornamento (loop rifiniture)

- [x] `lib:libFLAC` aggiunto ai requires dell'hpkg (dipendenza runtime del FlacDecoder).
