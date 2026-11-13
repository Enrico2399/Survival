# Arcane Survival

Gioco browser single-player in stile "survivors-like" (bullet heaven, genere Vampire Survivors): sopravvivi a orde infinite di nemici, sali di livello scegliendo potenziamenti, fondi gli incantesimi in versioni evolute e fai crescere il tuo Santuario tra una run e l'altra.

## Come si gioca

1. Apri `index.html` in un browser (Chrome, Edge, Firefox, Safari). Non serve internet, non serve installare nulla: è un unico file HTML autosufficiente.
2. Muoviti con **WASD** o le **frecce** (da desktop) oppure trascinando lo stick virtuale (da mobile/touch).
3. Gli incantesimi sparano in automatico: il tuo compito è muoverti, raccogliere le gemme XP ed evitare i nemici.
4. Quando sali di livello scegli una carta tra quelle proposte: **1 / 2 / 3** da tastiera, oppure clic/tap.
5. Sconfiggi i Guardiani (mini-boss) per ottenere forzieri con potenziamenti garantiti.
6. Alla fine della run, l'oro guadagnato si spende nel **Santuario** (progressione permanente tra una partita e l'altra): edifici, **Albero delle Abilità**, relitti e produzione passiva anche a gioco chiuso.

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

## Nuove feature (seconda ondata di implementazione)

**Nuovi contenuti di gioco:**
- Nuovo incantesimo base: **Aura Glaciale** (danno ad area attorno al giocatore + rallenta i nemici colpiti)
- Due nuovi tipi di nemico: **Dasher** (carica a scatti ad alta velocità) e **Splitter** (alla morte si divide in 2 Ombre più deboli)
- Varianti dei Guardiani (mini-boss): oltre al classico corpo a corpo, ora possono essere **a carica** o **a distanza** (questi ultimi telegrafano ed eseguono attacchi ad area a terra)
- **Boss finale** che compare al minuto 12 di ogni run, con una seconda fase (si attiva sotto il 50% di vita) che intensifica l'attacco
- 7ª struttura del Santuario: **Cristallo dell'Eco**, genera sia Essenza che Frammenti d'Anima
- Palette visiva di sfondo che cambia ogni 3 minuti di run (puramente estetico, nessun impatto sul bilanciamento)

**Feedback e qualità della vita:**
- Screen-shake e leggero "hit-stop" (rallentamento di un istante) su colpi critici e uccisioni di boss
- Scia luminosa dietro ai proiettili
- Vibrazione del dispositivo estesa a più eventi di gioco (mobile)
- Riepilogo dettagliato di fine run: danno inflitto per arma, nemici uccisi per tipo
- Storico delle scelte di livello (quali potenziamenti/fusioni hai preso) consultabile in pausa
- Classifica locale (Top 5) delle run migliori, salvata sul dispositivo

**Albero delle Abilità (nuova tab "Albero" del Santuario, separata dall'Altare):**
- L'Altare resta esattamente come prima (griglia piatta dei 6 potenziamenti permanenti, nessuna modifica).
- Nuova 5ª tab del Santuario con un vero albero di potenziamenti, acquistato con l'Essenza Arcana: un **Nucleo Arcano** centrale (100 ✨, +5% danno magico generale) che va risvegliato per sbloccare 3 rami tematici.
  - **Vitalità** 🩸: Rubavita (recupera HP in base al danno inflitto, generale) → Rigenerazione Profonda (richiede Rubavita Grado 2)
  - **Fulmini** ⚡: Sovraccarico (+danno a Tempesta di Fulmini) → Catena Prolungata (bersagli extra garantiti, richiede Sovraccarico Grado 3)
  - **Gelo** ❄️: Gelo Profondo (+danno ad Aura Glaciale) → Tempesta Glaciale (+raggio, richiede Gelo Profondo Grado 3)
- Ogni nodo figlio si sblocca solo quando il suo nodo radice raggiunge il grado richiesto, con un connettore visivo che mostra la dipendenza; tutti i rami richiedono il Nucleo già risvegliato.
- I bonus sono reali e persistenti tra una run e l'altra: rubavita cura in combattimento, rigenerazione aumenta la rigenerazione passiva di HP, i bonus di Fulmini/Gelo aumentano rispettivamente il danno (e le scariche garantite) di Tempesta di Fulmini e il danno/raggio di Aura Glaciale.

**Accessibilità:**
- Interruttore "Riduci bagliore" (disattiva gli effetti `shadowBlur` per chi preferisce un'interfaccia più sobria o ha un dispositivo meno potente)
- Slider per la scala dell'interfaccia (HUD) in gioco

**Non implementato in questa ondata (valutato ma rimandato per invasività/rischio rispetto al tempo disponibile — fattibile su richiesta):**
- Arma turret (torretta stazionaria)
- Missioni giornaliere
- Sistema di prestigio/ascensione
- Codice di run seminato e condivisibile

## Verifiche effettuate

Testato con una simulazione accelerata di un'intera run da 15 minuti (oltre la comparsa del boss finale) e con emulazione di viewport/touch mobile: nessun errore riscontrato, tutti i nuovi sistemi (Aura Glaciale, Dasher/Splitter, varianti Guardiani, boss finale, Santuario a 7 strutture, riepilogo di fine run, classifica, controlli di accessibilità) funzionano correttamente sia da desktop che da mobile.

## Prossimi passi possibili

Vedi il documento di analisi completo (Priorità 2: bilanciamento/economia; Priorità 3: contenuti; Priorità 4: rifinitura) per l'elenco dettagliato delle migliorie ancora da valutare, incluse le feature rimandate sopra.
