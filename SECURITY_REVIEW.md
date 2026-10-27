# Security-Review: Irrigation Control Card

Repository: `BeGiBue/home-assistant-irrigation-control-card`.
Der Review entstand am 25.10.2026 für **2.0.0** (Ausgangsbasis 1.1.4, damals
`irrigation-card.js`). Die Karte heißt seit **3.0.0** `irrigation-control-card`;
siehe Abschnitt „Umbenennung zu 3.0.0“.

## Ergebnis

Drei reproduzierbare Probleme in der Bewässerungsbedienung wurden behoben. Es gab
keine bestätigte XSS-, Authentifizierungs- oder Credential-Lücke und keine offenen
bestätigten hohen oder kritischen Befunde im geprüften Umfang.

## Befunde (2.0.0)

| ID | Risiko | Schweregrad | Korrektur / Status |
|---|---|---|---|
| IRR-01 | Fehlt der dauerfähige Hydrawise-Statussensor, fiel ein Minutenstart still auf einen manuellen Schalter oder ein Ventil zurück. Die gewählte Dauer wurde nicht übermittelt, eine angezeigte 5-Minuten-Bewässerung konnte mit der Controller-Vorgabe laufen (HA-Core 2026.10.2: 15 Minuten beim manuellen Hydrawise-Schalter; unbegrenzte Laufzeit nicht nachgewiesen). | **Mittel** | Minutentasten nur mit dauerfähigem Hydrawise-Ziel; andere Starts ausdrücklich als **Controller-Vorgabe** gekennzeichnet. Positive Dauern ohne passendes Ziel, ungültige Werte und Offline-Starts werden abgewiesen. **Behoben.** |
| IRR-02 | Nach dem Start zeigte die Karte bei noch altem Zustand `off` nur gesperrte Starttasten und keine nutzbare Stopptaste. | Niedrig | Stopp bleibt während der Startbestätigung bedienbar, sofern ein Ziel existiert. Ein laufender Startauftrag wird vorher abgearbeitet; fehlende Ziele und doppelte Stoppaufträge lösen keinen ungezielten Dienstaufruf aus. **Behoben.** |
| IRR-03 | Nach 90 Sekunden wurde der Wartezustand auch bei unbeantwortetem Auftrag gelöscht; ein weiterer Start war möglich, obwohl der alte noch ausstand. | Niedrig | Unbeantwortete Aufträge bleiben gesperrt. Die Frist beginnt nach erfolgreicher Dienstannahme, die Antwort-Identität verhindert das Überschreiben neuerer Aufträge. **Behoben.** |
| IRR-H01 | Normale Objekte als Registry-/Aktionsindizes konnten bei absichtlich ungültigen YAML-Geräte-IDs wie `constructor` geerbte Eigenschaften auflösen. Kein privilegierter Angriff nachgewiesen. | Härtung | Prototyplose Indizes, reservierte IDs werden ignoriert. **Umgesetzt und getestet.** |
| IRR-INFO01 | Externe Bild-URLs übertragen Browser-Netzwerkmetadaten, HTTP bietet keinen Transportschutz. | Niedrig / Konfiguration | Lokale `/local/…`-Bilder empfohlen, keine Zugangsdaten in URLs. Attribute werden escaped; kein Schema- oder SVG-XSS-Exploit nachgewiesen. |

## Geprüfte Vertrauensgrenzen

- **DOM/HTML:** Titel, Namen, Entity-IDs, Zustände, Einheiten und Optionswerte werden über
  `_e()` maskiert. Ereignisse sind JavaScript-Listener, keine Inline-Handler.
- **URLs:** Die konfigurierbare Bild-URL landet nur in `img.src`, das Standardbild ist
  eingebettet. Keine dynamische Navigation oder Script-URL.
- **API:** Nur `hass.callService` über die authentifizierte HA-Sitzung. Kein `fetch`,
  kein eigener WebSocket-Client, kein gespeicherter Token oder Controller-Passwort.
  Rechte prüfen HA und die Geräteintegration serverseitig; UI-Sperren sind keine
  Autorisierungsgrenze.
- **Physische Wirkung:** Start/Stopp, Automatik und Pumpenschalter können reale Geräte
  schalten. Die Tests nutzen nur lokale Attrappen und gemockte Dienste.
- **Geheimnisse:** Keine hartcodierten Zugangsdaten im Laufzeitcode; kein Geheimnisscan
  über die gesamte Git-Historie.
- **Abhängigkeiten:** Keine Laufzeitabhängigkeiten oder mitgelieferten Fremdbibliotheken.
  Playwright 1.64.1 ist nur Testwerkzeug (`pip-audit` samt transitiver Pakete: 0 bekannte
  Befunde, Momentaufnahme).

## Restrisiken

- HA-, Controller- und Cloud-Zustände können verzögert, veraltet oder falsch sein. Eine
  Anzeige beweist keinen Wasserfluss bzw. geschlossenen Ventilzustand. Bei Cloud-Ausfall
  muss die Anlage direkt geprüft werden.
- Standardläufe folgen den Controller-Regeln; es gibt keinen zusätzlichen
  Browser-Abschalttimer. Die Pumpe bleibt unabhängig bedienbar, ohne eigene Verriegelung,
  Trockenlaufsicherung oder Überwachung. Automatik-Aus kann laut Hydrawise die Zone für
  365 Tage pausieren.
- Eine reale Prüfung mit HA, Hydrawise, HACS, Companion-App und Kiosk bleibt vor einem
  Release erforderlich.

## Umbenennung zu 3.0.0 (25.10.2026)

Der HACS-Review bestätigte eine Namenskollision mit `kratochj/hassio-irrigation-card`:
Beide Karten registrierten `irrigation-card`, die Ladereihenfolge konnte die falsche
Karte anzeigen oder die Registrierung der anderen abbrechen. Deshalb:

- Name **Irrigation Control Card**, Datei `irrigation-control-card.js`.
- Custom-Element / YAML: `irrigation-control-card` / `custom:irrigation-control-card`.
- Kein Alias für `irrigation-card`, da dieser die Kollision erhalten würde.
- Eine abgeschlossene Funktionsumgebung kapselt Konstanten und Klasse, damit fremde
  Karten und erneutes Laden keine globalen Deklarationen kollidieren lassen.

Das ist ein **Breaking Change**: Bestehende Karten brauchen den neuen YAML-Typ, die
Dashboard-Ressource die neue Datei. Die README beschreibt die Migration ohne
Deinstallation. Optionen, Entity-Zuordnungen, Steuerungsabläufe und das eingebettete
Bild blieben unverändert.

Prüfung: `node --check` bestanden, 21 Node-Tests und 46 Chromium-Prüfungen bestanden
(inklusive beider Ladereihenfolgen mit einer fremden `irrigation-card` und erneutem
Laden). Ein Vergleich des Laufzeitcodes gegen 2.0.0 zeigte nur Identitäts-/Versionsänderungen
und die Funktionskapselung.

Die HACS-Katalogaufnahme bleibt bis zum Merge von
[hacs/default #12053](https://github.com/hacs/default/pull/12053) ausstehend.
