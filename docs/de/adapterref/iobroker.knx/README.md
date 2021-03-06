---
translatedFrom: en
translatedWarning: Wenn Sie dieses Dokument bearbeiten möchten, löschen Sie bitte das Feld "translationsFrom". Andernfalls wird dieses Dokument automatisch erneut übersetzt
editLink: https://github.com/ioBroker/ioBroker.docs/edit/master/docs/de/adapterref/iobroker.knx/README.md
title: ioBroker.knx
hash: dd89TC8+mUVZyhj0EwZ7Du9fDTn0IgeGmiVSl27FBOc=
---
![Logo](../../../en/adapterref/iobroker.knx/admin/knx.png)

![NPM-Version](http://img.shields.io/npm/v/iobroker.knx.svg)
![Downloads](https://img.shields.io/npm/dm/iobroker.knx.svg)
![NPM](https://nodei.co/npm/iobroker.knx.png?downloads=true)

# IoBroker.knx
=================

## Beschreibung
de: Mit diesem Adapter können knxproj-Dateien aus der ETS importiert werden. Es generiert die Übersetzung zwischen KNX-Gruppenadressen und ioBroker und platziert die Geräte in Räumen (insb. Für MobileUI).

Es kann an Standard-KNX / LAN-Gateways angeschlossen werden.

Bevor Sie beginnen: Jeder DPT von com.Objects sollte in Ihrem ETS-Projekt eingestellt sein. Jedes Gerät sollte in Ihrer Anlagenstruktur sortiert sein.

## Eigenschaften:
* Importieren der knxproj-Datei
* Erzeugung einer ETS-ähnlichen Objektstruktur
* Finden und Kombinieren von Act-Channel und State-Channel (heuristisch)
* Aktualisierung aller Status beim Start
* Senden eines READ an den KNX-Bus, während auf das Statusobjekt geschrieben wird
* Sortieren von Kanälen zu Räumen

## Adapterkonfiguration
Öffnen Sie nach der Installation dieses Adapters die Adapterkonfiguration. Ergänze:

### KNX Gateway IP
<IP Ihres KNX / Lan GW> im IPv4-Format

### Hafen
Dies ist normalerweise Port 3671

### Phys. EIB-Adresse
geben Sie frei phys. Adresse entsprechend Ihrer KNX-Architektur, !!! ABER NICHT das gleiche wie bei Ihrem KNX Gateway !!!

### Debug-Ebene
erweitert den Ausgabepegel des Adapters für Debugging-Zwecke

### Knxproj hochladen
Hier können Sie Ihren ETS-Export im "knxproj" -Format hochladen.

Nach erfolgreichem Import zeigt ein Dialog die Anzahl der importierten Objekte. Drücken Sie nun "Speichern & Schließen" und der Adapter sollte starten.
Beim Start liest der Adapter alle Gruppenadressen mit read-Flag. Dies kann einige Zeit dauern und Ihren KNX-Bus stark belasten. Die Werte in Ihrem Vis werden jedoch nach dem Start aktualisiert.

### Objekte
Hier finden Sie unter knx.0 den Gruppenadressbaum wie in Ihrem ETS-Projekt.

### Aufzählungen
Wenn Sie in Ihrer ETS eine Gebäudestruktur mit den entsprechenden Geräten haben, wird diese hier angezeigt. Unter "Mitglieder" sind die Namen der Gruppenadressen aufgeführt, die von den Geräten mit Send-Flag in dieser Gruppe stammen.

### Verwendungszweck
Wenn der Adapter erfolgreich gestartet wird, stehen Ihre Datenpunkte für alle gewünschten Aktionen zur Verfügung.

### Datenpunkttypen
Alle DPTs gemäß "Systemspezifikationen, Interworking, Datenpunkttypen" der KNX Association sind verfügbar. Das bedeutet, dass Sie zwei Arten von Informationen erhalten können: 1) einen Wert oder eine Zeichenfolge 2) durch Kommas getrennte Werte oder ein Array von Werten (im Moment weiß ich nicht, wie ich besser damit umgehen soll)

Beispielsweise wird ein DPT5.001 als vorzeichenlose Ganzzahl mit 8-Bit codiert. Dies ergibt einen einzelnen Wert. Der DPT3.007 (Control Dimming) ist als 1Bit (Boolean) + 3Bit (Int ohne Vorzeichen) codiert.
Dies führt zu f.e. in einem Wert wie "0,5", wobei "0" "Abnahme" bedeutet und "5" die Anzahl der Intervalle bedeutet.

## Wie werden die Datenpunkte generiert
### 1) Auslesen aller Kommunikationsobjektreferenzen (im folgenden KOR)
Dabei werden den Gruppenaddressreferenz (im folgenden GAR) ID's der jeweiligen DPT der KOR zugeordnet, wenn er vorhanden ist. Ausserdem bekommt der erste Eintrag das Attribut write = yes und read = no. Alle darauf folgenden GAR ID's bekommen nur den DPT zugeordnet

### 2) Suchen der Gruppenadressstruktur (im folgenden GAS)
Hier wird das Gas anhand der GAR ID's erzeugt und ebenfalls den DPT's zugeordnet, falls dies unter 1) noch nicht geschehen ist.

