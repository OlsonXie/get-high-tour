# Get High Tour – CLAUDE.md

Delad webbapp för ett kompisgäng på 6 som ska åka topptur kring **Tromsø/Lyngen 15–21 mars 2027**. Man hittar turer, bedömer läget (lavin, väder, snö) och betygsätter dem efteråt. Ägare: Oscar (designsystemdesigner, tänker i tokens och komponenter).

**Språk:** hela appen ska vara på **engelska** (beslut 2026-11-20). I dag är allt UI på svenska, så det översätts i UI/UX-omtaget (uppgift 3). Ny text skrivs på engelska redan nu. Kod, kommentarer och commit-meddelanden skrivs på engelska.

Den här filen ligger kvar i repot och är källan till sanningen för projektet. Både Claude Code och Claude i projektchatten uppdaterar den. Håll den aktuell när ett beslut ändras, och pulla före ändringar.

---

## ▶ FÖRSTA UPPGIFT FÖR CLAUDE CODE

**Ny spårstil + legend + grunden för tokens**, i `beta/index.html`. Fullständig specifikation finns i avsnitt 4.

1. Skapa `tokens/tokens.json` (DTCG-format, se avsnitt 3) med **bara spår-tokens** till att börja med: primitiver `green-track`, `red-track`, `yellow-track`, `blue-flat`, `neutral-known` och de semantiska `track.skin`, `track.descent`, `track.flat`, `track.bootpack`, `track.known` och `track.casing`. Spegla dem som CSS-variabler i `:root` och läs in dem i ett JS-objekt `T` via `getComputedStyle`. Inga hex-värden för spår i kartlagren.
2. Låt workern returnera klass per segment även för uppben: `skin` / `boot` (boot pack vid terränglutning ≥ `BOOTPACK_DEG = 35` eller över gränsen för "skidor på ryggen" i färdriktningen, med utjämning ±30 m).
3. Rita spåren: **skins up** = grön streckad linje, **descent** = röd heldragen linje, **flat** (<5° på nedfart) = tjockare blå linje, **boot pack** = gula prickar. Motlut ritas som flat. Behåll den mörka kantlinjen (casing).
4. Tona ned kända spår (GPS-heatmap + OSM-skidspår): tunn, neutral och halvgenomskinlig linje, utan rosa eller lila.
5. Ny legend centrerad längst ned på kartan, på **engelska**: `Skins up · Descent · Flat · Boot pack` (+ `Known tracks` när lagret är på), med inline-SVG som visar de riktiga linjestilarna. Den ersätter `LEGEND` i panelerna.
6. Höjdprofilen och GPX-exporten använder samma klasser och färger.
7. Testa lokalt (`python3 -m http.server 8000` → `/beta/`) på en känd tur med brant topp, t.ex. Steinskarstinden, och på en egen ritad rutt. Kontrollera desktop, iPad (820×1180) och mobil (375×812).
8. Commit i en branch och visa Oscar. Rör inte `index.html` (live).

**Klart när:** stilarna går att skilja åt även i gråskala, legenden täcker aldrig bottenarket på mobil, inga rosa eller lila spår finns kvar och boot pack inte fladdrar.

**Därefter:** uppgift 2 = full tokenisering utan visuell förändring (avsnitt 3). Uppgift 3 = UI/UX-omtag + översättning till engelska (avsnitt 1). Stäm av med Oscar mellan uppgifterna.

---

## 1. Prioritet just nu: minimera först, maximera sedan

Appen har för mycket intryck. Gör först det viktigaste riktigt bra:

1. **Befintliga turer**: hitta, se på kartan och öppna en tur.
2. **Skapa egna turer**: rita med automatiskt vägval.
3. **Bästa möjliga karta**: tydlig, lugn och lätt att läsa.

Allt annat (lavinterräng, status, betyg, GPX, bokmärken) får finnas kvar, men ska ligga ett steg bort och inte konkurrera visuellt. **Superenkel går före funktionsrik.** Mobilen först, men det ska fungera i helskärm på dator och iPad.

Arbetsordning:
1. **Spårstil och legend** (avsnitt 4) med grunden för tokens, se FÖRSTA UPPGIFT.
2. **Full tokenisering** (avsnitt 3).
3. **Stramare UI/UX + engelska**: bättre knappar, färre element och tydligare hierarki, och all text översatt till engelska. Diskutera med Oscar innan större omtag.

