# Mein-Homeserver-Projekt
Privates Projekt zum Aufbau, Betrieb und zur Wartung eines eigenen Linux-Server - als Vorbereitung auf die Ausbildung zum Fachinformatiker für Systemintegration.

## Überblick
- Betriebssystem: Ubuntu Server (aktuelle LTS-Version)
- Zugriff: SSH von Windows-PC aus
- Webserver: nginx
- Cloud-Speicher: Nextcloud (Docker)
- Eigene Website: HTML/CSS, gehostet auf dem Server
- Backups: automatisiert per Cronjob (Minecraft-Server, Nextcloud, Home-Verzeichnis)

## Eigenständig aufgebaut (vor Beginn der KI-gestützten Weiterentwicklung)
- Ubuntu Server auf einem externen Rechner selbstständig installiert
- Minecraft-Server aufgesetzt und zum Laufen gebracht
- Nextcloud eingerichtet (inkl. Docker-Umgebung)

Diese Grundlagen bildeten den Ausgangspunkt für alle weiteren Projekte und Fehleranalysen (siehe unten).

## Aufbau
Server (Ubuntu)
 ├─ nginx  → Website (/var/www/...)
 ├─ Docker → Nextcloud (Datenbank + Dateien)
 ├─ Minecraft-Server
 └─ Cronjobs → automatische Backups (18:00 / 18:15 / 18:30 Uhr)

## Herausforderungen & Lösungen

### Hintergrundbild auf Website wurde nicht angezeigt
*Problem:* Bild erschien trotz korrektem CSS nicht.
*Analyse:* Mit Browser-Entwicklertools (Netzwerk-Tab) geprüft, welche Dateien geladen werden. Dabei entdeckt: Der Webserver lieferte eine ganz andere, alte Seite aus einem anderen Ordner aus.
*Lösung:* Backup der alten Seite erstellt, eigenes Projekt in den richtigen nginx-Ordner kopiert. Zusätzlich einen doppelten, überschreibenden CSS-Block entfernt.

### Zwei parallele Website-Ordner
Problem: Beim Einbau des Serverstatus-Moduls fiel mir auf, dass es zwei verschiedene Versionen der Website gab – eine mit den aktuellen Buttons "Meine Projekte" und "Mein Fuhrpark", die andere mit altem Dark-Mode-Umschalter ohne richtigen Hintergrund. Meine neue Funktion tauchte zunächst auf der falschen Seite auf.
Analysieren: Beim Vergleich wurde klar, dass es zwei getrennte Ordner auf dem Server gab – einen aktiven und einen alten Test-Ordner.
Lösung: /var/www/meine-seite/ wurde als die echte, aktuelle Website bestätigt. Alte Kopien wurden gelöscht, und das Skript schreibt jetzt korrekt in den richtigen Ordner.

### Monitoring-System eingerichtet (Uptime Kuma)
*Ziel:* Serverdienste automatisch überwachen, um Ausfälle frühzeitig zu erkennen.
*Umsetzung:* Uptime Kuma per Docker installiert und drei Monitore eingerichtet – für die eigene Website, Nextcloud und den Minecraft-Server. Zusätzlich eine Windows-Verknüpfung erstellt, um den Minecraft-Server künftig per Doppelklick starten zu können.

*Problem 1:* Der Nextcloud-Monitor zeigte "Inaktiv" an.
*Analyse:* Die URL enthielt automatisch "https" statt "http", weil das Feld das Präfix schon vorausgefüllt hatte.
*Lösung:* URL korrigiert – danach lief der Monitor fehlerfrei.

*Problem 2:* Die Windows-Verknüpfung öffnete beim Doppelklick nur eine Textdatei statt den Server zu starten.
*Analyse:* Windows blendet Dateiendungen standardmäßig aus – die Datei hieß in Wirklichkeit minecraft_starten.bat.txt, nicht minecraft_starten.bat.
*Lösung:* Im Explorer unter "Ansicht" die Dateinamenerweiterungen sichtbar gemacht und die Datei korrekt umbenannt. Danach funktionierte die Verknüpfung wie gewünscht.

### Serverstatus-Modul
Ziel: CPU-, RAM- und Speicherauslastung des Servers live auf der Website anzeigen.
Problem: Beim Installieren der Python-Bibliothek psutil (liest Systemwerte wie CPU/RAM aus) funktionierte die Installation über sudo nicht, da mir dafür die Admin-Rechte fehlten.
Lösung: Installation nur für den eigenen Benutzer durchgeführt (landet in ~/.local), ohne Eingriff ins Gesamtsystem. Damit hat die Installation von psutil funktioniert.
Das eigene Skript serverstatus.py liest CPU, RAM und Speicherplatz aus und schreibt die Werte in eine status.html, die per JavaScript in die Hauptseite eingebunden wird.

### Uptime-Kuma-Ausbau: Desktop-Verknüpfung, Discord-Benachrichtigungen, kurze Servernamen
*Ziel:* Das Monitoring-System bequemer nutzbar machen und bei Ausfällen automatisch benachrichtigt werden.
*Umsetzung:*
- Desktop-Verknüpfung erstellt, die das Uptime-Kuma-Dashboard per Doppelklick im Browser öffnet
- Eigenen Discord-Server erstellt und über einen Webhook mit Uptime Kuma verbunden, sodass bei Statusänderungen automatisch eine Nachricht in Discord ankommt
- Windows-hosts-Datei bearbeitet, damit kurze Namen (z. B. "monitoring") statt der vollen IP-Adresse im Browser genutzt werden können

