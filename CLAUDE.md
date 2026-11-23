# Get High Tour – CLAUDE.md

Delad webbapp för ett kompisgäng på 6 som ska åka topptur kring **Tromsø/Lyngen 15–21 mars 2027**. Man hittar turer, bedömer läget (lavin, väder, snö) och betygsätter dem efteråt. Ägare: Oscar (designsystemdesigner, tänker i tokens och komponenter).

**Språk:** hela appen är på **engelska** (beslut 2026-11-20, genomfört 2026-11-23). Kod, kommentarer och commit-meddelanden skrivs på engelska. Ortnamn behålls som de är (Rødtinden osv.). Äldre kodkommentarer är fortfarande delvis på svenska och översätts när koden rörs.

Den här filen ligger kvar i repot och är källan till sanningen för projektet. Både Claude Code och Claude i projektchatten uppdaterar den. Håll den aktuell när ett beslut ändras, och pulla före ändringar.

---

## ▶ AKTUELL UPPGIFT FÖR CLAUDE CODE (2026-11-23)

> **Status 2026-11-23: del 1–7 är klara** i branchen `claude/adoring-ptolemy-0hg6vv` (inte mergad till `main`, väntar på Oscars test). Beslut som togs under arbetet står under "Gjort" nedan och i avsnitt 2–5. Nästa steg: Oscar testar på telefon, sedan merge och full tokenisering (avsnitt 3).
>
> **Gjort:**
> - **Del 1:** felet var att uppförssökningen saknade vändkostnad, så sicksacken kollapsade till 1-rutas-växling (rak linje i fallinjen) som `smoothLine()` sedan rätade ut. Den blå linjen var `main`:s gamla uppförsfärg; dessutom räknades uppför efter högsta klickade punkten som nedfart (rättat). Se "Vägvalets regler" i avsnitt 2. Kortare ben (beslut 2026-11-23) och flackt = upplysning, inte krav.
> - **Del 2:** skins up = dubbelstreck, se avsnitt 4.
> - **Del 3:** Vinterkarta förvald, Svartvit och Sentinel-2 borta (kod också), Heatmap ny baskarta (violett täthetsskala `--heat-low/high`, krockar inte med ruttfärgerna). Vinterkartan målas i 3× (768 px) på skärmar med devicePixelRatio > 2.
> - **Del 4:** lavinterräng = skrafferad yta ±30 m kring spåret, se avsnitt 4.
> - **Del 5:** startvyn är bara kartan. Sök fick en egen ikonknapp (överst), eftersom att hitta turer är prioritet 1; tom sökning listar alla turer. Egna rutter öppnas fortfarande i panelen (de har ingen detaljvy). Lagervyn är `#layers` och lägger sig över vyn man kom från. På mobil göms lagerknappen medan en panel är öppen.
> - **Del 6:** Archivo Black via `--font-heading`, rubrikerna något mindre än med Instrument Serif.
> - **Del 7:** allt UI på engelska, datum en-GB, väderstreck N/NE/E…, Varsom-texter på engelska (API-språk 2). Hash-routes på engelska, svenska omdirigerar.

Allt görs i `beta/index.html`. Gör delarna i ordning och **stäm av med Oscar efter del 1 och efter del 4**. Ge en raw.githack-länk med commit-SHA efter varje push (se avsnitt 2).

**Utgångsläge:** uppgiften från 2026-11-20 (spårstil, legend och spår-tokens) ligger i branchen `claude/adoring-ptolemy-0hg6vv` (commit `61c1832`) och är **inte mergad till `main`**. Skapa en ny branch från den och bygg vidare där. `main` har fortfarande den gamla spårstilen.

### Del 1. Fixa vägvalet (viktigast)
**Problem:** en egen ritad uppstigning går rakt upp längs en brant rygg utan sicksack och ritas som blå heldragen linje, alltså "Flat". Både vägen och klassningen är fel.
1. **Diagnostisera först och skriv kort vad du hittade.**
   - Varför ritas ett uppben blått? Möjliga orsaker: benet har fått `mode: "down"` så att uppför blir `mot`, gamla `KCOL`-färger används fortfarande eller mappningen i `resFeatures()` är fel.
   - Varför bryter vägen mot lutningsreglerna? Möjliga orsaker: för grov eller för utjämnad höjdgrid, efterutjämning eller förenkling som rätar ut sicksacken, lutning räknad på fel skala (cellstorlek eller meter jämfört med grader), eller för låg kostnad för brant terräng.
