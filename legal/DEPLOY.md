# NoteBay-Website & Legal-Seiten auf GitHub Pages veröffentlichen

Diese Anleitung richtet das öffentliche Repo **`Greg-null/NoteBay-site`** ein und veröffentlicht
darüber die Website und die Rechtsseiten. Die fertigen URLs matchen die Links, die im OnePager
(WP-39) bereits im Footer hinterlegt sind:

- `https://greg-null.github.io/notebay-site/` (OnePager)
- `https://greg-null.github.io/notebay-site/privacy.html` (Datenschutz)
- `https://greg-null.github.io/notebay-site/impressum.html` (Impressum)

> Du führst das Deployment selbst durch — die Dateien liegen fertig lokal unter `website/`
> (`index.html`) und `website/legal/` (`privacy.html`, `impressum.html`).

## In 5 Schritten

### 1. Repo anlegen
1. Auf <https://github.com/new> gehen (als Nutzer **Greg-null** eingeloggt).
2. **Repository name:** `notebay-site`
3. **Visibility:** `Public` (Pflicht für kostenloses GitHub Pages).
4. Häkchen bei „Add a README file" **weglassen** (wir pushen eigene Dateien).
5. **Create repository** klicken.

### 2. Dateien hochladen
Auf der leeren Repo-Seite **„uploading an existing file"** anklicken und diese drei Dateien
per Drag-and-drop in den **Repo-Root** ziehen (nicht in einen Unterordner):

| Quelle lokal                     | Ziel im Repo (Root) |
| -------------------------------- | ------------------- |
| `website/index.html`             | `index.html`        |
| `website/legal/privacy.html`     | `privacy.html`      |
| `website/legal/impressum.html`   | `impressum.html`    |

Danach unten **„Commit changes"** klicken. Wichtig: Alle drei Dateien liegen flach im Root,
damit die URLs oben stimmen (kein `/legal/`-Unterordner im Repo).

### 3. GitHub Pages aktivieren
1. Im Repo auf **Settings → Pages**.
2. Unter **Build and deployment → Source:** `Deploy from a branch`.
3. **Branch:** `main`, Ordner: `/ (root)`.
4. **Save** klicken.

### 4. Warten & URLs prüfen
1. Nach ~1–2 Minuten zeigt **Settings → Pages** oben die Live-URL an.
2. Diese drei Links im Browser öffnen und prüfen:
   - `https://greg-null.github.io/notebay-site/`
   - `https://greg-null.github.io/notebay-site/privacy.html`
   - `https://greg-null.github.io/notebay-site/impressum.html`
3. Sicherstellen, dass die Footer-Links im OnePager (Impressum / Datenschutz) korrekt auf die
   beiden Legal-Seiten zeigen.

### 5. Impressum-Angaben gegenprüfen
NoteBay wird von **Gregory Reinke als Privatperson** vertrieben (bestätigt 2026-07-04):
Gregory Reinke, Jean-Monnet-Str. 4, 28217 Bremen, reinke.support@gmail.com — so steht es
im Impressum. Vor Veröffentlichung nur noch prüfen, ob die Anschrift aktuell ist.

Falls etwas nicht passt: in `impressum.html` (bzw. für den Support-Kontakt auch in `privacy.html`)
korrigieren und erneut hochladen (Schritt 2).

---

## Änderungen später aktualisieren
Datei lokal ändern, dann im Repo dieselbe Datei über **„Add file → Upload files"** erneut hochladen
und committen. GitHub Pages baut automatisch neu (wenige Minuten).