### 3) Herausfinden der Schalt- und Statusadressen
In dem ETS Export sind die Schalt- und Statusadressen nicht hinterlegt. Somit führe ich eine Ähnlichkeitsprüfung aller Gruppenadressnamen durch der Auswertung auf Status und Zustand.
Wird ein Pärchen gefunden, dessen Ähnlichkeit mehr als 90% beträgt, dann wird angenommen, dass der GA1 die Schaltadresse und der GA2 die Statusadresse ist. Dabei erhält GA1 das Schreiben = wahr und Lesen = falsch und GA2 das Schreiben = falsch und Lesen = wahr.
Ausserdem werden die DPT abgeglichen aus der jeweiligenig korrespondierenden GA. Aus diesem Grund ist es schwierig, Pärchen zu finden, wenn die Gruppenadressbeschriftungen nicht konsistent sind.

Weiterhin werden die Flags in den Gerätekonfigurationen betrachtet. Dabei werden die Flags wie folgt umgesetzt:

| KNX ||| iobroker |||
| Lesen | Schreiben | Übertragen | Lesen | Schreiben | Erklärung |
|-----------------------------------------------------------|
| - | - | - | - | - | der wert wird über GroupValueResponse aktualiesiert |
| x | - | - | x | x | ein Trigger darauf löst GroupValueRead aus |
| - | x | - | - | x | Schreibt den angegebenen Wert mit GroupValueWrite auf den KNX-Bus |
| - | - | x | x | - | der Wert wird über GroupValueResponse aktualisiert |
| x | - | x | x | x | ein Trigger darauf löst GroupValueRead aus |

### 4) Suchen der Datenpunktpaaren (im folgenden DPP)
Ein DPP wird erzeugt, wenn die GA, GAR und der DPT gültig sind. Mit diesen DPP arbeitet der Adapter. Fehlen auch die DPT's in einer GA, weil sie auf keinen der o. A. Wege gefunden worden, so wird für diese GA kein DPP erzeugt und is im weiteren nicht nutzbar.

Im Idealfall wird somit für einen Schaltkanal 2 DPP erzeugt. Das erste ist das Schalten. In diesem ist die GAR ID des Status DPP hinterlegt. The second is then the status DPP without other refenrenz.

## Beim Start des Adapters
Alle mit dem Lesen-Flag markieren DPP wird beim Start abgefragt. Dies verursacht u.U. eine höhere Buslast und dauert einen Moment. Im Anschluss sind aber alle aktuellen Werte vorhanden.

## (versteckt) Features:
Per GroupValueRead abgefragt.

### Vermeidung von Problemen
1) saubere ETS Programmierung und saubere ETS Programmierung und saubere ETS Programmierung

* zuweisen der DPT's !!
* einheitliche Bezeichnung der GA-Namen (z.B "EG Wohnen Decken Licht schalten" und "EG Wohnen Decken Licht schalten")
* Vermeidung von Sonderzeichen ",. /; \ &% $ § []" (kann zu Problemen bei der Erzeugung der GAS führen)

2) Prüfen ob das KNX / LAN GW erreichbar ist. Wenn es das nicht ist, versucht der Adapter sich kontinuierlich zu verbinden.

3) Physikalische Adresse richtig wählen (wichtig beim Einsatz von Linienkopplern). !!! ACHTUNG: die hier eingetragene physikalische Adresse ist NICHT die Adresse des LAN Gateways und darf nicht auf 0 enden !!!

4) Der Port der LAN Schnittstelle ist i.d.R. 3671

5) Durch die Möglichkeit der Statusabfrage wird Folgendes beachtet: Es wird das nicht mehr als 40 Anfragen pro Sekunde vom ioBroker generiert, da diese physikalisch bedingt nicht mehr durch den Adapter an das Gateway weitergereicht werden können.

## Geplante Funktionen
* Hinzufügen von Adressen zur Objektbeschreibung (ID)
* selektiver import von knx-projekten
* Node Version> 8.9.4 erforderlich!

## Changelog
### 1.0.33 (2019-09-12)
* fixed bug while writing to bus
* added units to states
* fixed "read/write of undefined" error

### 1.0.32 (2019-09-03)
* updated importer for ETS V5.7.2, some changes in KNX-stack statemachine

### 1.0.31
* some fixes on ETS5.7.2 importer
* small changes in knx-stack statemachine
* added (again) phys address to admin config dialog

### 1.0.31
* fixed bug in deviceTree generation

### 1.0.30
* new Importer for ETS5.7.2 knxproj files
* extended accepted Datapointtypes
* new adapter configuration menu
* implemented a switch for the user to decide to use "true" and "false" or "0" or "1" for binary values
* fixed bug in GroupValue_Read
* implemented a selector for local network interface for KNX to Gateway communiction
* extended State Object for later features
* fixed some small other bugs

### 1.0.20
* fixed bug in handling KNX-data packages, which occures periodical reconnects
* fixed bug in KNX-projectfile upload procedure

