# 🛠️ Development & Maintenance Guide

This document contains everything you need to know about developing, building, and maintaining \*The Vtuber's Website\*\*.

---

## ⚡ Getting Started

### 📦 Prerequisites

- **Node.js**: v18 or later (v20+ recommended)
- **Git**: For version control and deployment

### 🏗️ Installation & Setup

1. **Clone the Repo**:
   ```bash
   git clone <repository-url>
   cd <project-folder>
   ```
2. **Install Dependencies**:
   ```bash
   npm install
   ```
3. **Launch local Workspace**:
   ```bash
   npm run dev
   ```

---

## 📜 Available Commands

| Command               | Description                                                                      |
| :-------------------- | :------------------------------------------------------------------------------- |
| `npm run dev`         | Spins up the **Vite dev server** with Hot Module Replacement (HMR).              |
| `npm run build`       | Compiles TypeScript and builds the project for production.                       |
| `npm run preview`     | Run the local **Production Preview** to test final assets.                       |
| `npm run deploy:prod` | Automated pipeline: Switches CNAME, pushes to Production Repo & reverts to Test. |

---

## 🧩 Maintenance & Updates

### 🌍 Adding/Editing Translations

The project uses a custom i18n system. `en.ts` is the reference language: every other language must contain exactly its keys.

**Adding a new language** (e.g. French, `fr`):

1. Copy `src/lang/de.ts` to `src/lang/fr.ts`, rename the export to `fr` and translate every value. Keep the type `CompleteTranslation` – the build then fails if a key is missing or misspelled.
2. Add one entry to the `languages` registry in `src/lang/index.ts`:
   `{ code: "fr", name: "Français", strings: fr }` (the name in the language itself).
3. Add the flag as `public/images/flags/fr.svg`.
4. Run `npm run audit:i18n` – it checks every language against `en.ts`, the registry and the flags.

Everything else follows automatically: the language menu is built from the registry, and visitors whose browser prefers the new language get it on their first visit. The imprint and privacy texts (`imprint_full_text`, `privacy_full_text`) are legal texts – have a translation of those reviewed.

**Adding/changing a text:** add the key to `en.ts` first, then to every other language (the build points out where it is missing), and reference it in HTML via `data-i18n`, `data-i18n-html`, `data-i18n-placeholder`, `data-i18n-aria-label`, `data-i18n-title` or `data-i18n-alt` (image descriptions). Purely decorative images (emotes, logos inside an already labelled link) get `alt=""` instead.

### ♿ Barrierefreiheit (WCAG 2.2 AA)

Die Seite wurde im September 2026 gegen WCAG 2.2 AA geprüft und korrigiert. Damit das so bleibt, gelten diese Regeln:

**Farben**
- **Niemals feste Farbwerte für Text oder Flächen.** Immer die Theme-Variablen aus `src/styles.css` nutzen. Drei der schlimmsten gefundenen Fehler kamen genau daher: Das Mobilmenü hatte `#0a0a1f` fest verdrahtet und war im Light-Theme unlesbar (1,54:1), ebenso die Partner-Beschriftung (1,26:1) und die Erfolgsmeldung im Kontaktformular (2,28:1).
- **Mindestkontrast:** 4,5:1 für normalen Text, 3:1 ab 24px bzw. ab 18,66px fett.
- **Verlaufs-Buttons:** Textfarbe ist `var(--bg-primary)`, nicht `#ffffff`. Weiß erreicht auf dem Akzent-Verlauf nur 2,4–3,4:1.
- `--accent-secondary` ist im Light-Theme `#0f766e` (nicht `#0d9488`). Der hellere Ton liegt luminanzmäßig in der Mitte, dort erreicht **weder** heller **noch** dunkler Text 4,5:1.

**Bilder**
- Inhaltlich → `data-i18n-alt` mit Text in allen Sprachdateien.
- Dekorativ (Emotes, Logos in einem Link, der schon `aria-label` hat) → `alt=""`. Sonst liest der Screenreader Dinge doppelt vor.