2. **Uppför:** undvik terräng ≥ 30° kraftigt och välj hellre en längre väg. Håll lutningen i färdriktningen under ca 16–20°, så att det blir sicksack. Om ≥ 30° inte går att undvika klassas sträckan som **boot pack**. `BOOTPACK_DEG = 30` (ändrat från 35). Utjämning ±30 m, så att det inte fladdrar.
3. **Nedför:** följ fallinjen och undvik > 35° (> 45° ska vara nästan spärrat). **Flat** gäller bara < 5° och motlut på ett **nedben**. Ett uppben får aldrig klassen flat.
4. Logga max-, medel- och minlutning i terrängen per klass (`skin`/`boot`/`descent`/`flat`) i konsolen när en rutt räknas fram.
5. Testa på Steinskarstinden och på en egen rutt uppför en brant rygg. Visa före och efter.

### Del 2. Spårstil
- **Skins up:** två parallella streckade linjer bredvid varandra, som två skidspår. Använd längre, avlånga streck (t.ex. `[3, 1.5]`) och `line-offset` ±~2 px. Strecken ska ligga omlott (förskjutna mellan linjerna). Färg `--track-skin`.
- **Boot pack:** gula prickar (från 30°). **Descent:** röd heldragen linje. **Flat:** tjockare blå linje, bara på nedfart.
- Miniatyren för skins up i legenden ska visa dubbelstrecket.

### Del 3. Kartlager
- **Förvald baskarta = Vinterkarta** (`vmal://`).
- **Ta bort** Svartvit (`topograatone`) och Satellit vinter (Sentinel-2), och ta bort koden om det går rent. Kvar blir Vinterkarta, Topo, Satellit (Esri) och Heatmap (ny).
- Kontrollera att vinterkartan är skarp på högupplösta skärmar (devicePixelRatio).
- **Known tracks blir baskartan "Heatmap"** och är inte längre ett reglage ovanpå alla kartor. Heatmap = nedtonad vinterkarta plus GPS-spåren och OSM-skidspåren som en tydlig täthets-heatmap, i en färg som inte krockar med ruttfärgerna. Ta bort reglaget "Known tracks" och den diskreta known-track-stilen på övriga kartor. Legenden visar heatmap-skalan bara när Heatmap är vald.
- Bratthet (NVE) är kvar som reglage.

### Del 4. Lavinterräng
- Visa lavinterräng längs rutten som en **gul/orange halvgenomskinlig yta med svart diagonal skraffering** (fill-pattern, opacitet ~0,35–0,5). Den ersätter den färgade randen under spåret.
- Förslag: startzon orange och övrig lavinterräng gul. Spåren ligger ovanpå.
- Uppdatera legenden och listan i detaljvyn.

**→ Stäm av med Oscar innan del 5.**

### Del 5. Startvy och navigering
- **Ta bort startpanelen** (GET HIGH TOUR, sök, Bokmärken, Rita egen rutt). Startvyn ska vara bara kartan.
- **Runda ikonknappar vertikalt på högra sidan:** Bookmarks och Draw route, med tooltip och aria-label. Sök läggs i Bookmarks-vyn eller som egen ikonknapp. Föreslå vilket.
- **Klick på en rutt** (etikett eller linje) ska öppna **detaljvyn direkt**, utan förhandsvisning. Det gäller även ej bokmärkta turer.
- **Kartlagerknappen** ska ligga **nere till höger** och vara större (~64–72 px). Den öppnar en **helskärmsvy** med samma layout som detaljvyn, inte en panel eller ett bottenark. Tillbaka/stäng går till kartan.
- Legenden får inte krocka med lagerknappen.

