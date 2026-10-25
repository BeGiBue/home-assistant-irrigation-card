# Irrigation Control Card v3.0.0

Diese Version bereitet die Umbenennung des Projekts vor. Sie wird per Pull Request geprüft;
Review, erfolgreiche CI und eine gesonderte Veröffentlichungsfreigabe bleiben erforderlich.

- Name in HACS und im Karteneditor: **Irrigation Control Card**.
- Neuer YAML-Typ: `custom:irrigation-control-card`.
- Neue JavaScript-Datei: `irrigation-control-card.js`.
- Repository und Projektlinks: <https://github.com/BeGiBue/home-assistant-irrigation-control-card>.
- Die Umbenennung verhindert die Kollision mit dem Custom-Element `irrigation-card` der
  bereits im HACS-Katalog enthaltenen Karte von `kratochj/hassio-irrigation-card`.
  Der alte Tag wird nicht als Alias registriert.

## Erforderliche Migration

Die Umbenennung ist ein **Breaking Change**. Die bestehende Installation und Dashboard-Karten
behalten; keine Entitäten neu anlegen:

1. Die neue Version in der bestehenden HACS-Installation laden und den HACS-Eintrag auf das
   umbenannte Repository prüfen.
2. Die zu diesem Projekt gehörende Dashboard-Ressource auf `irrigation-control-card.js`
   aktualisieren. Die von HACS bereitgestellte URL verwenden; bei gleichem Verzeichnis nur
   den Dateinamen ersetzen und vorhandene Cache-Parameter beibehalten.
3. In den bestehenden Karten dieses Projekts `type: custom:irrigation-card` durch
   `type: custom:irrigation-control-card` ersetzen. Alle anderen Einstellungen bleiben erhalten.
4. Alte Ressourcen dieses Projekts nicht zusätzlich laden. Die Ressource und Karten der
   anderen Irrigation Card von `kratochj` unverändert lassen.
5. Browser bzw. Companion-App vollständig neu laden und die Karten prüfen.

Die vollständige Anleitung einschließlich Repository-Umbenennung und manueller Installation
steht in der [README](README.md#bestehende-karten-auf-den-neuen-namen-umstellen).
Eine Major-Version bedeutet nicht automatisch Breaking Changes; in dieser Version ist die
Umbenennung von Kartentyp und Ressource ausdrücklich eine erforderliche Umstellung.

## HACS-Katalogstatus

Die Aufnahme in den offiziellen HACS-Katalog ist weiterhin ausstehend
([hacs/default #12053](https://github.com/hacs/default/pull/12053)). Bis zur tatsächlichen
Verfügbarkeit bleibt die Custom-Repository-Installation der aktuelle Weg. Nach der Aufnahme
ist die reguläre HACS-Suche der Standardweg. Erst dann den Custom-Repository-Eintrag entfernen,
**ohne die bestehende Installation zu deinstallieren**; Konfigurationen und Karten behalten.

Die Sicherheitskorrekturen aus 2.0.0 bleiben erhalten. Es werden keine neuen
Steuerungsfunktionen oder Entitätszuordnungen eingeführt.