### 1.0.19
* reverted to true/false handling for DPT1.x

### 1.0.18
* fixed upload issue with ETS5.6.x project files
* switched values for "boolean" from 1 and 0 to true false 
* fixed recognition of role set for DPT1.x to switch
* fixed DPT16.xxx writing to KNX-Bus with values < 14Byte

### 1.0.17 (2018-08-16)
* Better state processing
* Add configurable package rate
* corrected Bug in "import only new objects"

### 1.0.15 (2018-07-18)
* change ChID on reconnect
* on Startup read wait for response of Statechannel or timeout

### 1.0.13 (2018-07-04)
* elemination of special signs while importing
* small bugfixes

### 1.0.12 (2018-06-19)
* reduced and sorted log output
* small bugfixes
* NEW Feature: request State/Val of stateObject from KNX-Bus

### 1.0.11 (2018-05-27)
* fixed DPT1 correcting value problem
* fixed reconnect problem
* other small optimizations and fixes

### 1.0.10 (2018-05-04)
* closing local port in case of undefinded connection state
* added advanced debug-level via adapter-config
* many fixes

### 1.0.9 (2018-04-29)
* changed to state-wise processing
* fixed "disconnect-request"
* changed connection handling with knxd
* many small fixes

### 1.0.8 (2018-04-04)
* modified package queue
* fixed ACK if sending to KNX-Bus
* many small fixes

### 1.0.7 (2018-03-16)
* fixed Adapter-lock while uploading projects

### 1.0.6 (2018-03-11)
* fixed connection problem
* corrected package counter

### 1.0.5 (2018-03-01)
* fixed empty objects, related to DPT1 (error message [object Object] unkown Inputvalue)
* fixed path variable
* fixed bug with GA's containing a "/" in the name (on proj-import)
* start implementing crosswise propery update on corresponding DPT (on proj-import)

### 1.0.4 (2018-02-27)
* schema update for room enumeration coming up with ETS 5.6

### 1.0.2 (2018-02-27)
* kleine Fehler beseitigt

### 1.0.1 (2018-02-26)
* fixed certifate error

### 1.0.0 (2018-02-25)
* substitution of used KNX-stack with own from scratch build stack
* implemented full scale of DPT according to "System Specifications, Interworking, Datapointtypes" from KNX Association
* hardening connection handling for tunneling connections
* upgrade Adapterconfiguration Interface to be ready with Admin3
* removed "Delay Slider" because of the new knx-stack
* many other small changes
* fixed postcomma values to scale-value of DPT
* implemented "add" mode for knxproject upload (existing Objects stay as they are, only new Objects where added)

### 0.8.6 (2017-06-17)
* some small bugfixes
* insert slider to set a sendDelay for slow KNX/LAN Gateways to prevent connection loss

### 0.8.5 (2017-06-05)
* project loader rebuild, dpt13-fix

### 0.8.3 (2017-04-24)
* added act channel update of corresponding state
* fix bug in state-vis update
* optimized knxproj upload

### 0.8.2 (2017-02-26)
* implemented device-config parsing from knxproj
* better choice of state/val of DP objects

### 0.8.1 (2017-02-06)
* fixed DPT1 switch problem

### 0.8.0 (2017-02-xx) comming soon

### 0.7.3 (2016-12-22)
* (chefkoch009) more DPT's are supported
* faster Startup
* implemented generation of room list with device dependicies

### 0.7.2 (2016-11-20)
* (chefkoch009) added necessary dependicies

### 0.7.1 (2016-11-19)
* (chefkoch009) Support standard KNX/LAN Gateways.

### 0.7.0 (2016-10-13)
* (chefkoch009) Support of project export

### 0.6.0 (2016-07-20)
* (chefkoch009) redesign

### 0.5.0
  (vegetto) include vis widget

#### 0.4.0
* (bluefox) fix errors with grunt

#### 0.2.0
* (bluefox) initial release

## License
The CC-NC-BY License (CC-NC-BY)

Copyright (c) 2016-2018 K.Ringmann <info@punktnetzwerk.net>

THE WORK IS PROVIDED UNDER THE TERMS OF THIS CREATIVE
COMMONS PUBLIC LICENSE ("CCPL" OR "LICENSE"). THE WORK IS PROTECTED BY
COPYRIGHT AND/OR OTHER APPLICABLE LAW. ANY USE OF THE WORK OTHER THAN AS
AUTHORIZED UNDER THIS LICENSE OR COPYRIGHT LAW IS PROHIBITED.

BY EXERCISING ANY RIGHTS TO THE WORK PROVIDED HERE, YOU ACCEPT AND AGREE
TO BE BOUND BY THE TERMS OF THIS LICENSE. TO THE EXTENT THIS LICENSE MAY
BE CONSIDERED TO BE A CONTRACT, THE LICENSOR GRANTS YOU THE RIGHTS
CONTAINED HERE IN CONSIDERATION OF YOUR ACCEPTANCE OF SUCH TERMS AND
CONDITIONS.

Read full license text in [LICENSE](LICENSE)