**Interaktion**
- Klappelemente brauchen `aria-expanded` **und** `aria-controls`, und `aria-expanded` muss in *allen* Schließpfaden gepflegt werden (Auswahl, Escape, Klick daneben).
- Aktiver Menüpunkt bekommt `aria-current="page"`.
- Statusmeldungen brauchen `role="status"`, sonst erfährt ein Screenreader nichts.
- `aria-label` muss den sichtbaren Text enthalten (WCAG 2.5.3), sonst funktioniert Sprachsteuerung nicht.
- Klickziele mindestens 24px hoch.
- `prefers-reduced-motion` ist in `styles.css` berücksichtigt; neue Animationen dort mit abschalten.

**Messfallen** (haben in der Prüfung mehrfach zu falschen Ergebnissen geführt)
- Ein Kontrast-Skript, das `background-color` liest, **übersieht Farbverläufe** (`background-image`) und meldet Unsinn. Verlaufsstopps einzeln prüfen.
- **Verlaufstext** (`background-clip: text` + `-webkit-text-fill-color: transparent`) macht `color` unsichtbar. Dort zählen die Verlaufsfarben, nicht `color`.
- Der **mausgesteuerte Lichtschein** hellt Hintergründe auf. Ein Badge lag statisch bei 4,63:1 und unter dem Schein bei 4,37:1.
- Während der Einblend-Animationen ist `opacity: 0` – Messungen erst nach ~3 Sekunden.
- Fehler **immer im echten Browser gegenmessen**, nicht nur im CSS lesen.

### 📈 Analytics & Datenschutz

- **Google Analytics lädt erst nach „Alle akzeptieren"** (`initAnalytics()` in `main.ts`). Wer „Nur essenzielle" wählt, wird nicht gezählt.
- **Zwei GA-Properties, nach Domain getrennt:** `nhywyll.com` → `G-8NZ8JX48ZP`, `test.nhywyll.com` → `G-R18WRP31XQ`. Die Auswahl macht `isProduction()` über den Hostnamen.
- **Metricool wurde wieder entfernt** (Tracker und Datenschutz-Abschnitt).
- **Der Twitch-Live-Status** ruft `decapi.me` auf. Das überträgt die IP der Besucher an einen Dritten und steht deshalb als Abschnitt 7 in der Datenschutzerklärung. `decapi.me` muss in der CSP unter `connect-src` stehen, sonst blockiert der Browser den Aufruf still und der Indikator zeigt dauerhaft „offline".
- **Jede neue externe Domain** (Skript, Bild, API) braucht einen CSP-Eintrag in `src/components/head-common.html` **und** meist einen Absatz in der Datenschutzerklärung.

### ⚠️ Stolpersteine

- **`init()` muss am Dateiende von `main.ts` stehen.** Das Modul läuft oft schon bei `readyState === "interactive"`. Steht der Aufruf weiter oben, sind später deklarierte Konstanten noch nicht belegt, `init()` bricht ab und die Seite zeigt **nur Übersetzungsschlüssel**. Genau das ist vom 06.09. bis 15.09.2026 passiert, und zwar nur bei Besuchern mit Cookie-Zustimmung – deshalb fiel es tagelang nicht auf. Der Kommentar dazu steht im Code.
- **Ungültige gespeicherte Sprache.** Steht in `localStorage` ein Sprachcode, den es nicht (mehr) gibt, fiel die Seite früher auf rohe Schlüssel zurück. `getInitialLanguage()` prüft den Wert jetzt und nimmt sonst die Browsersprache bzw. `DEFAULT_LANGUAGE`. Beim Entfernen einer Sprache also nicht wieder aufweichen.
- **Der Footer überlebt Seitenwechsel.** Die SPA tauscht nur `#main-content` aus. Wer in einer pro Seite laufenden Funktion einen Listener an Footer, Header oder `document` hängt, sammelt bei jedem Wechsel einen weiteren an. Solche Listener gehören in den `isFirstLoad`-Block von `init()` oder brauchen eine Sperre (siehe `setupEasterEgg`).
- **Der SPA-Router darf Klicks mit Strg/Cmd/Umschalt/Alt und Mittelklick nicht abfangen**, sonst ist „In neuem Tab öffnen" kaputt. Ebenso reine `#`-Links in Ruhe lassen, sonst springt die Seite nach oben.
- **Der Cookie-Banner ist `position: sticky`, nicht `fixed`.** Er belegt Platz im Seitenfluss und muss deshalb das letzte Element im `<body>` bleiben. Weiter oben eingefügt schiebt er den Header nach unten.

