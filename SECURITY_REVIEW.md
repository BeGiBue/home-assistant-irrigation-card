# Security-Review und HACS-Major-Vorbereitung: Irrigation Card

Stand: 25.10.2026. Repository: `BeGiBue/home-assistant-irrigation-card`.
Ausgangsbasis: `f6c3e8c3e44f81ba765af23ba662898fcbd7cb6d` (**1.1.4**).
Zielversion: **2.0.0**. Befundzeilen beziehen sich auf diese Ausgangsbasis.

## Ergebnis und wichtiger HACS-Status

Drei reproduzierbare Probleme in der Bewässerungsbedienung sind behoben.
Keine bestätigte XSS-, Authentifizierungs- oder Credential-Lücke und keine offenen
bestätigten hohen/kritischen Befunde im geprüften Umfang.

**Die offizielle HACS-Aufnahme ist noch ausstehend.** Der Eintrag fehlt im geprüften
`hacs/default/plugin`; [Aufnahme-PR #12053](https://github.com/hacs/default/pull/12053)
ist offen und ungemergt. Die Dokumentation behauptet daher keine bereits erfolgte Aufnahme.
Bis die Karte im Katalog verfügbar ist, bleibt die Custom-Repository-Installation bestehen.
Die Migration wird für den tatsächlichen Aufnahmezeitpunkt vorbereitet.

## Befunde

| ID | Repository | Datei / Zeile | Risiko | Schweregrad | Korrektur / Status |
|---|---|---|---|---|---|
| IRR-01 | home-assistant-irrigation-card | `irrigation-card.js:117–120`, Minutentasten `:148` | Fehlt der dauerfähige Hydrawise-Statussensor, fällt ein Minutenstart still auf einen manuellen Schalter oder ein Ventil zurück. Die gewählte Dauer wird nicht übermittelt. Eine angezeigte 5-Minuten-Bewässerung kann mit der Controller-Vorgabe laufen. Der geprüfte HA-Core 2026.10.2 verwendet beim manuellen Hydrawise-Schalter 15 Minuten; eine unbegrenzte Laufzeit wurde nicht nachgewiesen. | **Mittel** (unbeabsichtigt längere physische Bewässerung) | Minutentasten ausschließlich mit dauerfähigem Hydrawise-Ziel; andere Starts ausdrücklich **Controller-Vorgabe**. Positive Dauern ohne passendes Ziel, ungültige Werte und Offline-Starts werden abgewiesen. **Behoben.** |
| IRR-02 | home-assistant-irrigation-card | `irrigation-card.js:145–148` | Nach dem Start zeigt die Karte bei noch altem Zustand `off` nur gesperrte Starttasten, keine nutzbare Stopptaste. Verzögerte Cloud-Zustände können den Abbruch über die Karte verhindern. | **Niedrig** (Bedienbarkeit / physische Steuerung) | Stopp bleibt während der Startbestätigung sichtbar und bei verfügbarem Ziel bedienbar. Ein laufender Startauftrag wird vor dem Stopp abgearbeitet; fehlende Ziele und doppelte Stoppaufträge führen zu keinem zusätzlichen/ungezielten Dienstaufruf. **Behoben.** |
| IRR-03 | home-assistant-irrigation-card | `irrigation-card.js:81`, `:118`, `:125` | Nach 90 Sekunden wird der Wartezustand auch bei unbeantwortetem Auftrag gelöscht; ein weiterer Start ist dann möglich, obwohl der alte Start oder Stopp noch aussteht. Ein verspäteter Auftrag kann weiter physisch wirksam werden. | **Niedrig** (Race-Bedingung bei verzögerten Antworten) | Noch unbeantwortete Aufträge bleiben gesperrt. Bestätigungsfrist beginnt nach erfolgreicher Dienstannahme; Antwort-Identität verhindert das Überschreiben neuerer Aufträge. Fehler entfernen nur den eigenen Wartezustand. **Behoben.** |
| IRR-H01 | home-assistant-irrigation-card | `irrigation-card.js:15`, `:60` | Normale Objekte als Registry-/Aktionsindizes können bei absichtlich ungültigen YAML-Geräte-IDs wie `constructor` geerbte Eigenschaften auflösen. Reguläre HA-Geräte-IDs sind generiert; kein privilegierter Angriff nachgewiesen. | Härtung | Prototyplose Registry-/Aktionsindizes. Unbekannte reservierte IDs werden ignoriert. **Umgesetzt und getestet.** |
| IRR-INFO01 | home-assistant-irrigation-card | `irrigation-card.js:177`, `:180–182` | Externe Bild-URLs übertragen Browser-Netzwerkmetadaten; HTTP bietet keinen Transportschutz. | Niedrig / Konfigurationsrisiko | Lokale `/local/…`-Bilder und keine Zugangsdaten in URLs empfohlen. Attribute werden escaped. Ein `img.src` ist kein nachgewiesener JavaScript-Ausführungskontext; kein Schema- oder SVG-XSS-Exploit behauptet. |

## Weitere geprüfte Vertrauensgrenzen

- **DOM/HTML:** Konfigurationstitel, Pumpen-/Zonen-/Controller-Namen, Entity-IDs, Zustände,
  Einheiten und Optionswerte werden über `_e()` für Text-/Attributkontexte maskiert.
  Zahlen in Skalierung, Prozentbalken und Dauertasten werden numerisch erzeugt.
  Ereignisse werden als JavaScript-Listener gebunden; keine dynamischen Inline-Handler.
- **Dynamische URLs:** frei konfigurierbare Bild-URL ausschließlich in `img.src`; das
  Standardbild ist eingebettet. Keine dynamische Navigation oder Script-URL.
- **API und Berechtigungen:** nur `hass.callService` über die authentifizierte HA-Sitzung
  für Hydrawise, Home Assistant und Ventile. Keine eigene Cloud-API, kein `fetch`,
  eigener WebSocket-Client, gespeicherter Zugriffstoken oder Controller-Passwort.
  HA und Geräteintegration prüfen die Rechte serverseitig; UI-Sperren sind keine
  Autorisierungsgrenze.
- **Physische Wirkung:** Start/Stopp, Automatik und Pumpenschalter können reale Geräte
  schalten. Die Prüfungen verwenden ausschließlich lokale Attrappen und gemockte
  Dienste. Es wurden keine Pumpen oder Ventile dieser Anlage betätigt.
- **Python und Dateien:** keine Python-Integration, eigenen Laufzeit-Dateipfade oder
  blockierenden Python-Aufrufe im HA-Event-Loop. Die neue Python-Datei dient lokalen
  Browser-Tests mit festen Repository-Pfaden.
- **Geheimnisse:** keine hartcodierten Zugangsdaten im geprüften Laufzeitcode.
  Kein vollständiger Geheimnisscan über die gesamte Git-Historie.
- **Abhängigkeiten:** keine npm-/Python-Laufzeitabhängigkeiten oder mitgelieferten
  Fremdbibliotheken. Playwright 1.64.1 ist nur Testwerkzeug. Das lokal wiederverwendete
  isolierte Test-Venv wurde mit `pip-audit` einschließlich transitiver Pakete geprüft:
  **0 bekannte Befunde** für Playwright 1.64.1, greenlet 3.5.6, pyee 14.0.0 und
  typing-extensions 4.16.0. Das Venv enthält zusätzlich Pillow 13.0.0 (ebenfalls 0),
  das Irrigation weder benötigt noch über `requirements-dev.txt` installiert.
  Der Abgleich ist eine Momentaufnahme und kein Audit von HA-Core oder Browser-Code.

## Korrekturen und Tests

- `node --check irrigation-card.js`: bestanden.
- `node --test --test-isolation=none tests/security.test.mjs`: **17 bestanden**.
  Geprüft werden Escaping, Registry-Daten, Duration-Grenzen, expliziter Standardlauf,
  genaue Dienstziele, Offline-/Fehlerzustände, Doppelstarts, ausstehende Antworten,
  Stopp-Reihenfolge und Pumpenaktion.
- Dieselbe neue Suite auf der unveränderten Ausgangsbasis: **11 erwartete Fehler bei
  17 Prüfungen**. Die bestehenden Escaping-Fälle bestehen bereits auf 1.1.4.
- `python tests/browser_security.py`: **33 bestanden** in Chromium bei **350 und
  800 px** Breite. Tatsächliche DOM-Attribute/-Texte, Klicks und gemockte Dienste
  werden geprüft; keine injizierten Elemente, Event-Attribute oder JavaScript-Fehler.
- Vorschauen beider Breiten erzeugt und visuell geprüft. Es sind Testdaten mit
  HA-Attrappen; die endgültige Schrift-/Icon-Darstellung kommt im Produkt aus HA.
- `python -m compileall -q tests`, `git diff --check`: bestanden.
- Standard-Konfiguration (`DEF`) und eingebettetes Bild unverändert gegen Baseline geprüft.
- `VERSION`, JavaScript-Konstante/Kopfzeile, README, Changelog und Release Notes
  konsistent auf **2.0.0**; Release-Notes-Inhalt entspricht dem Changelog-Eintrag.
- Die neuen Logik- und Browserprüfungen laufen zusätzlich zur bestehenden HACS-CI.
  GitHub-CI ist vor Freigabe am konkreten PR-Commit zu prüfen.

## Dokumentation, Migration und Version

README ist auf Funktionen, Einrichtung, Bedienung, Voraussetzungen und Grenzen
ausgerichtet; Glas-Look und dekorative Anordnungsbegründungen sind entfernt.
Der offizielle HACS-Katalog ist der künftige Standardweg **nach** Aufnahme.
Bis dahin wird das bestehende Custom Repository verwendet.

Vor der Migration zuerst die Katalogverfügbarkeit prüfen, nur den benutzerdefinierten
Repository-Eintrag entfernen, **die Karte nicht deinstallieren**, HACS aktualisieren
und bestehende Ressourcen/Konfigurationen prüfen. Changelog und Release Notes
dokumentieren ausdrücklich ausstehende Aufnahme und spätere Migration.

**Major bedeutet nicht automatisch Breaking Changes.** Kartentyp
`custom:irrigation-card`, Standard-Entitäten, Konfigurationsschlüssel und normale
Minutenstarts bleiben erhalten. Der bisher stille Start-Fallback hat jetzt eine
eindeutige Beschriftung und verspricht keine ungeprüfte Dauer.

## Restrisiken und Freigaben

- Auf die reale Katalogaufnahme warten, bevor Custom-Repository-Einträge entfernt
  oder die Aufnahme als abgeschlossen beworben wird. Am bestehenden HACS-Aufnahme-PR
  werden keine Änderungen oder Kommentare vorgenommen.
- HA-/Controller-/Cloud-Zustände können verzögert, veraltet oder falsch sein.
  Eine gestartete oder gestoppte Anzeige beweist keinen Wasserfluss bzw. geschlossenen
  Ventilzustand. Bei unbeantworteten Aufträgen sind neue Starts gesperrt; der
  Controller oder die Anlage muss bei Cloud-Ausfall direkt geprüft werden.
- Standardläufe verwenden Controller-Regeln; kein zusätzlicher Browser-Abschalttimer.
  Eine getrennte Pumpe bleibt unabhängig bedienbar; keine eigene Verriegelung,
  Trockenlaufsicherung oder Überwachung. Automatik-Aus kann gemäß Hydrawise
  die Zone für 365 Tage pausieren.
- Reale HA-/Hydrawise-/HACS-/Companion-/Kiosk-Prüfung sowie Maintainer- und
  Veröffentlichungsfreigabe bleiben vor einem Release erforderlich.
- Keine aktiven Rulesets; gelesene Branch-Metadaten melden `protected:false`.
  Schutzregeln werden weder verändert noch umgangen.
- Das Repository enthält nur einen Validate-Workflow und **keinen Release-Workflow**.
  Es werden kein Merge, Tag, Release oder Veröffentlichungsworkflow ausgelöst.

Dieser Folgeauftrag hebt den ursprünglichen Ausschluss der Irrigation Card ausdrücklich
auf. Die zehn zuvor geprüften Repositories werden für diesen Auftrag nicht verändert.