### Del 6. Typsnitt
- Testa **Archivo Black** för rubriker via `--font-heading` (Google Fonts). Brödtext och knappar är fortfarande Montserrat.

### Del 7. Engelska
- **All** UI-text ska vara på engelska: knappar, etiketter, legender, tomma lägen, fel, tooltips, statuskoll, datumformat och turbeskrivningar.
- Hash-routes byts till `#bookmarks`, `#draw`, `#map` och `#<route-id>`. De gamla svenska (`#bokmarken`, `#rita`, `#karta`) omdirigerar.

### Klart när
- Ett uppben aldrig visas som flat, och boot pack syns bara där terrängen är ≥ 30° och inte gick att undvika
- Egna rutter uppför branta ryggar sicksackar eller väljer en annan väg
- Vinterkartan är förvald, Svartvit och Satellit vinter är borta och Heatmap finns
- Lavinterrängen syns som skrafferad gul/orange yta
- Startvyn har bara kartan, högerknapparna och lagerknappen nere till höger, och lagervalet är en helskärmsvy
- Inga svenska UI-strängar finns kvar
- Allt är testat på desktop, iPad (820×1180) och mobil (375×812)
- Den här filen är uppdaterad om något ändrats under arbetet

**Därefter:** full tokenisering (avsnitt 3), om den inte redan är gjord.

---

## 1. Prioritet: minimera först, maximera sedan

Gör det viktigaste riktigt bra:
1. **Befintliga turer**: hitta, se på kartan och öppna en tur (klick ger detaljvyn direkt).
2. **Skapa egna turer**: rita med automatiskt vägval som faktiskt följer terrängen.
3. **Bästa möjliga karta**: vinterkartan, tydlig, lugn och lätt att läsa.

Allt annat (lavinterräng, status, betyg, GPX, bokmärken) finns kvar men ligger ett steg bort. **Superenkel går före funktionsrik.** Mobilen först, men det ska fungera i helskärm på dator och iPad.

---

## 2. Repo, hosting och testning

- Publikt repo `OlsonXie/get-high-tour`. GitHub Pages serverar från `main`.
- `index.html` = live (v0.4, rör inte utan OK). `beta/index.html` = **här sker utvecklingen**: https://olsonxie.github.io/get-high-tour/beta/
- Varje version är **en enda HTML-fil** med CSS, JS och en web worker (inlindad i en Blob). Det finns inget byggsteg. Dela gärna upp i moduler, men Pages ska fortfarande fungera utan byggsteg. Alternativt sätter vi upp Vite efter att ha frågat Oscar.
- **Testa lokalt:** `python3 -m http.server 8000` och öppna `http://localhost:8000/beta/`. Kartdata hämtas live (CORS ok), så det behövs internet.
- Pusha inte kod till `main` utan Oscars OK. Jobba i en branch. (CLAUDE.md får uppdateras direkt på `main`.)
- **Molnsessioner (claude.ai/code):** Oscar kör främst Claude Code i molnet och ser inte `localhost`. Efter varje push ska du **alltid ge en förhandsgranskningslänk** via commit-SHA: `https://raw.githack.com/OlsonXie/get-high-tour/<commit-sha>/beta/index.html` (en mellansida visas först, klicka "Open the page"). Använd SHA, inte branchnamn, eftersom branchnamn med `/` inte fungerar där. Kartdata hämtas från Oscars webbläsare, så det fungerar även om molnmiljöns nätverk är begränsat.

### Datakällor (live i webbläsaren)
| Vad | Källa |
|---|---|
| Höjd (3D + vägval) | Kartverket DTM `hoydedata.no/arcgis/rest/services/DTM/ImageServer/exportImage` (`format=bsq&pixelType=F32`), protokoll `kvdem://`. Reserv: AWS Terrain Tiles |
| Baskartor | **Vinterkarta `vmal://` (förvald)**, Kartverket topo, Esri World Imagery, Heatmap (vinterkarta + known tracks). Svartvit och Sentinel-2 tas bort 2026-11-23 |
| Bratthet | NVE Bratthet_2024 |
| Lavin | Varsom API v6.3.0 (region 3010 Lyngen, 3011 Tromsø), språk 2 = engelska |
| Väder | Open-Meteo |
| Known tracks | OSM publika GPS-spår `gps-tile.openstreetmap.org` (`gpsh://`) och inritade skidspår `OSM_TRK`. Datan är tunn i Nordnorge. Strava-heatmapen är inte öppet tillgänglig (bara länk per tur) |

