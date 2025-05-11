# Family2 - HackMyVM (Medium)

![Family2.png](Family2.png)

## Übersicht

*   **VM:** Family2
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Family2)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 4. November 2023
*   **Original-Writeup:** https://alientec1908.github.io/Family2_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war die Kompromittierung der virtuellen Maschine "Family2" auf der HackMyVM-Plattform. Die Enumeration mit `enum4linux` deckte die Benutzernamen `dad`, `mum` und `baby` auf. Der initiale Zugriff erfolgte per SSH als Benutzer `baby` (der Weg zur Erlangung des SSH-Schlüssels wurde im Writeup nicht detailliert beschrieben). Die erste Rechteausweitung zu `mum` gelang durch eine `sudo`-Regel, die `baby` erlaubte, `soelim` als `mum` auszuführen, womit der private SSH-Schlüssel von `mum` ausgelesen wurde. Die Eskalation zu `dad` erfolgte, nachdem das Passwort von `mum` in einer Umgebungsvariable gefunden wurde, was die Nutzung einer `sudo`-Regel (`(dad) ALL`) ermöglichte, um eine Shell als `dad` zu starten. Schließlich wurden Root-Rechte durch PATH-Hijacking erlangt: Ein SUID-Programm (`/opt/clock`), das den `date`-Befehl ohne absoluten Pfad aufrief, wurde ausgenutzt, indem eine bösartige `date`-Datei im Suchpfad platziert wurde, die eine Root-Shell startete.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `enum4linux`
*   `chmod`
*   `ssh`
*   `sudo`
*   `cat`
*   `grep`
*   `soelim`
*   `printenv`
*   `bash`
*   `find`
*   `echo`
*   `export`
*   `ls`, `cd`
*   Standard Linux-Befehle

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Family2" gliederte sich in folgende Phasen:

1.  **Reconnaissance (enum4linux):**
    *   (Annahme: IP 192.168.2.125 war bekannt).
    *   `enum4linux -a 192.168.2.125` wurde verwendet, um die lokalen Unix-Benutzer `dad`, `mum` und `baby` zu identifizieren.

2.  **Initial Access (SSH als `baby`):**
    *   (Annahme: Ein privater SSH-Schlüssel für `baby` wurde zuvor erlangt).
    *   Erfolgreicher SSH-Login als `baby` mit dem privaten Schlüssel.

3.  **Privilege Escalation (von `baby` zu `mum` via `sudo soelim`):**
    *   `sudo -l` für `baby` zeigte: `(mum) NOPASSWD: /usr/bin/soelim`.
    *   Der Befehl `sudo -u mum soelim /home/mum/.ssh/id_rsa` wurde genutzt, um den privaten SSH-Schlüssel von `mum` auszulesen.

4.  **Privilege Escalation (von `mum` zu `dad` via `sudo bash`):**
    *   Login als `mum` mit ihrem extrahierten SSH-Schlüssel.
    *   `sudo -l` für `mum` zeigte u.a.: `(dad) ALL`.
    *   Der Befehl `printenv` offenbarte das Passwort von `mum` (`LA0172`) in einer Umgebungsvariable (`passwd=LA0172`).
    *   Mit `sudo -u dad bash` und dem Passwort von `mum` wurde eine Shell als `dad` erlangt. Die User-Flag wurde gelesen.

5.  **Privilege Escalation (von `dad` zu `root` via SUID Binary & PATH Hijacking):**
    *   Die Suche nach SUID-Dateien (`find / -type f -perm -4000`) fand das benutzerdefinierte SUID-Programm `/opt/clock`, das `root` gehörte.
    *   Es wurde angenommen, dass `/opt/clock` intern den Befehl `date` ohne absoluten Pfad aufruft.
    *   In `/tmp` wurde eine ausführbare Datei namens `date` erstellt, die `/bin/bash` enthielt.
    *   Die `PATH`-Umgebungsvariable wurde mit `export PATH=/tmp:$PATH` manipuliert.
    *   Beim Ausführen von `/opt/clock` wurde die präparierte `date`-Datei im `/tmp`-Verzeichnis gefunden und als `root` ausgeführt, was zu einer Root-Shell führte. Die Root-Flag wurde gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Benutzerenumeration über RPC:** `enum4linux` konnte Systembenutzer über RPC-Abfragen aufdecken.
*   **Unsichere `sudo`-Konfigurationen:**
    *   `baby` durfte `soelim` als `mum` ausführen, was das Lesen von Dateien als `mum` ermöglichte (hier: SSH-Schlüssel).
    *   `mum` durfte jeden Befehl als `dad` ausführen (passwortgeschützt, aber Passwort war exponiert).
*   **Passwort in Umgebungsvariable:** Das Passwort des Benutzers `mum` war in einer ihrer Umgebungsvariablen gespeichert und für sie lesbar.
*   **SUID Binary mit unsicherem Aufruf externer Befehle (PATH Hijacking):** Ein SUID-Programm (`/opt/clock`) rief einen externen Befehl (`date`) ohne Angabe des absoluten Pfades auf. Dies ermöglichte es, die `PATH`-Variable zu manipulieren und eine eigene, bösartige Version des `date`-Befehls auszuführen, die dann mit Root-Rechten lief.

## Flags

*   **User Flag (`/home/dad/user.txt`):** `802809d489f8684f00fd6b8970724283`
*   **Root Flag (`/root/root.txt`):** `7177e6f98837966e1ce5d93c59ba0453`

## Tags

`HackMyVM`, `Family2`, `Medium`, `Enum4linux`, `SudoSoelim`, `PasswordInEnvironment`, `SudoBash`, `SUID`, `PATHHijacking`, `Linux`, `Privilege Escalation`
