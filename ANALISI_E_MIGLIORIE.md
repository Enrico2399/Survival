# Arcane Survival — Analisi e migliorie

*2026-11-13 — Enrico Baldasso*

> Documento di analisi tecnica del gioco, con lo stato di avanzamento delle correzioni e migliorie proposte. Aggiornato dopo due ondate di implementazione (Priorità 1 completa + parte della Priorità 3).

## In sintesi

Il gioco è un survivors-like (stile Vampire Survivors) già sorprendentemente completo: loop di gioco solido, fusioni, reliquie permanenti, Santuario con produzione idle, traguardi e UI mobile-first curata. Gira a 60 fps nei primi minuti. I problemi veri individuati in origine erano quattro: **(1)** alcuni bug di logica che rompevano le fusioni, **(2)** le prestazioni crollavano nel late game (ombre canvas + gemme che si accumulano), **(3)** l'economia del Santuario si rompeva dopo una notte di produzione offline, **(4)** mancava varietà: 3 nemici, 4 magie, boss che cammina e basta, nessun obiettivo di fine run.

**Stato attuale:** i punti (1), (2) e (3) sono stati risolti nella prima ondata (Priorità 1). Il punto (4) è stato affrontato in buona parte nella seconda ondata: nuovi nemici, varianti dei Guardiani, un boss finale con una seconda fase, una nuova magia, oltre a diverse rifiniture di Priorità 4 (hit-stop, statistiche di fine run, accessibilità).

## Come l'ho provato

- Lettura completa del codice (ora oltre 4.300 righe, un unico file HTML).
- Partita reale in Chrome headless a risoluzione iPhone (390×844, touch), input simulato da tastiera e scelte automatiche al level-up.
- Simulazione accelerata di 10-15 minuti di gioco (con un "bot" che gira in cerchio e immortalità forzata) per osservare late game, fusioni, numero di entità e, nell'ultima ondata, comparsa del boss finale e di tutti i nuovi nemici/varianti.
- Misure mirate: frequenza di fuoco delle fusioni, effetto degli scrigni, costo del rendering con/senza ombre, conti dell'economia.

## Bug confermati (con prova) — tutti corretti ✅

| # | Bug | Prova | Correzione |
|---|-----|-------|------------|
| 1 | **Timer condivisi tra fusione e magia base.** Dopo una fusione, la magia base torna a 0 e può riapparire come "NUOVO!". Pioggia di Saette usa lo stesso `boltTimer` del Dardo, Comete Infernali lo stesso `novaTimer` della Nova. | Pioggia di Saette in 10s: 92 proiettili da sola, **184** con il Dardo ripreso (spara il doppio). Comete: 33 da sole, **15** con la Nova ripresa (dimezzate). | Timer separati per ogni magia/fusione, oppure togliere dal pool le magie già fuse. |
| 2 | **Scrigno del boss potenzia passivi senza applicarne l'effetto.** Il filtro include i passivi, ma fa solo `skills[id]++`. | Essenza Vitale portata a LV2 dallo scrigno: HP massimi restano 100 invece di 125. Stesso problema per Celerità, Passo Etereo, Magnete. | Far passare lo scrigno da `applyUpgrade()`, o limitarlo alle magie attive come dice il testo. |
| 3 | **Font dei critici non valido.** `'extrabold 16px Inter'` non è un font CSS valido: il canvas lo ignora e i critici escono uguali ai danni normali. | Verificato in Chrome: dopo l'assegnazione `ctx.font` resta `bold 13px`. | Usare `'800 16px Inter, sans-serif'`. |
| 4 | **Riprendere Orbi di Mana dopo Anello Solare / Tempesta Orbitale non serve a nulla.** La fusione fissa 5 orbi e danno 32, il livello degli orbi è ignorato. | Lettura del codice (righe 1838–1841). | Togliere dal pool, o far scalare la fusione col livello. |
| 5 | **Le fusioni non scalano mai.** Danno fisso (34, 45, 32) mentre gli HP dei nemici crescono in modo esponenziale: dopo la fusione la build si "congela". | Al minuto 8 HP medi nemici ~513 contro danno fisso 34–45. | Livelli per le fusioni (es. ogni level-up successivo può potenziarle) o scaling con il livello del giocatore. |
| 6 | **Clessidra sbloccata a metà run non riduce la ricarica** finché non prendi Celerità Arcana (aggiorna `baseCooldownMult` ma non `cooldownMult`). | Lettura del codice (righe 1591–1593). | Ricalcolare `cooldownMult` subito. |
| 7 | **Badge "Traguardo Fusione!" senza fusione disponibile** ogni 3 livelli, anche se non hai due magie compatibili: confonde. | Lettura del codice. | Mostrare il badge solo se `availableFusions.length > 0`. |
| 8 | **Produzione passiva persa con la scheda in background.** Il tick usa `setInterval` a 1s con incremento fisso: i browser lo rallentano a 1/min in background. | Comportamento noto dei browser. | Calcolare l'incremento dal tempo reale trascorso (`Date.now() - lastTick`). |
| 9 | **Senza connessione il gioco non parte.** Tailwind arriva da CDN: offline il layout salta e il canvas copre il pulsante "Inizia". | Riprodotto: con il CDN bloccato il click su Inizia è intercettato dal canvas. | CSS Tailwind compilato e incluso nel file (28 KB, ora cresciuto a ~31 KB con i nuovi elementi UI). |
| 10 | **Mescolamento carte level-up distorto** (`sort(() => 0.5 - Math.random())`). | Bias noto di questa tecnica. | Fisher–Yates. |