Cache: Cache API `ght-tiles-v{PAINTV}`.

### Kodkarta (beta/index.html)
- Routing via `location.hash`: `""`/`#map` = helskärmskarta (start), `#search`, `#bookmarks`, `#draw`, `#<route-id>` = detaljvy, `#layers` = lagervyn ovanpå vyn man kom från. Gamla svenska hash (`#bokmarken`, `#rita`, `#karta`) omdirigerar (`HASH_ALIAS`). `route()` styr.
- Översiktskartan: `renderPanel()` visar panel bara för sök (`renderSearch`), bokmärken (`renderBm`), ritläge och egna rutter; annars `setPanel(false)`. `selectOv(id)` öppnar detaljvyn för turer. Ikonknappar `#nSearch/#nBm/#nDraw`, lagerknapp `.lyrthumb`.
- Vägval: `routerCore()` körs i en worker: `searchCell()` (A* över rutor, 5–8 m/cell) och för uppben `corridor()` + `searchUp()` (A* över ruta × färdriktning). Ben (`legs`) har `mode: "up" | "down"` och `cls[]` per segment från `hydrate()` (`skin`/`boot` uppför, `ride`/`flat`/`mot` nedför; `rkey()` ger renderklass). `CALCV` höjs när vägvalet ändras, så sparade rutter räknas om.
- Rendering av beräknade vägar: `resFeatures()`, `addPathLayers()` (ett lager per linjestil i `TRACK_LAYERS`), `setPaths()` (spår + separat källa för etiketter, så att ett glyph-fel inte släcker spåren). Tokens via objektet `T`.
- Heatmap-baskartan: `gpsHeatTile()` (täthetsfärg), lagren `gps`, `known`, `osm`, styrs i `applyLyr()` tillsammans med nedtoningen av vinterkartan.
- Lavinterräng längs vägen: `aval()` i workern, `avSegs()`, `avFeatures()` + `traceAreas()` (ytor), `addAvalLayer()` (fill-pattern), `avHTML()` (listan).

### Vägvalets regler
- **Upp:** undvik terräng ≥ 30° kraftigt. Lutningen i färdriktningen hålls under ca 16–20° (sicksack). Det som ändå är ≥ 30°, eller över gränsen för "skidor på ryggen" (~26° i färdriktningen), är **boot pack** (`BOOTPACK_DEG = 30`, utjämning ±30 m).
  - Uppben räknas i två steg. Först en **mittlinje** med A* över rutor (där är sicksack gratis, så linjen går där en sicksack skulle gå). Sedan A* över (ruta, färdriktning i 8 sektorer) i en **korridor ±150 m** runt mittlinjen (`CORR`), där en **kick turn kostar ~40 s** (`KICK`) och ben som går längre än 40 m från mittlinjen kostar mer (`LAT0`, `LATK`). Det ger sicksack med ben på ungefär 70–250 m (beslut 2026-11-23: hellre lite kortare ben). Utan vändkostnaden kollapsar sicksacken till en 1-rutas-växling som på kartan blir en rak linje i fallinjen (felet 2026-11-23). Riktningen spelar bara roll i skin-terräng (16° till boot), annars är vändning gratis.
  - I terräng ≥ `BOOTPACK_DEG` räknas boot pack: rak linje, ca 290 höjdm/h, gånger 6/14/30/60 för 30/35/40/45°+.