### 🔍 SEO & Sitemap

Each page has a dedicated `<title>` and `<meta description>` in the root HTML files. 

**Automatisierung:**
- **Entry Points:** Das System erkennt neue HTML-Dateien im Hauptverzeichnis automatisch. Du musst die `vite.config.ts` **nicht** mehr manuell anpassen.
- **Sitemap:** Die **`sitemap.xml`** wird bei jedem Build automatisch generiert und enthält alle gefundenen Seiten.

---

## ⚙️ Deployment

The project uses a two-stage deployment process via GitHub Pages:

### Test / Staging (`test.nhywyll.com`)

Changes to the `main` branch are automatically pushed to the test subdomain.

```bash
git push origin main
```

### Production (`nhywyll.com`)

To push the current state to the main domain, use the integrated script:

```bash
npm run deploy:prod
```

_This automatically toggles the CNAME configuration and pushes to the production repository._

**Wie das zusammenhängt (wichtig, sonst verwirrend):**

- Es sind **zwei GitHub-Repos**: `origin` = `NhywyllTV/test_website` (Staging), `production` = `NhywyllTV/nhywyll-website` (Live). Beide bauen per GitHub Actions aus demselben Quellcode.
- Die Domain entscheidet die Datei `public/CNAME`. Lokal steht dort immer `test.nhywyll.com`. `deploy:prod` legt einen temporären Branch an, setzt den CNAME auf `nhywyll.com`, pusht ihn per **Force** auf `production/main` und stellt lokal alles zurück. Deshalb hat `production/main` immer genau einen Commit mehr (`chore: release to production`), und deshalb funktioniert ein direkter `git push production main` nicht.
- **Arbeitsablauf:** immer erst `git push origin main`, den Workflow abwarten, auf **test.nhywyll.com** prüfen – und erst dann `deploy:prod`. Die Testumgebung existiert genau dafür.
- **`deploy:prod` committet nur `public/CNAME` – sonst nichts.** Inhaltsänderungen müssen vorher committet sein. Uncommittete Dateien nimmt `git checkout -b` zwar ins Arbeitsverzeichnis des temporären Branches mit, in den Release-Commit kommen sie aber nicht. Das Skript meldet trotzdem fröhlich „✅ Deployment erfolgreich abgeschlossen!", und live ändert sich nichts – es gibt keine Warnung. Aufgefallen am 01.10.2026 beim Wechsel des Bluesky-Handles. Also vor jedem `deploy:prod` ein `git status` – das Arbeitsverzeichnis muss sauber sein.
- **Falle beim Prüfen:** `gh run list` direkt nach dem Push liefert oft noch den *vorherigen* Lauf. Die `headSha` gegen `git rev-parse --short HEAD` prüfen, sonst misst man den alten Stand.

---

## 🏗️ Project Structure

```text
test_website/
├── scripts/                # Deployment Automation
├── public/                 # Static assets (images, logos, icons, fonts)
│   ├── images/
│   │   ├── Emotes/           # Live stream emotes
│   │   ├── media/            # Social media icons
│   │   └── artwork-library/  # Curated artist showcase and commissions
│   ├── fonts/              # Local typography (Outfit)
│   ├── sitemap.xml         # SEO engine optimization
│   └── robots.txt          # Crawler instructions
├── src/                    # Source files for build
│   ├── main.ts               # Core logic: i18n, Transitions, Effects & UI
│   ├── styles.css            # Global style themes and variables
│   └── lang/                 # Translations (en.ts = reference, de.ts) + registry in index.ts
├── index.html              # Hero, About & FAQ
├── links.html              # Social hub
├── contact.html            # Business center
├── credits.html            # Artist Showcase
├── imprint.html            # Legal framework
└── 404.html                # Custom error page
```

---

## 📌 Offene Punkte (Stand 30.09.2026)

Gefunden und geprüft, aber **nicht** behoben:

