![Artikel-Leitbild zeigt ein Smartphone, das mit vielen Dingen kommuniziert](feinstaub-iot-heropic.jpg)
_Bildquelle: radoma/stock.adobe.com_

# Bauen Sie Ihr eigenes Internet-of-Things-Endgerät zur Messung der Feinstaub-Belastung

**Das "Internet of Things" (IoT) ist ein Netzwerk von smarten Geräten, die durch Sensoren und durch ihre Vernetzung den Alltag und das Leben der Menschen verbessern können. Insbesondere beim zukünftigen Ausbau von Städten zu Smarter Cities wird IoT eine besondere Rolle zugesprochen. In diesem Workshop bauen wir ein Werkzeug für die Smart-City-Umgebung: ein IoT-Gerät für die Messung der Feinstaub-Belastung der Luft auf Basis eines Raspberry Pi!**

Dass IoT tatsächlich das Leben von Menschen verbessern kann, zeigt der bekannte Programmierer und Blogger _Scott Hanselman_ eindrucksvoll in diesem Videomitschnitt der _Microsoft Connect() Conference_ von 2015 (Englisch):

> **📺 Video-Beitrag**
>
> **Thema:** Scott Hanselman's best demo! IoT, Azure, Machine Learning  
> **Link:** [▶ Video auf YouTube öffnen](https://www.youtube.com/watch?v=u5oTz1e5qqE)
>
> _Hinweis: Durch Klicken auf den Link verlassen Sie diesen Artikel und werden zu YouTube weitergeleitet. Dort gelten die Datenschutzbestimmungen von Google._

IoT kann also auch die Gesundheit und damit das Leben und den Alltag unmittelbar verbessern. Auch das IoT-System, das wir in diesem Workshop bauen, kann einen positiven Effekt auf die Lebensqualität in Städten bewirken, denn Feinstaub-Messgeräte haben einen Aufklärungs-Effekt, der zu mehr Partizipation und letztendlich Optimierung durch die Bürger führen kann. Bürger-Projekte zur Überwachung der Luftqualität sind daher wichtig und verzeichnen auch tatsächlich einen immer grösseren Zulauf. Lassen Sie uns also starten!

Per Definition wird für ein IoT-Gerät ein Computer benötigt. Dies kann einfach nur ein Mikrocontroller sein, oder aber auch ein ganzer Ein-Platinen-PC im Kleinstformat, wie es der Raspberry Pi ist. Der Raspberry Pi wird aufgrund seiner kompakten Form, seines sparsamen Stromverbrauchs und seiner offenen Architektur häufig in IoT-Umgebungen verwendet. Er erweist sich immer mehr als dezentrales Universal-Werkzeug für Smart-City-Anwendungen.

Für diesen Workshop wird vorausgesetzt, dass Sie bereits über einen lauffähigen Raspberry Pi verfügen und Grundkenntnisse im Umgang damit haben. Ich habe für dieses Experiment einen Raspberry Pi 2 Model B verwendet, aber ein Raspberry Pi 3 oder 4 tut es sicherlich auch. Zum Messen von Feinstaub benötigen wir zusätzlich zum Raspberry Pi noch das folgende Sensorsystem:

* Feinstaubsensor "nova PM sensor" Typ SDS011, 17 CHF

Der Sensor dieses Systems misst Feinstaub in den Korngrössen 2.5 μm (auch "PM2.5", wobei PM für "Particulate Matter" steht) und 10 μm (auch "PM10"). Dies sind die typischen Korngrössen, die bei Feinstaub gemessen werden. Ausserdem wurde die Messqualität des Sensors durch [influenceair](https://influencair.be/accuracy-of-the-sds011/) und durch [hackAir](https://www.hackair.eu/how-accurate-are-the-hackair-sensors/) in einem Vergleich mit höherwertigen Sensoren untersucht. Die Ergebnisse sind durchaus positiv zu bewerten.

Das Sensorsystem liefert das Messresultat in der Einheit vom zehnfachen Mikrogramm pro Kubikmeter (μg/m^3). Wenn also der Sensor zum Korndurchmesser 2.5 μm (oder "2.5mym" geschrieben) einen Wert von 32 misst, dann bedeutet das, dass sich in einem Kubikmeter Raum insgesamt 3.2 Mikrogramm Feinstaub der Korngrösse 2.5 Mikrometer befinden.

Zusätzlich zum Raspberry und zum Sensor empfiehlt es sich, noch eine Powerbank zu verwenden, um die Messungen auch mobil und in entlegenen Gebieten durchführen zu können. Ich habe die folgende Powerbank verwendet und kann diese nur empfehlen:

* Optional: Powerbank "Innoo Tech Portable Solar Charger" mit 10000 mAh, 50 CHF

Die Verkabelung ist schnell und einfach gemacht. Der nova-Sensor kommt mit einem USB-Stecker, den Sie einfach an einem der USB-Ports des Raspberry anschliessen können, und das wäre es:

![Verkabelung des Raspberry Pi mit dem Feinstaub-Sensor](raspi_nova_cable.jpg)

In dem obigen Bild hat der Raspberry noch zusätzlich einen WiFi-Dongle und ein USB-Kabel zur Stromversorgung (z.B. per Powerbank) eingesteckt. Der WiFi-Dongle ist praktisch, um den Raspberry auch mal per Remote-Zugriff bedienen zu können, für unseren Test-Aufbau spielt dieses "Gadget" jedoch gar keine Rolle.

Der nova-Sensor benötigt keinen separaten Stromanschluss, sondern bezieht seinen Strom vollständig über das USB-Kabel vom Raspberry. Sobald der Raspberry also an den Strom angeschlossen wird, wird auch der Feinstaub-Sensor aktiviert und beginnt mit den Messungen. Allerdings müssen wir irgendwie an die Messwerte heran kommen. Und auch hier hilft uns der Raspberry weiter. Allerdings kommen wir nicht drum rum, zu programmieren, um an diese Daten heran zu kommen.

Zur Programmierung verwenden wir die "Standard-Programmiersprache" des Raspberry Pi, die ihm auch seine Namensendung verleiht, es ist _Python_. Die Programmierung in Python ist auf dem Raspberry besonders praktisch, da sowohl ein Python-Interpreter als auch die Python-Entwicklungsumgebung _Thonny Python IDE_ standardmässig vorinstalliert sind.

Bevor wir jedoch mit der Programmierung beginnen können, sollten wir noch eine Bibliothek installieren, die uns den Lese-Zugriff auf die USB-Schnittstelle aus Python heraus ermöglicht. Diese Bibliothek heisst _pyserial_. Dazu öffnen wir die Befehlszeile (_Terminal_) im Raspberry Pi und setzen folgenden Befehl ab:

```bash
pip3 install pyserial
```

Danach öffnen wir auf dem Raspberry das Programm Thonny Python IDE. Dieses sollte vorinstalliert sein. Sie finden es im Hauptmenü (das Himbeer-Symbol oben links) im folgenden Menüast:

_Himbeer-Symbol => Entwicklung => Thonny Python IDE_

Es sollte sich sodann die folgende Entwicklungsumgebung (IDE) öffnen:

![Thonny Python IDE](raspi_thonny_start.png)

Die IDE ist dreigeteilt: ein Fenster für den Quelltext (links oben), ein Fenster für die _Variablen-Übersicht_ (rechts) und ein Fenster für die _Konsolen-Ausgaben_ (_Shell_ links unten). Unten links in der Shell sollte mindestens "Python 3..." aufgeführt sein, denn diese Version benötigen wir für einige der folgenden Funktionsaufrufe.

Kommen wir nun zum Kernstück unseres Aufbaus, dem Python-Skript. Dies ist das gesamte Python-Skript, das wir für die Messungen benötigen:

```python
import serial, time
import datetime

ser = serial.Serial('/dev/ttyUSB0')

while True:
    data = []
    for index in range(0,10):
        datum = ser.read();
        data.append(datum)
        
    zweikommafuenf = int.from_bytes(b''.join(data[2:4]), byteorder='little') / 10 
    zehn = int.from_bytes(b''.join(data[4:6]), byteorder='little') / 10
    
    messwerteDatei = open("/home/pi/Dokumente/develop/python/messwerte.csv", "a")
    messwerteDatei.write(str(datetime.datetime.now()) + ";" + \
        str(zweikommafuenf) + ";" + str(zehn) + "\r\n")
    messwerteDatei.close()
    
    time.sleep(10)
```

Tippen Sie das einfach in den Quelltext-Bereich oben links in der Thonny Python IDE ein. Speichern Sie anschliessend das Skript an einem beliebigen Ort auf den Raspberry z.B. unter dem Dateinamen _feinstaub.py_. Achten Sie darauf, die Dateiendung "py" zu verwenden. Ich habe das Skript z.B. in diesem Ordner gespeichert:

```bash
/home/pi/Dokumente/develop/python/feinstaub.py
```

Was passiert genau in dem Skript? Zunächst besorgen wir uns den Zugriff auf die USB-Schnittstelle und damit auf die Daten des Feinstaub-Sensors ("`ser = serial.Serial('/dev/ttyUSB0')`"). Anschliessend führen wir die Messungen in einer Endlos-Schleife durch ("`while True:`"). Innerhalb der Endlos-Schleife werden zunächst die Bytes von der USB-Schnittstelle ausgelesen, immer zehn Bytes pro Schleifendurchgang. Die Messwerte selbst stehen in dem Byte-Array an 3. und 4. Stelle (`[2:4]`) für den Messwert der Korngrösse _2.5 μm_ und an der 5. und 6. Stelle (`[4:6]`) für den Messwert der Korngrösse _10 μm_. Jeweils zwei Bytes werden zusammengefasst zu einem Wert vom Datentyp Integer (Ganzzahl) der Länge _16 bit_.

Der vom Sensor zurückgelieferte Messwert ist in der "unschönen" Einheit vom Zehnfachen eines Mikrogramms. Um also auf die Einheit Mikrogramm zu kommen, dividieren wir den gelieferten Wert durch zehn.

Anschliessend schreibt das Skript die gelesenen Werte in eine neue Zeile einer CSV-Datei. Im Beispiel hat die Datei den Namen messwerte.csv. Achten Sie bitte hier darauf, einen Pfad zu einem Ordner zu verwenden, der bereits existiert oder legen Sie den Ordner entsprechend an. Nach dem Schreiben einer Messung in eine Zeile macht das Programm eine Pause von 10 Sekunden bis zur nächsten Messung (`time.sleep(10)`).

Sie können nun mit der Messung beginnen, indem Sie in der Thonny Python IDE den Play-Button oben in der Werkzeug-Leiste klicken. Die Messwerte werden in die Datei messwerte.csv auf dem Raspberry Pi zusammen mit Datum und Uhrzeit gespeichert. Alle Werte werden durch Semikola getrennt. Das Ergebnis einer Messung kann dann beispielsweise so aussehen:

```csv
2020-11-08 22:50:38.196183;3.2;4.8
```
Ganz links sind Datum und Uhrzeit der Messung, mittig (nach dem Semikolon) die _PM2.5_-Menge, rechts die _PM10_-Menge.

Dies sind typische Messwerte für den Schweizer Ort Steckborn. Dieser hat also relativ niedrige Feinstaub-Werte. Zum Vergleich: In der nahegelegenen Stadt Singen habe ich bei _PM10_ an einem typischen Samstag einen Wert von _10 μg/m^3_ gemessen. In stark Feinstaub-verschmutzten Gebieten werden auch mal über _400 μg/m^3_ erreicht (mehr dazu ganz unten in diesem Artikel).

Das Python-Skript kann nicht nur aus der IDE, sondern auch von der Konsole aus mit folgendem Befehl gestartet werden:

```bash
python3 /home/pi/Dokumente/develop/python/feinstaub.py
```

Um das Sensor-System spontan einsetzen zu können, ohne den Raspberry an einen Bildschirm anschliessen zu müssen, empfiehlt es sich, die Messung automatisch beim Start des Betriebssystems mit starten zu lassen. Hierzu wird ein Startup-Skript benötigt, das wiederum das Python-Skript aufruft. Um ein Startup-Skript zu erzeugen, geben wir folgendes in der Konsole ein:

```bash
crontab -e
```

Wählen Sie aus der nun folgenden Liste Ihren bevorzugten Text-Editor aus. Für Einsteiger am einfachsten zu bedienen ist der Nano-Editor. In der Liste im folgenden Screenshot ist es die zweite Option, weshalb wir hier "2" eintippen und mit _[ENTER]_ bestätigen:

![Aufruf crontab auf der Konsole](aufruf_crontab_konsole.png)

Daraufhin öffnet sich der Text-Editor Nano mit der Konfigurationsdatei von _crontab_. crontab ist Task-Scheduler, mit dem wir automatisiert Prozesse starten lassen können. Fügen Sie in der erste Zeile den bereits oben erwähnten Befehl zum Starten der Messung hinzu, mit dem Hinweis, dass das Skript bei einem Reboot ("`@reboot`") starten soll:

```bash
@reboot python3 /home/pi/Dokumente/develop/python/feinstaub.py &
```

Das "`&`" am Ende stellt sicher, dass nach dem Absetzen des Befehls die aktuelle Zeile verlassen wird. Sichern Sie die Änderung mit _Ctrl+X_, "j", _[ENTER]_. Nach einem Neustart des Systems startet die Messung nun automatisch. Ab jetzt startet die Messung immer automatisch, sobald der Raspberry Pi gestartet wird.

Und das wäre es im Grunde, ihr IoT-Messsystem ist nun abgeschlossen. Um das System praktischer transportieren zu können, bietet es sich noch an, eine Frischhaltedose einzusetzen und mit schönen Stickern zu bekleben, wie in der folgenden Abbildung. :)

![Raspberry Pi mit Feinstaub-Sensor im offenen Behälter](raspi_feinstaub_offener_behaelter.jpg)

Als kleines Goodie zum Finale, können wir unsere Messdaten noch hübsch in einem Diagramm visualisieren. Dank des CSV-Formats ist das mit wenigen Schritten möglich z.B. mit Microsoft Excel oder mit LibreOffice Calc. Da wir uns mit dem Raspi bereits in der Open-Source-Welt bewegen, bleibe ich im folgenden Beispiel dabei und verwende LibreOffice Calc (mit der Benutzeroberfläche "In Registern"). Hier finden Sie unter

_=> Daten => Verknüpfung zu externen Daten..._

![Datenverknüpfung im LibreOffice Calc](calc_datenverknuepfung.png)

die Möglichkeit, die CSV-Datei einzulesen. Wählen Sie darauf folgenden Dialog-Fenster als "Trennoption" die Option "Getrennt" und dann das "Semikolon":

![LibreOffice Calc Textimport](calc_textimport.png)

Als Ergebnis sollten die Messungen in Calc öffnen. Fügen Sie noch eine leere oberste Zeile ein und benennen Sie die Spalten z.B. "timestamp", "2.5mym" und "10mym" (in dieser Reihenfolge). Wählen Sie anschliessend nur die Inhalte der Spalten "timestamp" und "10mym":

![LibreOffice Calc Spaltenauswahl](calc_auswahl_spalten.png)

Nun ist es soweit, wir können ein Diagramm erstellen. Wählen Sie hierzu in Calc den Menüpunkt:

_=> Einfügen => Diagramm..._

Im Anschliessenden Dialogfenster wählen Sie:

_=> Liniendiagramm => Nur Linien => Fertigstellen_

Und damit ist auch das erledigt. In der Calc-Tabelle sollte nun ein Messwerte-Diagramm ähnlich diesem hier eingeblendet sein:

![Messwerte-Diagramm in LibreOffice Calc](messwerte_diagramm.png)

Das Diagramm zeigt die Entwicklung der gemessenen Feinstaub-Werte der Grösse 10 μm über einen Zeitraum von 6 min. Diese schwanken im abgebildeten Beispiel in einem Bereich zwischen _2.5_ und _2.9 μg/m^3_. Alle _10 Sekunden_ liegt ein Datenpunkt vor.

Jetzt stellt sich noch die Frage, wie lange die Stromversorgung dieses Messaufbaus mit der oben angegebenen _10000-mAh_-Powerbank hält. Ich habe das in einem Test-Aufbau geprüft und das System einfach mal im Zimmer bis zum bitteren Ende laufen gelassen. Das Ergebnis: Bei Zimmertemperatur und einem Messintervall von _10 Minuten_ (=_600 Sekunden_):

```python
time.sleep(600)
```

kann dieses System über einen Zeitraum von ca. _12 Stunden_ Messdaten liefern.

Solch ein Projekt ist immer eine gute Gelegenheit um auf untragbare gesundheitliche Zustände im Zusammenhang mit Feinstaub-Belastungen hinzuweisen. So gibt es Städte, die in Arealen, in denen Menschen leben müssen, Feinstaub-Werte von bis zu _417 μg/m^3_ bei _PM 2.5_ erreichen. Dies wird im folgenden Video besonders eindrücklich thematisiert:

> **📺 Video-Beitrag**
>
> **Thema:** Most Polluted Major City On Earth  
> **[▶ Video auf YouTube öffnen](https://www.youtube.com/watch?v=mNZIdHhdQs8)**
>
> _Hinweis: Durch Klicken auf den Link verlassen Sie diesen Artikel und werden zu YouTube weitergeleitet. Dort gelten die Datenschutzbestimmungen von Google._

---

> **💾 Artikel als PDF**
>
> [▶ Artikel als PDF herunterladen](https://github.com/edgarbutwilowski/technik-und-teilhabe/blob/main/beitr%C3%A4ge/2020-12-feinstaub-iot/2020-12-feinstaub-iot.pdf)
> -

---

> **💬 Gemeinsam weiterdenken**
>
> Technik und Wissenschaft leben vom Austausch. Haben Sie Fragen zum Aufbau, eigene Erfahrungen mit Feinstaub-Messungen oder Anregungen zu den Daten? 
>
> Ich freue mich über eine sachliche und offene Diskussion zu diesem Beitrag auf Mastodon. Lassen Sie uns dort ins Gespräch kommen:
>
> [▶ Zum Diskussionsfaden auf swiss.social](https://swiss.social/@edgar_butwilowski/116167025003927679)
> -