- **Ned:** följer fallinjen. Partier under 5° och motlut klassas som flat (bara på nedben). Terräng över 35° undviks (gånger 8/25/150/400 för 35/40/45/50°+).
- **Flackt är upplysning, inte ett krav** (beslut 2026-11-23). Vägvalet behöver inte undvika flacka partier till varje pris, men de ska **pekas ut**: var de är, hur långa och hur flacka (medellutning, motlut visas som uphill). Flacka partier ≥ 60 m (`FLAT_LABEL_M`) får en etikett på kartan, t.ex. `220 m flat · 2°`, och listas i panelen med längd, lutning och höjder.
- Upp och ned får gå olika vägar. Vändpunkten är högsta punkten men kan ändras. Efter högsta punkten räknas en sträcka som stiger mer än 20 m också som upp.
- Varje framräknad rutt loggar terrängens lutning (max/medel/min) per klass i konsolen (`[route]`).

---

## 3. Designtokens

Tokensystem med **primitiver → semantiska tokens**. Komponenter använder bara semantiska tokens, aldrig primitiver eller hex.
- **Källa:** `tokens/tokens.json` i W3C DTCG-format (`$value`, `$type`, alias `{color.red.500}`), med grupperna `primitive` och `semantic`. Den speglas i `:root { … }`. Formatet ska gå att importera som Figma Variables (primitiver och semantiska tokens som två samlingar med alias).
- **MapLibre kan inte läsa CSS-variabler.** Läs dem med `getComputedStyle` till JS-objektet `T` och använd dem i `paint`. Inga hex-värden i kartlagren.
- Gör full tokenisering **utan visuell förändring först**, och ändra sedan utseendet via tokens.

### Startvärden
| Primitiv | Värde | Används till |
|---|---|---|
| neutral-950 | `#11161d` | text, kantlinjer på kartan |
| neutral-900 | `#14181d` | primärknapp (svart) |
| neutral-500 | `#6a7380` | sekundär text |
| neutral-300 | `#c9ced6` | kontur på sekundärknapp |
| neutral-200 | `#e2e5ea` | linjer |
| neutral-100 | `#eef0f3` | bakgrund |
| neutral-50 | `#f7f8fa` | yta |
| white | `#ffffff` | kort |
| red-600 | `#D60000` | accent |
| green-track | `#6DAE8A` | skins up |
| red-track | `#E0504A` | descent |
| yellow-track | `#E8B24E` | boot pack |
| blue-flat | `#3B7DD8` | flat på nedfart (tjockare linje) |
| sand-100 | `#EDDFD7` | kandidat för legend/ytor |
| ink-warm | `#2D2E2B` | kandidat för text |
| Lavin 1–4 | `#56b56f` `#f2c200` `#f07f1a` `#e0261b` | Varsoms faregrader (behåll) |
| NVE bratthet | `#f7e04b` `#f39a25` `#e3342a` `#9b3fb5` | 27–29°, 30–34°, 35–39°, 40°+ |
| Status | g `#2f8a57`, y `#a07400`, o `#c4561c`, r `#b3261e` (+ ljusa bakgrunder) | grön/gul/röd bedömning |

- **Typsnitt:** `--font-heading` = **Archivo Black** (test 2026-11-23, tidigare Instrument Serif). `--font-body` = Montserrat. Båda finns som `font.heading/body` i tokens.json.
- **Redan tokeniserat (2026-11-23):** spår (`track.*`, inkl. `track.halo`), heatmap (`heat.low/high`, violet-300/700), lavinterräng (`aval.start/other/hatch`, avalanche-3/2 + neutral-950) och typsnitt. Resten av UI:t är inte tokeniserat än.
- **Typskala:** ~6 steg (t.ex. 11/13/15/18/24/32). **Radier:** sm 8, md 12, lg 16, xl 26, pill 999. **Spacing:** 4-px-skala.

---

## 4. Spårstil och kartspråk