---

## 2. Repo, hosting och testning

- Publikt repo `OlsonXie/get-high-tour`. GitHub Pages serverar från `main`.
- `index.html` = live (v0.4, rör inte utan OK). `beta/index.html` = **här sker utvecklingen** (v0.7): https://olsonxie.github.io/get-high-tour/beta/
- Varje version är **en enda HTML-fil** (~165 kB) med CSS, JS och en web worker (inlindad i en Blob). Det finns inget byggsteg. Dela gärna upp i moduler om det hjälper, men då ska Pages fortfarande fungera utan byggsteg, eller så sätter vi upp ett enkelt Vite-flöde efter att ha frågat Oscar.
- **Testa lokalt:** `python3 -m http.server 8000` och öppna `http://localhost:8000/beta/`. Kartdata hämtas live från externa tjänster (CORS ok), så det behövs internet.
- Pusha inte kod till `main` utan Oscars OK. Jobba i en branch. (CLAUDE.md får uppdateras direkt på `main`.)

### Datakällor (live i webbläsaren)
| Vad | Källa |
|---|---|
| Höjd (3D + vägval) | Kartverket DTM `hoydedata.no/arcgis/rest/services/DTM/ImageServer/exportImage` (`format=bsq&pixelType=F32`), protokoll `kvdem://`. Reserv: AWS Terrain Tiles |
| Baskartor | Kartverket topo / `topograatone` (Svartvit, förvald), målad vinterkarta `vmal://`, Esri World Imagery, Sentinel-2 via Planetary Computer `s2://` |
| Bratthet | NVE Bratthet_2024 |
| Lavin | Varsom API v6.3.0 (region 3010 Lyngen, 3011 Tromsø) |
| Väder | Open-Meteo |
| Kända spår | OSM publika GPS-spår `gps-tile.openstreetmap.org` (`gpsh://`) + inritade skidspår `OSM_TRK` |

Cache: Cache API `ght-tiles-v{PAINTV}`.

### Kodkarta (beta/index.html)
- Routing via `location.hash`: `""` = helskärmskarta (start), `#bokmarken`, `#rita`, `#karta`, `#<rutt-id>` = detaljvy. `route()` styr.
- Vägval: `routerCore()` (A* på höjdrutnät 5–8 m/cell) körs i en worker. Ben (`legs`) har `mode: "up" | "down"`, och nedfartsben har `cls[]` per segment (`ride`, `flat`, `mot`).
- Rendering av beräknade vägar: `KCOL`, `PCOL`, `resFeatures()`, `addPathLayers()`, `LEGEND` (~rad 1545–1590).
- Spårlager: `gpsHeatTile()` (magentarosa heatmap) och lagren `known` / `osm` (lila streckat `#8e44d9`).
- Lavinterräng längs vägen: `CALCV`, `AVK`.

### Vägvalets regler (behåll)
- **Upp:** lutningen i färdriktningen hålls under ca 16–20° (ger sicksack). Över ca 26° i färdriktningen räknas det som att gå med skidorna på ryggen. Terräng över 30° straffas hårt.
- **Ned:** följer fallinjen. Partier under 5° och motlut straffas. Terräng över 35° undviks (över 45° nästan spärrat).
- Upp och ned får gå olika vägar. Vändpunkten är högsta punkten men kan ändras.

---

## 3. Designtokens (att införa)

Det finns inga tokens i dag, bara ett fåtal CSS-variabler (`--ink`, `--accent` …) och många hårdkodade värden: ~12 olika fontstorlekar (11–40 px, även 11.5/12.5/13.5), ~10 olika radier och hex-färger direkt i MapLibre-`paint`. **Skapa ett tokensystem utifrån det som finns nu**, så redigerar Oscar vidare.