## Prestazioni — tutte le voci S/M affrontate ✅

- **Ombre canvas (`shadowBlur`) su ogni nemico, gemma e proiettile**: con 237 nemici e 918 gemme a schermo, un frame costava 39,8 ms con le ombre contro 7,9 ms senza (5× più lento). **Risolto**: ogni tipo di nemico/gemma viene pre-disegnato con il bagliore su un canvas offscreen una volta sola, poi riusato con `drawImage` (ora anche con una cache per raggio, dopo l'aggiunta dei nuovi tipi di nemico a raggio variabile).
- **Gemme che si accumulano all'infinito**: risolto con doppio sistema di merge (per cella vicina + per distanza dal giocatore).
- **Collisioni O(n²)** tra nemici, proiettili e nova: risolto con griglia spaziale (spatial hash).
- **Ordinamento di tutti i nemici a ogni sparo**: risolto con ricerca del più vicino nella griglia.
- **Nessun culling**: risolto, nemici/gemme fuori camera non vengono più disegnati.
- **HUD aggiornato ogni frame via DOM**: ora aggiornato solo quando i valori cambiano.
- **Pool di oggetti** per proiettili/particelle/testi: **non implementato** — valutato a basso guadagno rispetto al rischio, dato che gli array coinvolti sono già piccoli e limitati e il vero collo di bottiglia (`shadowBlur`) è stato risolto altrove. Resta disponibile su richiesta se servisse ancora margine prestazionale.

## Bilanciamento ed economia — risolto nella prima ondata ✅

- **L'idle rompeva la meta-progressione** (12 ore offline = molte più Essenza del costo per maxare tutto l'Altare): corretto con rendimenti decrescenti e capienza massima potenziabile.
- **Anime a fine run pesate solo per numero di kill**: ora pesate anche per tipo di nemico.
- **Inizio lento** (livello 2 dopo 25s): curva XP iniziale ammorbidita.
- **Difficoltà a gradini**: curva di spawn più graduale dopo il minuto 4.
- **Nessuna fine run**: **risolto nella seconda ondata** — ora c'è un boss finale al minuto 12 con una seconda fase (sotto il 50% HP, intensifica l'attacco con zone di pericolo multiple).
- **Boss senza meccaniche**: **risolto nella seconda ondata** — i Guardiani ora hanno tre varianti (corpo a corpo, a carica, a distanza con attacchi ad area telegrafati), non più solo "un ottagono che insegue".
- **Magnete Runico con descrizione fuorviante ("+35%" ma composto)**: corretto.

## UX e mobile — risolto nella prima ondata ✅

- Auto-pausa in background (`visibilitychange`) con recupero della produzione passiva al ritorno.
- Pointer Events: joystick/level-up funzionano anche col mouse su desktop, non solo col touch.
- Tasti 1/2/3 per le carte di level-up.
- Salvataggio con versione + pulsanti Esporta / Importa / Reset nel Santuario.
- Pulsante Pausa e contrasto numeri danno: rifiniti.

