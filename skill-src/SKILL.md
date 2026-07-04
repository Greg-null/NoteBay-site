---
name: notebay-mcp
description: >
  Arbeitet mit der NoteBay-Notizbibliothek des Nutzers über den lokalen MCP-Server
  der NoteBay-macOS-App (Notizen lesen, durchsuchen, anlegen, ändern, umstrukturieren).
  Verwenden, wenn der Nutzer über seine Notizen, Ordner oder deren Inhalte sprechen
  will oder NoteBay/„meine Notizen" erwähnt. Zugriff besteht ausschließlich auf
  Ordner, die der Nutzer explizit für KI-Zugriff freigegeben hat.
---

# NoteBay MCP — Anleitung für KI-Agenten

NoteBay ist eine native macOS/iOS-Notiz-App. Die macOS-App enthält einen lokalen
MCP-Server. Alles, was du über diesen Server siehst, hat der Nutzer **pro Ordner
ausdrücklich freigegeben** — alles andere existiert für dich nicht. Verhalte dich
entsprechend: Was du nicht siehst, darfst du weder erraten noch erfragen wollen.

## 1. Verbindung

- **Endpunkt:** `http://127.0.0.1:27431/mcp` (Streamable HTTP; Port konfigurierbar,
  Default 27431). Der Server läuft **nur, solange die NoteBay-App geöffnet ist**,
  und bindet ausschließlich an localhost.
- **Auth:** Jeder Request braucht `Authorization: Bearer <Pairing-Token>`. Das Token
  zeigt die App unter **Einstellungen → KI-Zugriff** (dort auch erneuerbar). Ohne
  gültiges Token antwortet der Server mit 401 ohne weitere Details.
- **Client-Konfiguration** (Claude Desktop / Claude Code, `mcpServers`-Konvention):

```json
{
  "mcpServers": {
    "notebay": {
      "type": "http",
      "url": "http://127.0.0.1:27431/mcp",
      "headers": {
        "Authorization": "Bearer <TOKEN-AUS-DEN-EINSTELLUNGEN>"
      }
    }
  }
}
```

- **Drei Betriebsstufen** (Nutzer-Einstellung): *Aus* (Server läuft nicht),
  *Nur Lesen* (Schreib-Tools liefern `read_only_mode`), *Lesen + Schreiben*.

## 2. Sicherheitsmodell — was du wissen musst

- Sichtbar ist ein Objekt nur, wenn sein Ordner **und alle übergeordneten Ordner**
  freigegeben sind. Geblockte Objekte tauchen nirgends auf: nicht in Listings,
  nicht in Suchergebnissen, nicht per ID.
- Zugriff auf eine unbekannte/geblockte ID liefert **`not_found`** — niemals
  „forbidden". Du kannst also nicht unterscheiden, ob etwas nicht existiert oder
  geblockt ist. Akzeptiere das und frage nicht nach.
- **Löschen gibt es nicht.** Es existieren keine `delete_*`-Tools, kein
  Papierkorb-Zugriff, kein Zugriff auf Einstellungen/Token/Audit-Log. Aufrufe
  solcher Methoden ergeben JSON-RPC „Method not found". Das Äußerste ist
  `archive_note`/`archive_folder` (umkehrbar).
- Vor **jeder** schreibenden Notiz-Operation legt die App automatisch einen
  Snapshot an (30 Tage aufbewahrt, vom Nutzer wiederherstellbar). Jede Operation
  wird im Audit-Log der App protokolliert (90 Tage).

## 3. Die 16 Tools mit Beispielaufrufen

IDs sind stets UUID-Strings aus vorherigen Listings — es gibt keine Pfade.
Beispiele zeigen die `arguments` eines `tools/call`.

### Ordner

| Tool | Zweck | Beispiel-Arguments |
|---|---|---|
| `list_folders` | Freigegebene (Unter-)Ordner listen; ohne `parent_id` die freigegebenen Wurzelordner | `{}` bzw. `{"parent_id":"FA3F7022-48AE-4A77-A46D-D1398555670C"}` |
| `create_folder` | Unterordner unter freigegebenem Ordner anlegen | `{"parent_id":"FA3F7022-…","name":"Recherche"}` |
| `rename_folder` | Ordner umbenennen | `{"folder_id":"1B7E43D0-…","name":"Recherche 2026"}` |
| `move_folder` | Ordner verschieben (Quelle **und** Ziel müssen freigegeben sein) | `{"folder_id":"1B7E43D0-…","new_parent_id":"9C2A11EE-…"}` |
| `archive_folder` | Ordner rekursiv archivieren | `{"folder_id":"1B7E43D0-…"}` |
| `unarchive_folder` | Archivierung aufheben | `{"folder_id":"1B7E43D0-…"}` |

### Notizen