### Struktur: primitiver → semantiska tokens
- **Primitiver** = rå skala, utan betydelse: `--color-red-500`, `--color-neutral-900`, `--space-4`, `--radius-md`, `--font-size-300`.
- **Semantiska** = betydelse, pekar alltid på primitiver: `--color-bg`, `--color-text`, `--color-accent`, `--color-btn-primary-bg`, `--track-skin`, `--track-descent`, `--track-bootpack`, `--track-known`, `--danger-level-3` …
- Komponenter använder **bara semantiska tokens**, aldrig primitiver eller hex.
- **Källa:** `tokens/tokens.json` i W3C DTCG-format (`$value`, `$type`, alias `{color.red.500}`), med grupperna `primitive` och `semantic`. Den genererar (eller speglas manuellt i) en `:root { … }`-block. Formatet ska gå att importera som **Figma Variables** senare (primitiver som en samling, semantiska som en annan, med alias). Oscar vill få in dem i Figma för överblick.
- **MapLibre kan inte läsa CSS-variabler.** Läs dem en gång med `getComputedStyle(document.documentElement).getPropertyValue(...)` till ett JS-objekt `T` och använd `T.trackDescent` osv. i `paint`. Inga hex-värden i kartlagren.

### Startvärden (från nuvarande kod plus Oscars spårskiss)
| Primitiv | Värde | Används i dag till |
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
| blue-500 | `#2f6fdc` | spår upp (ersätts) |
| red-500 | `#e0342a` | spår ned (ersätts) |
| **green-track** | `#6DAE8A` | **skin track upp (ny)** |
| **red-track** | `#E0504A` | **nedfart (ny)** |
| **yellow-track** | `#E8B24E` | **boot pack (ny)** |
| **blue-flat** | `#3B7DD8` | **flacka partier på nedfart, tjockare linje (ny)** |
| sand-100 | `#EDDFD7` | bakgrund i Oscars skiss (kandidat för legend/ytor) |
| ink-warm | `#2D2E2B` | text i Oscars skiss |
| Lavin 1–4 | `#56b56f` `#f2c200` `#f07f1a` `#e0261b` | Varsoms faregrader (behåll, standardfärger) |
| NVE bratthet | `#f7e04b` `#f39a25` `#e3342a` `#9b3fb5` | 27–29°, 30–34°, 35–39°, 40°+ |
| Status | g `#2f8a57`, y `#a07400`, o `#c4561c`, r `#b3261e` (+ ljusa bakgrunder) | grön/gul/röd bedömning |

- **Typsnitt:** rubriker i Instrument Serif och brödtext/knappar i Montserrat (v0.7). Oscar vill se över typsnitt och färger, så lägg dem som tokens (`--font-heading`, `--font-body`) så att de är lätta att byta. Skissens rubriktypsnitt är en kraftig grotesk (i stil med Archivo Black). Föreslå gärna alternativ.
- **Typskala:** reducera till ~6 steg (t.ex. 11/13/15/18/24/32). **Radier:** ~4 steg (sm 8, md 12, lg 16, xl 26, pill 999). **Spacing:** 4-px-skala.
- Gör tokeniseringen **utan visuell förändring först** (samma utseende, bara tokens), och ändra sedan utseendet via tokens.

---

## 4. Spårstil och legend (första konkreta uppgift)

Oscars skiss: tre spårtyper med tydliga, olika linjestilar, så att typen syns även utan färg.

| Typ | Stil | Färg | När |
|---|---|---|---|
| **Skins up** | Streckad linje (rundade streck, t.ex. `line-dasharray [2.2, 1.6]`, `line-cap: round`) | `--track-skin` `#6DAE8A` | All uppstigning med stighudar |
| **Descent** | **En** heldragen linje | `--track-descent` `#E0504A` | All nedfart |
| **Flat** | Heldragen men **tjockare** (~1,6× descent) | `--track-flat` `#3B7DD8` | Nedfart under 5° och motlut, där brädan kan behöva gå (snowboard) |
| **Boot pack** | Prickar (runda punkter: dasharray `[0, 2]` med `line-cap: round`, eller symbol-cirklar längs linjen) | `--track-bootpack` `#E8B24E` | Uppför där man tar av skidorna och bär dem på ryggen |