1. **Rohe Übersetzungsschlüssel im statischen HTML.** `imprint.html` und `404.html` haben `<title>page_title_imprint</title>` bzw. `page_title_404`, dazu einige Überschriften und Texte auf index/credits. Erst JavaScript ersetzt sie. Wer kein JS ausführt – Link-Vorschauen in Discord/WhatsApp, manche Crawler – sieht „page_title_imprint". **Fix:** englischen Text als Standard ins HTML schreiben, so wie es die anderen Seiten schon machen.
2. **Cookie-Banner überdeckt das Mobilmenü.** Banner `z-index: 10000`, Menü `var(--z-menu)` = `6000`. Solange noch keine Cookie-Auswahl getroffen wurde, verdeckt der Banner auf dem Handy den untersten Menüpunkt (WCAG 2.4.11).
3. **Copyright-Jahr fest verdrahtet.** `&copy; 2026` in `src/components/footer.html` – ab Januar veraltet.
4. **`<html lang="en">` steht statisch in allen Seiten.** JavaScript korrigiert es, aber vor dem Laden ist der Wert für deutsche Besucher falsch.
5. **Meta-Beschreibungen und Open-Graph-Texte werden nicht übersetzt.** Sie sind nur englisch vorhanden; die Sprachumschaltung erfasst sie nicht.
6. **Sprache nicht in der URL.** Es gibt keine getrennten URLs pro Sprache und kein `hreflang`. Suchmaschinen sehen nur die englische Fassung. Für mehr Sprachen irgendwann relevant.
7. **Offene Frage:** „Mehr Sprachdateien" ist noch nicht geklärt – gemeint sein kann (a) weitere Sprachen wie `fr.ts` anlegen oder (b) die großen Dateien `de.ts`/`en.ts` pro Seite/Thema aufteilen. Für (b) müsste `src/lang/index.ts` die Teildateien zusammenführen.

**Noch nicht live:** `deploy:prod` lief zuletzt am 21.09. Auf Staging, aber nicht auf nhywyll.com, liegen der i18n-Umbau (`15d2ccf`) und die übersetzten Bildbeschreibungen (`f319fa8`).

---

## 🎨 Design System & Style Guidelines

To keep the website looking cohesive and aligned with Nhywyll's VTuber identity, all styles should adhere to the following logo-matched color system and visual guidelines:

### 1. Logo-Matched Color Palette
All color values are defined as CSS variables in `src/styles.css`.

| Theme | Element | Variable | Color | Note |
| :--- | :--- | :--- | :--- | :--- |
| **Dark Theme** | Primary Background | `--bg-primary` | `#0f101b` | Deep dark slate/indigo |
| | Secondary Background | `--bg-secondary` | `#17192c` | Slightly lighter dark indigo |
| | Cards & Containers | `--bg-card` | `rgba(30, 33, 58, 0.65)` | Dark indigo glass panel |
| | Primary Accent | `--accent-primary` | `#7d88c4` | Dusty lavender from logo text |
| | Secondary Accent | `--accent-secondary` | `#4db6ac` | Soft mint-teal from logo stars |
| **Light Theme**| Primary Background | `--bg-primary` | `#f6f7fb` | Soft lavender cool-white |
| | Card Background | `--bg-card` | `rgba(235, 238, 248, 0.85)`| Soft lavender-gray |
| | Primary Typography | `--text-primary` | `#2d314e` | Deep dark slate-navy |
| | Primary Accent | `--accent-primary` | `#5c6494` | Darker dusty lavender |
| | Secondary Accent | `--accent-secondary` | `#0f766e` | Deep teal (dark enough for light text on the button gradient, 5.1:1) |

### 2. Typography
* **Primary Font**: `Outfit` (loaded locally in `public/fonts/`). Used for headings, body, and navigation.
* **Secondary / Fallbacks**: `Segoe UI`, `Tahoma`, `Geneva`, `Verdana`, `sans-serif`.

### 3. Visual Guidelines (Cozy, Not "Tech-RGB")
* **Soft & Organic**: Avoid pure black/white backdrops and harsh neon/cyber-RGB scrolling lines. The design should feel cozy, warm, and integrated with the character illustration.
* **Glassmorphic Panels**: Cards should use subtle blur (`backdrop-filter: blur(20px)`) and thin, tinted borders (`--glass-border`) matching the primary accent color.
* **Gradients**: Text gradients and link underlines must transition smoothly between the primary (lavender) and secondary (teal) accents.

