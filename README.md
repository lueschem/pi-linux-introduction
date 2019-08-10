# Raspberry Pi und Linux Einführung

## Einleitung

In einem ersten Schritt richten wir uns eine zweckmässige Entwicklungsumgebung ein, so wie sie bei einem typischen
Embedded Projekt zum Einsatz kommt. Normalerweise wird nicht direkt auf dem **Zielsystem** entwickelt, sondern auf einem
viel leistungsfähigeren **Entwicklungsrechner**.

Der Entwicklungsrechner ist in unserem Fall mit Windows 10 ausgerüstet. Für Linux entwickelt man am bequemsten mit
Linux. Daher installieren wir auf dem Windows 10 mittels WSL (Windows Subsystem für Linux) ein Debian buster Linux.

Auf dem Raspberry Pi installieren wir das sehr ähnliche Raspbian buster.

Damit wir effizient arbeiten können, konfigurieren wir ssh, so dass wir vom WSL Debian buster bequem und ohne Passwort
ins Raspbian buster einloggen können.

Wir führen sämtliche Arbeiten auf der Kommandozeile durch, da dadurch besser erklärt werden kann, was "hinter der Bühne"
passiert.

Als erste Aufgaben schreiben wir ein "Hello World!" Programm in Python und in C++. Das entstandene Programm wollen wir
sowohl unter WSL als auch auf dem Raspberry Pi ausführen.

Damit wir unseren Quellcode gut aufbewahren können, verwenden wir git zur Versionierung des Quellcodes.

Zum Schluss installieren wir noch einen Webserver auf dem Raspberry Pi und stellen sicher, dass wir per Browser vom
Entwicklungsrechner darauf zugreifen können.

## Installation von Debian unter WSL