### Regler
- **Boot pack-tröskel (beslutad 35°):** ett uppförssegment klassas som boot pack när **terrängens lutning ≥ 35°** *eller* när lutningen i färdriktningen överstiger vägvalets befintliga gräns för "skidor på ryggen" (~26°). Lägg tröskeln i en konstant `BOOTPACK_DEG = 35` så att den går att ändra.
- **Utjämning:** boot pack-sträckor kortare än ~30 m slås ihop med omgivningen, och luckor under ~30 m mellan två boot pack-sträckor fylls. Det ska inte "fladdra" mellan streck och prickar.
- Workern ska returnera klass per segment även för uppben (`up`/`boot`), som `cls[]` redan gör för nedben.
- **Nedfarten är en röd linje**, där flacka partier och motlut markeras som en **tjockare blå** sträcka (beslutat 2026-11-20, kan ändras senare). Dagens gula och orange markeringar försvinner, eftersom gult nu betyder boot pack. Board-bedömning, meter flackt och höjdprofil behålls. Alternativet som diskuterades (skins up blått och flackt tjockare grönt) hålls lätt att byta till via tokens.
- Behåll den mörka, halvgenomskinliga kantlinjen (casing) under spåren så de syns på alla baskartor. Färdriktningspilarna på uppstigningen kan tas bort, eftersom den streckade stilen redan säger "upp" (fråga om du är osäker).
- Höjdprofilen och GPX-exporten använder samma tre klasser och färger.
- Lavinterrängens rand under rutten (röd/orange/lila) krockar med nya färger. Se över den, t.ex. med en tunnare halo eller ett separat mönster, men lös det efter att spårstilen är klar.

### Kända spår (dagens rosa lager)
- Ska bli **mycket mindre synliga** och få en **neutral färg**: tunn linje (~1,5–2 px), varm grå eller vit beroende på baskarta, opacitet ~0,35–0,5, utan streck. Token `--track-known`.
- Gäller både GPS-heatmapen (`gpsHeatTile`, som i dag färgar magentarosa) och de inritade skidspåren (`known`/`osm`, lila streckat). Båda blir samma diskreta stil.
- De ska fungera som bakgrundsinformation och aldrig se ut som en beräknad rutt.
- Uppdatera texter som säger "Rosa linjer är GPS-spår …" (på engelska).

### Legend
- Liten, **centrerad längst ned** på kartan (startkarta, ritläge och detaljvy).
- Fyra poster med **miniatyr av riktig linjestil** (inline-SVG: streckad grön, heldragen röd, tjock blå, prickad gul) och etiketterna på engelska: **Skins up · Descent · Flat · Boot pack**. Lägg till **Known tracks** som en nedtonad post när det lagret är på.
- Stil: kompakt pill med `--color-surface` och lätt skugga. På mobil placeras den ovanför bottenarket och får aldrig täcka det. Gärna ihopfällbar till en liten "i"-knapp.
- Ersätter dagens textlegend (`LEGEND`, med Upp/Ned/Flackt/Motlut) inne i panelerna. NVE-legenden för bratthet (`#legend`) är separat och visas bara när det lagret är på. Placera den så att de två inte krockar.

---

## 5. Övriga beslut som gäller

- **Kärnflöde:** sök tur → se rutten → "Check status for route" (lavin, väder och snö matchat mot ruttens aspekter/höjder) → åk → betygsätt (snö och åkning, uppstigning och utsikt, fritext och bilder).
- **En i gruppen åker snowboard:** långa flacka partier och traverser på nedfarten är ett minus och flaggas (board-bedömning: bra < 150 m flackt, ok < 500 m, annars jobbig).
- **Statuskoll** görs av Claude på begäran och skrivs in i `SAVED_STATUS`. Det är beslutsstöd, inte ett beslut.
- **12 förberedda turer:** Kvaløya: Rødtinden, Finnlandsfjellet, Kjølen, Buren, Steinskarstinden, Lille Blåmannen, Skittentinden. Fastlandet: Tromsdalstinden, Fagerfjellet, Stormheimfjellet. Lyngen: Rørnestinden, Fastdalstinden.
- **Designriktning v0.7:** accent `#D60000`, svarta primärknappar, vita sekundärknappar med tunn grå kontur, förvald baskarta Svartvit som roterar långsamt i 3D på start, kartlager-modul via kartminiatyr. Allt detta kan ändras i UI/UX-omtaget.
- Allt sparas i dag i `localStorage` (bokmärken `favs`, egna rutter, betyg och GPX). **Delad data** (t.ex. Supabase, ingen inloggning, bara en delad länk) kommer efter att kartan och UI:t känns rätt.

## 6. Senare (inte nu)
- Delad data för bokmärken, betyg, statuskollar och egna rutter
- Riktiga GPX-spår och verifierade startpunkter/parkeringar
- PWA/offline för bokmärkta turer
- Serverbakade kartrutor (PMTiles)
- Tokens till Figma Variables
