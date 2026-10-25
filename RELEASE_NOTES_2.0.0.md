# Irrigation Card v2.0.0

- Version 2.0.0 zur Weiterentwicklung und Vorbereitung des Wechsels in den offiziellen HACS-Katalog; die Aufnahme ist noch ausstehend ([hacs/default #12053](https://github.com/hacs/default/pull/12053)).
- Reguläre Installation über die HACS-Suche als Standardweg nach der Aufnahme dokumentiert. Bis zur tatsächlichen Katalogverfügbarkeit bleibt das benutzerdefinierte Repository der aktuelle Installationsweg.
- Migration bestehender Custom Repositories nach der Aufnahme beschrieben: Katalogverfügbarkeit prüfen, nur den benutzerdefinierten Eintrag entfernen, die Installation nicht deinstallieren und Dashboard-Konfigurationen sowie Ressourcen behalten.
- Die Major-Version bedeutet nicht automatisch Breaking Changes; Kartentyp und vorhandene Konfigurationsschlüssel bleiben erhalten. Sie bestätigt keine bereits erfolgte Katalogaufnahme.
- README auf Funktionen, Konfiguration, Bedienung, technische Voraussetzungen und Grenzen ausgerichtet; unnötige Erklärungen zu optischer Anordnung entfernt.
- Sicherheitskorrektur: Minutentasten nur mit dauerfähigem Hydrawise-Ziel; Standardläufe ohne solche Dauerübergabe sind ausdrücklich als „Controller-Vorgabe“ gekennzeichnet. Ungültige Dauern und stille Fallbacks von Minutenstarts werden zurückgewiesen.
- Sicherheitskorrektur: Stopp bleibt während der Startbestätigung bedienbar, sofern ein gültiges Ziel existiert; Aufträge werden geordnet und Wartezustände erst nach Dienstannahme bestätigt.
- Sicherheitskorrektur: Ein noch unbeantworteter Start-/Stoppauftrag bleibt auch nach 90 Sekunden gesperrt und gibt keinen konkurrierenden Start frei.
- Registry-/Aktionsindizes gegen geerbte Objektschlüssel gehärtet, Dokumentationslink korrigiert und Security-/Browser-Regressionstests mit gemockten Diensten ergänzt.
- Security-Review mit Datei-/Zeilenangaben, Korrekturen und Prüfgrenzen in `SECURITY_REVIEW.md` dokumentiert.