| Typ | Stil | Token | När |
|---|---|---|---|
| **Skins up** | Två parallella streckade linjer bredvid varandra, som två skidspår: bredd 3 px, `line-offset` ±2,9 px, streck `[3, 1.5]` med `line-cap: butt`. Strecken ligger omlott: MapLibre saknar dash-offset, så den andra linjen har mönstret `[0.75, 1.5, 2.25, 0]` (ett halvt varv förskjutet). Kantlinjen är 10,5 px under skins. Allt ligger i `TRACK_LAYERS` | `--track-skin` | Uppstigning med stighudar |
| **Boot pack** | Gula prickar | `--track-bootpack` | Uppför där terrängen är ≥ 30° och inte gick att undvika |
| **Descent** | En röd heldragen linje | `--track-descent` | Nedfart |
| **Flat** | Heldragen, tjockare (~1,6×) blå linje, plus etikett med längd och lutning när partiet är ≥ 60 m | `--track-flat` | Bara nedfart: < 5° och motlut (snowboard) |

- Behåll den mörka, halvgenomskinliga kantlinjen (casing) under spåren.
- Höjdprofilen och GPX-exporten använder samma klasser och färger.
- **Lavinterräng:** yta ±30 m kring spåret (`AV_BAND_M`) med `fill-pattern`: orange (startzon) eller gul (under startzon, terrängfälla) med svart diagonal skraffering, opacitet 0,45 (0,55 om sträckan träffar dagens prognos, 0,25 om den inte gör det). Alla sträckor ritas på ett gemensamt rutnät så att varje punkt bara målas en gång (startzon vinner), och konturen spåras till rena polygoner. Spåren ligger ovanpå. Listan i detaljvyn använder samma skrafferade rutor.
- **Known tracks:** visas bara i baskartan Heatmap, som en täthets-heatmap.
- **Legend:** kompakt, längst ned på kartan, på engelska: `Skins up · Boot pack · Descent · Flat`, plus `Start zone · Avalanche terrain` i detaljvyn när lavinterrängen visas och heatmap-skalan när Heatmap är vald. Den centreras i ytan till vänster om lagerknappen (krockar aldrig), och på mobil ligger den ovanför en öppen panel. NVE-legenden visas bara när Bratthet är på.

---

## 5. UI och navigering (från 2026-11-23)

- **Start** = bara kartan i helskärm, utan panel eller kort.
- **Högra sidan:** runda ikonknappar Search, Bookmarks och Draw route (tooltip och aria-label), under dem kompass, 3D och "visa allt".
- **Nere till höger:** kartlagerknapp 68 px (båda kartorna) som öppnar lagervalet i **helskärm** (`#layers`), med samma layout som detaljvyn: helsida på mobil/iPad, högerark 460 px över kartan på dator.
- **Klick på en rutt** öppnar detaljvyn direkt.
- Accent `#D60000`, svarta primärknappar, vita sekundärknappar med tunn grå kontur (från v0.7, kan ändras).

---

## 6. Övriga beslut som gäller

- **Kärnflöde:** sök tur → se rutten → "Check status for route" (lavin, väder och snö matchat mot ruttens aspekter/höjder) → åk → betygsätt (snö och åkning, uppstigning och utsikt, fritext och bilder).
- **En i gruppen åker snowboard:** långa flacka partier och traverser på nedfarten flaggas (board-bedömning: bra < 150 m flackt, ok < 500 m, annars jobbig). De pekas ut med längd och lutning men är inget krav för vägvalet.
- **Statuskoll** görs av Claude på begäran och skrivs in i `SAVED_STATUS`. Det är beslutsstöd, inte ett beslut.
- **12 förberedda turer:** Kvaløya: Rødtinden, Finnlandsfjellet, Kjølen, Buren, Steinskarstinden, Lille Blåmannen, Skittentinden. Fastlandet: Tromsdalstinden, Fagerfjellet, Stormheimfjellet. Lyngen: Rørnestinden, Fastdalstinden.
- Allt sparas i dag i `localStorage` (bokmärken `favs`, egna rutter, betyg och GPX). **Delad data** (t.ex. Supabase, ingen inloggning, bara en delad länk) kommer efter att kartan och UI:t känns rätt.

## 7. Senare (inte nu)
- Delad data för bokmärken, betyg, statuskollar och egna rutter
- Riktiga GPX-spår och verifierade startpunkter/parkeringar
- PWA/offline för bokmärkta turer
- Serverbakade kartrutor (PMTiles)
- Tokens till Figma Variables
