# Hedgehog Fit — TRMNL Plugin (Design)

**Datum:** 2026-06-24
**Status:** Entwurf zur Freigabe

## Konzept

Ein privates TRMNL-Plugin, das einen 80er-Jahre-Fitnessvideo-Igel (Schweißband,
Stulpen, „and one, and two!") zeigt, der den Betrachter augenzwinkernd zu kleinen
Fitnessübungen motiviert. Bei jedem Refresh wird pseudo-zufällig eine von sechs
Posen samt passendem, frech-witzigem Spruch ausgewählt.

Voll statisch — keine externe API, keine Datenquelle. Strategy `static`: der
gesamte Inhalt (Posen, Sprüche, Anrede-Pools) liegt als JSON im Plugin selbst
(`static_data`). Statische Plugins unterstützen kein `transform_js` — die Daten
werden direkt als JSON gepflegt.

**Scope dieses Specs:** Nur **TRMNL OG (800×480)**, nur die **Voll-Bildanzeige**
(`full.liquid`). Die übrigen Größen (half/quadrant) sind bewusst ausgeklammert.

## Lokale Entwicklung

Entwicklung mit dem [`trmnlp`](https://github.com/usetrmnl/trmnlp)-Gem für eine
Live-Preview. Der Preview-Server (`trmnlp serve`, Standard `localhost:4567`)
rendert das Plugin in allen Größen und lädt bei Dateiänderung neu.

## Projektstruktur

```
hedgehog-fit/
├─ .trmnlp.yml              # Preview-Server-Konfiguration
├─ src/
│  ├─ settings.yml          # Plugin-Metadaten + Custom Field "anrede"
│  ├─ full.liquid           # Voll-Bildanzeige (Layout A — Split)
│  └─ assets/
│     ├─ igel-1.png … igel-6.png
├─ docs/superpowers/specs/  # dieses Spec
└─ README.md
```

Hinweis: TRMNL erwartet je Größe eine eigene `*.liquid`-Datei
(`full.liquid`, `half_horizontal.liquid`, `half_vertical.liquid`,
`quadrant.liquid`). In diesem Scope legen wir nur `full.liquid` an; ein minimaler
Fallback für die anderen Größen wird beim Implementieren entschieden (entweder
schlichte Stubs oder vorerst weglassen, falls `trmnlp` das toleriert).

## Layout A — Split (Voll-Bildanzeige, 800×480)

```
┌──────────────────────────────────────────────┐
│                            │                   │
│  Aufstehn — der Stuhl      │                   │
│  gewinnt sonst!            │     [ Igel-Bild ] │
│  (title, fett)             │   col--span-5     │
│  col--span-7               │                   │
│  PLANKE                    │                   │
│  (label)                   │                   │
│                            │                   │
├────────────────────────────┴───────────────────┤
│ HEDGEHOG FIT                                    │  ← native title_bar
└──────────────────────────────────────────────┘
```

Aufbau strikt mit Framework-Klassen (keine inline-Styles, keine `<style>`-Blöcke
— TRMNL-Hard-Rule):

```
<div class="layout">
  <div class="grid">
    <div class="col--span-7"> … Spruch + Label … </div>
    <div class="col--span-5"> … Igel-Bild … </div>
  </div>
</div>
<div class="title_bar"> … </div>
```

- Linke Spalte (`col--span-7`, ~58 %): Spruch als großes `title` (z. B.
  `title--large`, ggf. `data-fit-value` gegen Überlauf); darunter ein
  `label`/`label--small` mit dem Übungs-Namen.
- Rechte Spalte (`col--span-5`, ~42 %): das Igel-Bild als
  `<img class="image image-dither" …>`, vertikal zentriert.
- Unten: native `title_bar` mit `instance_name` als Titel.
- Content-Höhe der Voll-Größe ist ~320px — Spruch-Länge dahingehend prüfen.

## Content-Modell

Sechs Posen, jede mit Bild und einem eigenen Spruch-Array (pose-spezifisch).

| # | Datei      | Pose                              | Übungs-Label       |
|---|------------|-----------------------------------|--------------------|
| 1 | igel-1.png | liegende Planke (Unterarmstütz)   | PLANKE             |
| 2 | igel-2.png | Liegestütz am Block               | LIEGESTÜTZE        |
| 3 | igel-3.png | Bein-Dehnung am Bürostuhl         | BEIN-DEHNUNG       |
| 4 | igel-4.png | Marschieren / Aufwärmen stehend   | AUFWÄRMEN          |
| 5 | igel-5.png | Sitz-Dehnung am Holzstuhl         | SITZ-DEHNUNG       |
| 6 | igel-6.png | Arm-Curl stehend am Stuhl         | BIZEPS             |

(Zuordnung Bild↔Pose wird beim Implementieren anhand der echten Dateien final
verifiziert.)

### Auswahl-Mechanismus

TRMNL-Liquid hat **keinen** `random`/`sample`-Filter. Wir nutzen einen
zeit-gesetzten Index über den Refresh-Zeitstempel:

```liquid
{% assign seed = "now" | date: "%s" %}
{% assign pose_idx = seed | modulo: pose_count %}
{% assign say_idx  = seed | divided_by: 7 | modulo: saying_count %}
```

- `pose_idx` wählt die Pose aus dem `poses`-Array.
- `say_idx` wählt den Spruch aus dem Spruch-Array dieser Pose (durch das
  Teilen vor dem Modulo entkoppelt von der Pose-Wahl).
- Da TRMNL je Refresh neu rendert, ändert sich der Inhalt mit der Zeit.

## Anrede (Custom Field)

TRMNL-Custom-Field in `settings.yml` (`custom_fields`), Feldtyp `select`.
Select-Werte kommen in Liquid **kleingeschrieben** an:

```yaml
custom_fields:
  - keyname: anrede
    field_type: select
    name: Anrede
    description: Wie der Igel dich anspricht
    options: ['Neutral', 'Weiblich', 'Männlich']
```

Auslesen: `{{ trmnl.plugin_settings.custom_fields_values.anrede }}`
→ `neutral` / `weiblich` / `männlich`.

Die Sprüche enthalten einen Vokativ-Slot `{ANR}`. In Liquid wird der über den
`replace`-Filter ersetzt: `{{ saying | replace: "{ANR}", anr }}`, wobei `anr`
abhängig vom Feldwert gesetzt wird (Pool ebenfalls per zeit-gesetztem Index):

- `neutral` → `{ANR}` = `""` (kein Komma, kein Kosename)
- `weiblich` → `{ANR}` = `, ` + zufällig aus `[Süße, Liebes, Powerfrau]`
- `männlich` → `{ANR}` = `, ` + zufällig aus `[Großer, Champion, Kämpfer]`

(Vergleich in Liquid gegen die kleingeschriebene Option-Beschriftung. Falls der
Umlaut-Vergleich `== 'männlich'` Probleme macht, wird beim Implementieren auf
ASCII-Optionswerte ausgewichen.)

Beispiel-Roh-Spruch: `"Halt durch{ANR}! Der Boden ist dein Freund — and one, and two!"`
- neutral: „Halt durch! Der Boden ist dein Freund — and one, and two!"
- weiblich: „Halt durch, Süße! Der Boden ist dein Freund — and one, and two!"

## Die Sprüche (vollständig)

Ton: frech, witzig, 80er-Aerobic-Trainer. `{ANR}` = Vokativ-Slot (s. o.).

### Pose 1 — Planke
1. „Halt durch{ANR}! Der Boden ist dein Freund — and one, and two!"
2. „Zittern ist nur Applaus von innen. Weiter so!"
3. „Bauch anspannen, Po anspannen — Lächeln nicht vergessen{ANR}!"
4. „Noch zehn Sekunden{ANR}! Du bist härter als dein Schweißband."

### Pose 2 — Liegestütze
1. „Drück die Welt von dir weg{ANR} — and up, and down!"
2. „Ein Liegestütz für dich, einer für den Igel. Fair, oder?"
3. „Brust raus, Stacheln raus — wir geben alles!"
4. „Wer drückt, gewinnt{ANR}. Komm schon, noch fünf!"

### Pose 3 — Bein-Dehnung
1. „Bein lang machen, Sorgen kurz halten{ANR}. And stretch!"
2. „Der Stuhl ist heute dein Trainingsgerät — nutz ihn!"
3. „Schön dehnen, nicht reißen — wir sind ja kein Gummiband."
4. „Lang wie Spaghetti, stark wie Stahl{ANR}!"

### Pose 4 — Aufwärmen
1. „And march, and march! Aufwärmen ist die halbe Miete."
2. „Knie hoch, Laune hoch — los geht der Tanz{ANR}!"
3. „Beweg die Stacheln{ANR}, der Tag wartet nicht!"
4. „Erst wackeln, dann werkeln. And one, and two!"

### Pose 5 — Sitz-Dehnung
1. „Im Sitzen dehnen zählt auch — clever{ANR}!"
2. „Zeh berühren, Ehre gewinnen. Streck dich{ANR}!"
3. „Kein Aufstehen nötig — Ausreden auch nicht."
4. „Sanft dehnen, tief atmen — and relax!"

### Pose 6 — Bizeps
1. „And curl, and curl — die Ärmchen werden Ärmel sprengen!"
2. „Pump die Stacheln auf{ANR} — volle Power!"
3. „Heute leicht, morgen Legende. Weiter so{ANR}!"
4. „Bizeps wie ein Igel im Frühling — voll erblüht!"

## Bild-Handling

- **Vorverarbeitung:** Die Quell-PNGs sind ~1 MB groß (zu schwer zum Einbetten).
  Schritt im Plan: auf ~400px Breite herunterskalieren und in Graustufen-PNG
  wandeln → je ~30–80 KB. Ablage als `src/assets/igel-N.png`.
- **Darstellung:** `<img class="image image-dither" …>` ist Pflicht
  (Floyd-Steinberg-Dithering), sonst wirken die Bilder auf E-Ink ausgewaschen.
- **Lokale Preview:** Bilder über den Asset-Pfad des `trmnlp`-Servers
  referenziert.
- **Produktion (Gerät):** Bilder müssen für den TRMNL-Renderer erreichbar sein.
  Empfohlene Lösung: **Base64-Einbettung** der herunterskalierten PNGs direkt ins
  Liquid (selbst-enthaltend, kein externes Hosting; Netzwerk-Requests können auf
  dem Gerät fehlschlagen). Fallback: öffentliches Hosting (z. B. GitHub raw).
  Für den Start liegt der Fokus auf der lokalen Preview.

## Framework-Konformität (TRMNL Hard-Rules)

- **Keine inline-`style=""`-Attribute, keine `<style>`-Blöcke** — ausschließlich
  Framework-Klassen (`layout`, `grid`, `col--span-*`, `title`, `label`, `image`,
  `title_bar`).
- **Keine Emojis** im Markup (E-Ink hat keine Emoji-Glyphen).
- Markup **nicht** in `<div class="view view--*">` wrappen — die Plattform
  ergänzt den View-Wrapper selbst; Start direkt mit `layout`.
- URLs immer über `trmnl.com`.
- nil-Werte absichern (`{% if … %}` / `default:`), v. a. den Custom-Field-Wert.

## Preview-Server-Verkabelung

`.trmnlp.yml` wird so konfiguriert, dass:

- die Liquid-Templates aus `src/` geladen werden,
- die Custom-Field-Werte (`anrede`) für die Preview vorbelegt sind,
- der Server unter `localhost:4567` läuft und bei Dateiänderung neu rendert.

Abnahme-Kriterium: `trmnlp serve` startet, die Voll-Bildanzeige zeigt den Igel +
einen Spruch, und ein Wechsel des `anrede`-Werts verändert die Anrede sichtbar.

## Bewusst ausgeklammert (YAGNI)

- half_horizontal / half_vertical / quadrant Layouts
- Tageszeit-/Sequenz-Logik (nur Zufall)
- externe Datenquellen, APIs, Webhooks
- Mehrsprachigkeit (nur Deutsch)
```