| Tool | Zweck | Beispiel-Arguments |
|---|---|---|
| `list_notes` | Notizen eines Ordners (inkl. archivierter, ohne Papierkorb) | `{"folder_id":"FA3F7022-…"}` |
| `read_note` | Notiz als Markdown lesen; optional Block-IDs | `{"note_id":"E1AA20C4-…","include_block_ids":true}` |
| `search_notes` | Volltextsuche über freigegebene Notizen (Titel + Snippet je Treffer) | `{"query":"Rollback","limit":20}` |
| `create_note` | Notiz in freigegebenem Ordner anlegen | `{"folder_id":"FA3F7022-…","title":"Wochenreview"}` |
| `update_note` | **Gesamten** Notizinhalt aus Markdown ersetzen | `{"note_id":"E1AA20C4-…","markdown":"## Stand\nAlles grün."}` |
| `rename_note` | Titel ändern | `{"note_id":"E1AA20C4-…","title":"Go-Live Plan v2"}` |
| `move_note` | Notiz in anderen freigegebenen Ordner verschieben | `{"note_id":"E1AA20C4-…","new_folder_id":"9C2A11EE-…"}` |
| `duplicate_note` | Notiz im selben Ordner duplizieren | `{"note_id":"E1AA20C4-…"}` |
| `archive_note` | Notiz archivieren | `{"note_id":"E1AA20C4-…"}` |
| `unarchive_note` | Archivierung aufheben | `{"note_id":"E1AA20C4-…"}` |

## 4. Markdown-Konventionen & Roundtrip

`read_note` liefert Markdown, `update_note` akzeptiert Markdown. Der Dialekt:

- Überschriften `#`/`##`/`###` (H1–H3), Absätze, Zitate `>`, Trennlinien `---`.
- Listen: `- ` (Punkt), `1. ` (nummeriert), Checklisten `- [ ]` / `- [x]`.
  Einrückung = 2 Leerzeichen pro Ebene.
- Code-Blöcke als ```` ```sprache ```` -Fences (Sprache bleibt erhalten).
- Tabellen als GFM-Tabellen; Zeilenumbruch in Zellen als `<br>`.
- Inline: `**fett**`, `*kursiv*`, `<u>unterstrichen</u>`, `~~durchgestrichen~~`,
  `` `code` `` und Highlight als `==Text==`. Beim Zurückschreiben erhält ein
  Highlight die Standardfarbe Gelb (Farbnuancen codiert Markdown nicht).
- **Anhänge erscheinen als stabile Platzhalter `[[attachment:UUID]]`** (Bilder und
  Dateien). Lass diese Platzhalter beim Bearbeiten exakt unverändert an ihrer
  Stelle stehen — dann bleiben die Anhänge verlustfrei erhalten (Roundtrip-
  Garantie). Entfernst du einen Platzhalter, gilt der Anhang als aus der Notiz
  gelöscht (der Nutzer kann ihn nur über den Snapshot-Verlauf zurückholen).
- Mit `include_block_ids: true` stellt `read_note` jedem Block einen Kommentar
  `<!-- block:UUID -->` voran. Nutze das, um gezielt einzelne Blöcke zu ändern:
  unveränderte Blöcke samt Kommentar 1:1 zurückschreiben, nur die betroffenen
  Blöcke anfassen. `update_note` ersetzt immer den gesamten Inhalt — der übliche
  Ablauf ist daher: `read_note` → gezielt editieren → komplett zurückschreiben.

## 5. Fehlercodes (maschinenlesbar im Tool-Result)

| Code | Bedeutung | Richtiges Verhalten |
|---|---|---|
| `not_found` | ID existiert im freigegebenen Bereich nicht (nicht vorhanden **oder** geblockt — ununterscheidbar) | Nicht erneut versuchen, nicht spekulieren; ggf. Listing auffrischen |
| `note_limit_reached` | Free-Limit erreicht (20 Notizen; `create_note`/`duplicate_note`) | Nutzer informieren, dass die Vollversion (einmalig 2,99 €) das Limit aufhebt; **kein** Retry |
| `read_only_mode` | Schreib-Tool im Modus „Nur Lesen" | Nutzer bitten, in NoteBay unter Einstellungen → KI-Zugriff „Lesen + Schreiben" zu aktivieren |
| `invalid_operation` | Fachlich unzulässig (z. B. Zyklus beim Verschieben) | Operation anders planen |
| `invalid_params` | Parameter fehlen/falsch typisiert/keine UUID | Aufruf korrigieren |

Verbindungsfehler (keine Antwort / 401): Die App ist nicht gestartet, der
KI-Zugriff steht auf „Aus", oder das Token ist abgelaufen/erneuert worden —
bitte den Nutzer, NoteBay zu öffnen und Einstellungen → KI-Zugriff zu prüfen.