*Problem:* Die hosts-Datei ließ sich zwar öffnen und bearbeiten, aber nicht speichern.
*Analyse:* Notepad lief trotz "Als Administrator ausführen" nicht wirklich mit Admin-Rechten.
*Lösung:* Notepad direkt aus einer bereits als Administrator laufenden PowerShell heraus gestartet – dadurch übernimmt es automatisch die nötigen Rechte. Danach ließ sich die Datei speichern, und die kurzen Namen funktionieren seitdem einwandfrei.

### Monitoring-Status auf der eigenen Website einbinden
*Ziel:* Den Live-Status des Servers direkt auf der eigenen Website sichtbar machen.
*Umsetzung:* Eine öffentliche Status-Seite in Uptime Kuma erstellt und eine neue Unterseite (uptime.html) auf der Website angelegt.

*Problem:* Die Status-Seite ließ sich nicht per iframe in die eigene Website einbetten.
*Analyse:* Uptime Kuma blockiert das Einbetten aus Sicherheitsgründen bewusst (X-Frame-Options-Header).
*Lösung:* Statt eines iframes einen direkten Button eingebaut, der die Status-Seite in einem neuen Tab öffnet. Funktioniert seitdem einwandfrei und ist über die Startseite verlinkt.

### Besucherzähler: Position & Zähl-Bug

Problem: Der Besucherzähler steht zunächst oben rechts in der Ecke, sollte aber unter meinen drei Buttons (Meine Projekte, Mein Fuhrpark, Server-Status). Außerdem ist mir aufgefallen, dass der Zähler an einem Tag von 2400 auf 3800 gesprungen war, obwohl ich die Website an dem Tag maximal 3-mal besucht hatte. Analysieren: Mit Claude Code habe ich den HTML-Code der Seite durchgesehen, um erst das bestehende Konzept zu verstehen, und den Zähler dann an die gewünschte Stelle verschoben. Für den Zähl-Sprung haben wir den Code der Flask-App untersucht: Bei jedem automatisierten Scan durch Uptime Kuma (mein Monitoring-Tool) wurde die Seite fälschlich als echter Besuch mitgezählt. Lösung: Ich habe einen separaten /health-Endpunkt in der Flask-App erstellt, der nur den Serverstatus prüft, ohne den Zähler zu erhöhen. Uptime Kuma fragt jetzt diesen Endpunkt ab statt der eigentlichen Seite – der Zähler zeigt seitdem nur noch echte Besuche.

### Chrome-Erweiterung für Claude Code einrichten

Problem: Ich wollte die Chrome-Erweiterung "Claude in Chrome" verbinden, damit Claude Code direkt im Browser navigieren kann. Im Menü sprang die Auswahl aber immer wieder zurück auf "Install Chrome extension", statt "Reconnect extension" auszuführen – das Problem hatte ich schon einmal vorher, ohne es zu lösen. Analysieren: Diesmal habe ich es nicht über die Menü-Navigation, sondern direkt über einen anderen Befehl versucht. Lösung: Mit dem passenden Befehl hat die Verbindung dann funktioniert – Claude Code zeigt jetzt "Chrome Browser Automatisierung ist bereit" an.

### Nextcloud-Passwort aus dem Code entfernt Problem:
In der docker-compose.yml-Datei für Nextcloud stand das Passwort für die Datenbank im Klartext direkt im Code. Das ist unsicher, weil jeder, der Zugriff auf die Datei hat, das Passwort einfach lesen könnte – zum Beispiel auch, wenn die Datei versehentlich auf GitHub landet.
Analysieren:
Ich habe die Datei durchsucht und festgestellt, dass das Passwort an mehreren Stellen im Klartext stand. Die Lösung dafür ist, Passwörter grundsätzlich nicht direkt im Code zu speichern, sondern in einer separaten Datei, die nicht mit hochgeladen wird.
Lösung:
Ich habe eine neue .env-Datei angelegt und das Passwort dort als Variable gespeichert. In der docker-compose.yml wird das Passwort jetzt nur noch über diese Variable eingebunden, steht also nicht mehr direkt im Code. Zusätzlich habe ich die .env-Datei in die .gitignore eingetragen, damit sie niemals versehentlich auf GitHub hochgeladen wird. Zur Sicherheit habe ich außerdem ein neues Passwort vergeben. Nach einem Neustart habe ich geprüft, dass Nextcloud weiterhin fehlerfrei funktioniert.

### Serverstatus-Verlauf als Diagramm Problem:
Bisher konnte ich den Serverstatus (CPU, RAM, Speicherplatz) nur als aktuellen Momentwert sehen. Ein Verlauf über die Zeit war nicht sichtbar, obwohl die Werte längst in der Datenbank gespeichert wurden.
Analysieren:
Ich habe überlegt, wie ich die gespeicherten Messwerte sichtbar machen kann. Die Lösung war, die Daten über einen eigenen API-Endpunkt als JSON bereitzustellen und daraus mit der JavaScript-Bibliothek Chart.js ein Liniendiagramm zu zeichnen.
Lösung:
Ich habe eine neue Seite erstellt, auf der die Werte für CPU, RAM und Speicherplatz als farbige Linien im Diagramm dargestellt werden, mit einer Legende, die zeigt, welche Farbe zu welchem Wert gehört. Auf der unteren Achse steht die Uhrzeit, auf der seitlichen Achse der Prozentwert. Zusätzlich habe ich drei Buttons eingebaut ("Letzte Stunde", "Letzter Tag", "Alle Daten"), mit denen man den angezeigten Zeitraum auswählen kann.

## Nächste Schritte
- z. B. Reverse Proxy einrichten
- z. B. Monitoring (Uptime Kuma) ergänzen
- z. B. Docker-Compose für alle Dienste vereinheitlichen
