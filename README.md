# Arcane Survival

Gioco browser single-player in stile "survivors-like" (bullet heaven, genere Vampire Survivors): sopravvivi a orde infinite di nemici, sali di livello scegliendo potenziamenti, fondi gli incantesimi in versioni evolute e fai crescere il tuo Santuario tra una run e l'altra.

## Come si gioca

1. Apri `index.html` in un browser (Chrome, Edge, Firefox, Safari). Non serve internet, non serve installare nulla: è un unico file HTML autosufficiente.
2. Muoviti con **WASD** o le **frecce** (da desktop) oppure trascinando lo stick virtuale (da mobile/touch).
3. Gli incantesimi sparano in automatico: il tuo compito è muoverti, raccogliere le gemme XP ed evitare i nemici.
4. Quando sali di livello scegli una carta tra quelle proposte: **1 / 2 / 3** da tastiera, oppure clic/tap.
5. Sconfiggi i Guardiani (mini-boss) per ottenere forzieri con potenziamenti garantiti.
6. Alla fine della run, l'oro guadagnato si spende nel **Santuario** (progressione permanente tra una partita e l'altra): edifici, relitti e produzione passiva anche a gioco chiuso.

## Salvataggio

- Il progresso (Santuario, relitti, achievement) viene salvato automaticamente nel `localStorage` del browser.
- Dalla schermata del Santuario puoi **esportare** il salvataggio su file, **importarlo** su un altro dispositivo/browser, o **resettarlo** da zero.
- Nota: se apri il gioco come pagina pubblicata (artifact) invece che come file locale, il pulsante di esportazione potrebbe non riuscire a scaricare il file per limitazioni dell'ambiente — in quel caso apri `index.html` scaricato in locale per usare l'export.

## Stato del progetto — cosa è stato corretto (Priorità 1)

**Bug risolti (10/10):**
- Timer condivisi tra incantesimo base e versione fusa (causavano cadenza di fuoco raddoppiata o dimezzata)
- Livelli passivi dai forzieri che non applicavano il relativo effetto
- Font canvas non valido per i numeri di danno critico
- Livelli di `mana_orbs` sprecati dopo la fusione in `solar_ring`/`orbital_tempest`
- Danno delle fusioni ora scala con il livello del giocatore (prima restava fisso)
- Bonus cooldown della Clessidra Eterea applicato immediatamente, non al ricalcolo successivo
- Badge "fusione disponibile" non più mostrato quando non c'è nessuna fusione disponibile
- Produzione passiva offline ricalcolata su tempo reale (niente più drift)
- Gioco ora completamente funzionante offline (CSS Tailwind compilato e incluso nel file, non più caricato da CDN)
- Shuffle delle carte di livello non più sbilanciato (Fisher-Yates)

**Prestazioni:**
- Rimosso l'uso di `ctx.shadowBlur` nel ciclo di disegno (era il collo di bottiglia principale a centinaia di entità) sostituendolo con sprite pre-renderizzate e culling fuori schermo → tempo di draw sceso da ~40ms a ~6-7ms a parità di nemici/gemme (~6x)
- Sostituite le scansioni complete dell'array nemici (targeting incantesimi, collisioni proiettili, retargeting omingante, separazione tra nemici) con una griglia spaziale
- Le gemme XP non si accumulano più indefinitamente: doppio sistema di merge (per cella vicina + per distanza dal giocatore)

**Altre migliorie:**
- Tasti 1/2/3 per scegliere le carte di livello
- Input touch e mouse unificato con Pointer Events (ora funziona bene anche trascinando con il mouse su desktop)
- Pausa automatica quando il tab passa in background, con recupero della produzione passiva al ritorno
- Export/Import/Reset del salvataggio

**Non ancora implementato (segnalato, non dimenticato):**
- Object pooling per proiettili/particelle/testi di danno — valutato a basso guadagno rispetto al rischio, dato che gli array coinvolti sono già piccoli e limitati e il vero collo di bottiglia (shadowBlur) è stato risolto altrove. Da fare su richiesta.

## Correzioni UI (13/11)

- **Santuario, icona struttura tagliata in basso**: il nodo della struttura posizionato in basso nell'esagono finiva nascosto/tagliato dietro la barra flottante "Riscatta risorse" sovrapposta al canvas, soprattutto con il riquadro all'altezza minima (210px). Ora viene riservato esplicitamente lo spazio per quella barra e l'intero layout delle strutture viene centrato solo nell'area libera rimanente.
- **Pop-up a lato sinistro illeggibile a metà (carte di livello)**: il badge numerico (1/2/3) che indica la scorciatoia da tastiera sulle carte di livello era posizionato parzialmente fuori dal bordo della card (`-top-2 -left-2`); il contenitore scorrevole delle carte taglia l'overflow orizzontale, quindi sulla prima carta a sinistra meta' del badge spariva. Spostato dentro i bordi della card (`top-2 left-2`), ora è sempre interamente visibile.
- **Pop-up in alto a sinistra tagliato a metà, poi sparisce (eventi/reliquie durante il gioco)**: il banner che compare quando raccogli una reliquia o scatta un evento (orda, boss, fusione...) è gia' centrato orizzontalmente dal suo contenitore, ma l'animazione di comparsa/scomparsa applicava ANCHE lei uno spostamento di meta' larghezza verso sinistra sul singolo banner: il doppio spostamento lo faceva apparire spostato verso l'angolo alto-sinistro e tagliato, invece che centrato in alto. Rimosso lo spostamento orizzontale superfluo dall'animazione: ora il banner compare e scompare centrato, senza spostamenti laterali.

## Prossimi passi possibili

Vedi il documento di analisi completo (Priorità 2: bilanciamento/economia; Priorità 3: contenuti; Priorità 4: rifinitura) per l'elenco dettagliato delle migliorie ancora da valutare.