## Cosa copiare dagli altri giochi del genere (riferimento, non modificato)

- **Reroll / Skip / Banish** (Vampire Survivors): rilanciare le carte, saltare il level-up, o eliminare una scelta dal pool per tutta la run.
- **Evoluzioni con requisito di livello massimo + passivo** (Vampire Survivors): oggi la fusione chiede solo LV1 di entrambe le magie.
- **Reroll gratuiti** come opzione anti-frustrazione (Elemental Survivors).
- **Esplorazione della mappa con forzieri, negozi e casse distruttibili** (Spirit Hunters, Free Version Survivors).
- **Personaggi con arma iniziale diversa** e meta-progressione che sblocca nuove armi (HoloCure, Soulstone Survivors).
- **Obiettivi di fine gioco strutturati** e livelli di difficoltà (Soulstone Survivors).
- **Dash con ricarica** (Creature Survivors).
- **Boss con pattern leggibili**, evitando colpi "istantanei" ingiusti — **parzialmente applicato**: i nuovi Guardiani a distanza telegrafano l'attacco prima di colpire.

## Stato delle migliorie proposte

Stima di impegno: S = meno di un'ora, M = qualche ora, L = sessione lunga. Tutto resta in un singolo file HTML, senza dipendenze esterne.

### Priorità 1 — correzioni e prestazioni — ✅ **completata interamente**

1. Correggere i bug 1–10. ✅
2. CSS Tailwind compilato dentro al file + font con fallback. ✅
3. Sprite pre-renderizzati con bagliore al posto di `shadowBlur`. ✅
4. Fusione delle gemme lontane in gemme grosse + culling fuori schermo. ✅
5. Griglia spaziale per collisioni e ricerca del nemico più vicino. ✅
6. Object pooling per proiettili, particelle e testi. ⏸️ *non implementato, a basso guadagno — su richiesta*
7. Auto-pausa in background, Pointer Events, tasti 1/2/3 al level-up. ✅
8. Salvataggio con versione, Esporta / Importa / Reset nel Santuario. ✅

### Priorità 2 — bilanciamento — ✅ **completata interamente**

9. Capienza massima del Pozzo + rendimenti decrescenti. ✅
10. Anime pesate per tipo di nemico, non per kill. ✅
11. Curva XP iniziale più dolce, difficoltà più graduale. ✅
12. Fusioni con livelli + requisito "magia a livello massimo". ✅
13. Tutti i numeri di bilanciamento in un unico oggetto `BALANCE`. ✅

### Priorità 3 — contenuti e profondità — parzialmente completata

