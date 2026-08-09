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

## Nächste Schritte
- z. B. Reverse Proxy einrichten
- z. B. Monitoring (Uptime Kuma) ergänzen
- z. B. Docker-Compose für alle Dienste vereinheitlichen