Debian buster installieren wir gemäss [dieser Anleitung](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

*myuser* ist der Benutzername.
Das Passwort bewahren wir an einem sicheren Ort auf.

Das WSL Debian buster System kann in der PowerShell mit folgendem Befehl erreicht werden:

```
debian
```

## Zugriff von Debian buster (WSL) auf Raspberry Pi

Nun verbinden wir das Raspberry Pi mit dem Kabelnetzwerk. Somit erhält das Raspberry Pi vom Router per DHCP eine IP
Adresse.

```
-------------------------            -----------------                    ---------------------------
| Entwicklungsrechner   |            | Router (DHCP) |      192.168.40.21 | Raspberry Pi            |
| Windows 10 (amd64)    |------------|               |--------------------| Raspbian Buster (armhf) |
| WSL mit Debian buster |            |               |                    | Benutzer: pi            |
|                       |            |               |                    | sshd für Fernzugriff    |
-------------------------            -----------------                    ---------------------------
```

Die IP Adresse des Raspberry Pi finden wir mit folgendem Befehl heraus:

Auf der Konsole des Raspberry Pi:

```
sudo /sbin/ifconfig eth0
```

In unserem Beispiel lautet die IP Adresse **192.168.40.21**. Sobald das Rasperry Pi die Adresse von einem anderen Router
bezieht, wird die IP Adresse eine andere sein.

Nun testen wir, ob wir vom Entwicklungsrechner das Zielsystem erreichen können:

```
sudo ping 192.168.40.21
```

Auf dem Raspberry Pi konfigurieren wir nun, dass der [Secure Shell Daemon](https://de.wikipedia.org/wiki/Secure_Shell)
automatisch startet:

```
sudo raspi-config
```

Unter "Interfacing Options" lässt sich der Secure Shell Daemon einschalten.
`sudo` wird dabei dem Befehl `raspi-config` vorangestellt, um die Zugriffsrechte auf `root` zu erhöhen. Als
"normaler" Benutzer `pi` dürfte man solche Änderungen nicht vornehmen.

Nun wechseln wir zurück zum Entwicklungsrechner.

Im Debian buster WSL System installieren wir die Secure Shell Client Anwendung:

```
sudo apt update
sudo apt install openssh-client
```

Die Anwendung `openssh-client` wird dabei vom vorkonfigurierten [APT Repository](https://wiki.debian.org/DebianRepository)
bezogen. APT stellt dabei sicher, dass nur vertrauenswürdige Software installiert wird.

Nun können wir vom Entwicklungsrechner ins Zielsystem einloggen:

```
ssh pi@192.168.40.21
```

Mittels folgendem Befehl können wir feststellen, dass wir tatsächlich auf dem Raspberry Pi gelandet sind:

```
cat /etc/os-release
```

Nun verlassen wir das Zielsystem wieder:

```
exit
```

Damit wir beim Zugriff auf das Zielsystem kein Passwort mehr eingeben müssen, generieren wir nun auf dem
Debian buster WSL System ein Schlüsselpaar:

```
ssh-keygen -t rsa -b 4096 -C "myuser@example.com"
```

Den öffentlichen Schlüssel kopieren wir nun auf das Zielsystem:

```
ssh-copy-id pi@192.168.40.21
```

Ab jetzt können wir nun per `ssh pi@192.168.40.21` ohne Passwort ins Zielsystem einloggen.

## Erstes Projekt mit git Quellcodeverwaltung

Auf dem Debian buster WSL System richten wir uns nun einen Arbeitsbereich ein und darin verwalten wir
ein erstes Projekt mit der Quellcodeverwaltung [git](https://git-scm.com/).

Dazu erstellen wir passende Verzeichnisse:

```
mkdir -p ~/workspace/myfirstproject
cd ~/workspace/myfirstproject
```

Nun installieren wir git:

```
sudo apt install git
```

Und initialisieren die Quellcodeverwaltung für unser Projekt:

```
git init
```

## Hello World mit Python3

Auf dem Debian buster WSL System installieren wir nun den Python Interpreter:

```
sudo apt install python3
```

Der Python Interpreter lässt sich im interaktiven Modus bedienen:

```
python3
>>> print("Hello World!")
Hello World!
```

Mit *Ctrl+d* können wir den Interpreter wieder verlassen.

Nun wollen wir aber ein richtiges, ausführbares Python Skript schreiben.
Dazu installieren wir den Text Editor `vi`:

```
sudo apt install vim
```

Mit folgendem Befehl starten wir den Editor und eröffnen gleich unsere Skriptdatei:

```
vi hellopy
```

Mit der Taste *i* wechseln wir in den Schreibmodus und geben folgenden Python Code ein:

```
#!/usr/bin/python3

def main():
    print("Hello World!")

if __name__=="__main__":
    main()
```

Mit *Esc* wechseln wir in den Befehlsmodus und geben den Befehl `:wq` ein um die Datei zu
schreiben und `vi` zu verlassen.

Nun müssen wir unserem Skript noch die Berechtigung geben, ausgeführt zu werden:

```
chmod +x hellopy
```

Nun können wir das Skript ausführen:

```
./hellopy
```

Natürlich wollen wir auch sicherstellen, dass sich das Skript auf dem Zielsystem ausführen lässt.
Dazu kopieren wir es auf das Zielsystem:

```
scp hellopy pi@192.168.40.21:
```

Und wechseln ins Zielsystem, um das Skript auszuführen:

```
ssh pi@192.168.40.21
./hellopy
```

Mit `exit` verlassen wir das Zielsystem wieder.

Nach diesem erfolgreichen Test wollen wir unser Skript mit der Quellcodeverwaltung git verwalten.

Dazu teilen wir git einmalig mit, wer für die Änderungen verantwortlich ist:

```
git config --global user.name "Heinz Muster"
git config --global user.email "myuser@example.com"
```

Mit `git status` können wir verifizieren, welche Dateien verwaltet werden sollten.

Wir stellen nun unser Skript unter git Verwaltung:

```
git add hellopy
```

Und mittels *commit* speichern wir den aktuellen Stand von unserem Skript:

```
git commit -m "Added Python3 hello world script."
```

## Hello World mit C++

Nun wollen wir unser "Hello World" Experiment mit C++ wiederholen.

Mit `vi hello.cpp` schreiben wir folgenden Quellcode:

```
#include <iostream>

int main()
{
    std::cout << "Hello World!" << std::endl
    return 0;
}
```

Mit *Esc* und `:wq` verlassen wir den Editor wieder.

Der Quellcode muss nun noch in Maschinensprache übersetzt werden.

Dazu installieren wir den gcc Compiler:

```
sudo apt install build-essential
```

Und übersetzen unser Programm:

```
g++ hello.cpp -o hello
```

Und führen es aus:

```
./hello
```

Nach diesem Erfolgserlebnis kopieren wir die ausführbare Datei auf das Zielsystem und versuchen sie
dort auszuführen:

```
scp hello pi@192.168.40.21:
ssh pi@192.168.40.21
./hello
```

Leider schlägt dieser Versuch fehl, da unsere Ausführbare Datei für ein amd64 Zielsystem kompiliert wurde.

Mit `exit` verlassen wir unser Zielsystem wieder und kompilieren eine passende ausführbare Datei.

Dazu benötigen wir einen Crosscompiler:

```
sudo apt install crossbuild-essential-armhf
```

Damit lässt sich nun eine passende ausführbare Datei für unser Zielsystem erstellen:

```
arm-linux-gnueabihf-g++ hello.cpp -o hello2
```

Diese Datei kopieren wir nun aufs Zielsystem und führen sie dort erfolgreich aus:

```
scp hello2 pi@192.168.40.21:
ssh pi@192.168.40.21
./hello2
```

Mit `exit` verlassen wir das Zielsystem und fügen unseren neuen Quellcode dem git Repository hinzu.

```
git add hello.cpp
git commit -m "Added C++ hello world source code."
```

## Web Server

Zum Schluss starten wir einen Web Server auf dem Zielsystem. Dazu loggen wir ins Zielsystem ein
und installieren `nginx`:

```
ssh pi@192.168.40.21
sudo apt install nginx
```

Mit einem Web Browser auf unserem Windows 10 PC können wir nun verifizieren, dass der Web Server
tatsächlich erreichbar ist.

Dazu geben wir in der Adresszeile folgenden Text ein: `http://192.168.40.21`