14. Reroll, Skip e Banish al level-up. ⏸️ *non implementato — rimandato in questa ondata*
15. **Nuovi nemici**: tiratore a distanza, kamikaze, splitter, élite con aura, nemico che scappa. ✅ *in parte: aggiunti Dasher (carica a scatti) e Splitter (si divide alla morte); mancano ancora kamikaze esplosivo, élite con aura ed "scappa e lascia scrigno"*
16. **Boss con pattern**: carica telegrafata, anello di proiettili, evocazione minion; boss finale con vittoria. ✅ *in parte: aggiunto boss finale al minuto 12 con seconda fase e zone di pericolo telegrafate, più varianti a carica/distanza per i Guardiani normali; manca ancora anello di proiettili, evocazione minion e una vera schermata di "vittoria" (oggi la run continua anche dopo il boss finale)*
17. Nuove magie e 4–6 nuove fusioni. ✅ *in parte: aggiunta l'Aura Glaciale (nuova magia base); nessuna nuova fusione*
18. Dash con ricarica su doppio tap / barra spaziatrice. ⏸️ *non implementato — rimandato in questa ondata*
19. Pickup a terra: pollo/cura, calamita globale, bomba, forziere normale. ⏸️ *non implementato*
20. Più traguardi e sblocchi legati ai traguardi. ⏸️ *non implementato*
21. 2–3 personaggi giocabili con magia iniziale e bonus diversi. ⏸️ *non implementato*
22. Livelli di difficoltà / "Maledizione" per più ricompense. ⏸️ *non implementato*
23. Strutture del Santuario con effetti unici invece di "più Essenza". ✅ *in parte: la 7ª struttura (Cristallo dell'Eco) resta comunque un generatore di risorse, non un effetto unico*

**Extra non previsto nel documento originale, aggiunto su richiesta:** nuova 5ª tab del Santuario, **"Albero"**, separata e aggiuntiva rispetto all'Altare (che resta invariato, griglia piatta originale). Contiene un vero albero di potenziamenti con un **Nucleo Arcano** centrale da risvegliare (100 ✨, +5% danno magico generale) e 3 rami con dipendenze radice → figlio: **Vitalità** 🩸 (Rubavita, generale: cura in base al danno inflitto → Rigenerazione Profonda, rigenerazione passiva), **Fulmini** ⚡ (Sovraccarico, +danno a Tempesta di Fulmini → Catena Prolungata, scariche extra garantite) e **Gelo** ❄️ (Gelo Profondo, +danno ad Aura Glaciale → Tempesta Glaciale, +raggio). Usa la stessa valuta (Essenza Arcana) dell'Altare; tutti i bonus sono reali e persistenti in gioco (non solo estetici). ✅

### Priorità 4 — rifinitura ("juice") — parzialmente completata

24. Musica generativa con Web Audio e suoni distinti per ogni magia/fusione. ⏸️ *non implementato*
25. Hit-stop breve sui critici e sulle uccisioni del boss, numeri critici più grandi. ✅
26. Effetti di morte dei nemici diversi per tipo, sfondo con parallasse o decorazioni sparse. ✅ *in parte: aggiunta una palette di sfondo che cambia ogni 3 minuti (puramente estetica); gli effetti di morte restano uguali per tutti i tipi*
27. Schermata statistiche di fine run (danno per magia, DPS, gemme raccolte). ✅ *danno per arma e kill per tipo, più storico delle scelte di livello e classifica locale Top 5; DPS e gemme raccolte non mostrati esplicitamente*
28. Opzioni accessibilità: riduci scossoni/flash, numeri danno on/off, dimensione HUD. ✅ *in parte: aggiunti "riduci bagliore" e scala HUD; manca ancora "riduci scossoni/flash" separato e "numeri danno on/off"*
29. PWA: manifest + service worker per installarlo sul telefono come app e giocarlo offline. ⏸️ *non implementato*

## Prossimi passi possibili

Le voci rimaste aperte più significative, in ordine di probabile impatto:

- **Reroll / Skip / Banish** al level-up — tocca direttamente la sensazione di controllo sulla build (Priorità 3, voce 14).
- **Schermata di vittoria vera e propria** dopo il boss finale, invece di continuare all'infinito (completa la voce 16).
- **Dash con ricarica** — darebbe al giocatore uno strumento attivo contro gli accerchiamenti (voce 18).
- **Nuove fusioni** per sfruttare la nuova Aura Glaciale (voce 17).
- Il resto (personaggi giocabili, livelli di difficoltà, PWA, musica generativa) resta materiale per sessioni successive più lunghe.

## Fonti

- [Vampire Survivors — la meccanica del Banish (GMDQ)](https://gmdq.substack.com/p/vampire-survivors-banish-mechanic)
- [Recensioni Army of Ruin — critiche al sistema di evoluzione (Vaporlens)](https://vaporlens.app/app/1918040/army_of_ruin.md)
- [Recensioni Spirit Hunters: Infinite Horde — boss ed esplorazione (Vaporlens)](https://vaporlens.app/app/1914580/spirit_hunters_infinite_horde.md)
- [Elemental Survivors — evoluzioni e reroll gratuiti](https://weloveeverygame.com/games/2180000-elemental-survivors.html)
- [Soulstone Survivors 1.0 — meccaniche ed endgame (FinalBoss)](https://finalboss.io/soulstone-survivors-1-0-hits-pc-consoles-why)
- [15 giochi come Vampire Survivors — HoloCure e altri (Summer Engine)](https://www.summerengine.com/blog/games-like-vampire-survivors)
- [Slash the Hordes — clone open source HTML5 con object pool (GitHub)](https://github.com/esker/vs-clone)
- [Free Version Survivors — template open source, casse distruttibili (itch.io)](https://fodi.itch.io/free-version-survivors)
- [Creature Survivors — dash (itch.io)](https://ledoc.itch.io/creature-survivors)
