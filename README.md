# Heating control based by person state

## What is this blueprint for?

In the last few years I have been switching between home office and office very irregularly. Calendar-dependent heating control is not flexible enough for this, so I tried automation, but it became very confusing, error-prone and difficult to adapt. This blueprint solves these problems.

[comment]: <> (In den letzten Jahren wechsel ich sehr unregelmäßig zwischen Homeoffice und Büro. Eine kalenderabhängige Heizungssteuerung ist dafür nicht flexible genug, also habe ich es mit einer Automation versucht, die aber sehr unübersichtlich, fehleranfällig und schwer anpassbar wurde. Dieses Blueprint löst diese Probleme.)   


[comment]: <> (Das Blueprint ist sicher nicht ausgereift und enthält wahrscheinlich auch Fehler. Bevor du jetzt mutig das Blueprint importierst und einsetzt, ist sicher ein Backup sinnvoll! Außerdem solltest du deine bisherige Heizungsautomation auch deaktivieren. ) 
**The blueprint is certainly not fully developed and probably contains errors**   
**Before you boldly import and use the blueprint, a backup certainly makes sense! You should also deactivate your previous heating automation.**   

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fquietcry%2FtgHeating%2FtgHeating.yaml)

## Table of Contents

1. [Project Advantages](#-project-advantages)
2. [Project Disadvantages](#-project-disadvantages)
3. [Documentation & Resources](#-documentation)


## 🌟 Project Advantages
[comment]: <> (
| trennt Logik und Konfiguration
| der debug-Modus erlaubt die Nachvollziehbarkeit der Logik  
| einfach erweiterbar 
) 
- **No Coding Required:** separates logic and configuration
- **debug-mode:** The optional debug mode allows the logic to be traced
- **handling:** easily expandable or splitable in seperate automations

## Project Disadvantages
[comment]: <> (
ja, es gibt natürlich auch Nachteile     
| alle Befehle in der Konfiguration sind json-strings, das macht es fehleranfällig, da eine Überprüfung auf validen json-code in einem Blueprint nicht möglich ist. Ich empfehle die Verwendung eines json-Validierers 
| um das Blueprint richtig nutzen zu können, musst du wahrscheinlich ein paar Helfer anlegen  
) 
yes, of course there are also disadvantages
- **JSON-Code:** All commands in the configuration are json strings, which makes it error-prone because checking for valid json code is not possible in blueprint. I recommend using a json validator like [JSONLint](https://jsonlint.com/)
- **additionals:** To use the blueprint properly, you'll probably need to create a few helpers

## 📚 Documentation

#### briefly how it works
[comment]: <> (
\- Die Automation wird bei jedem Statusänderungsereignis und dem Start von HA getriggert. Das Triggerereignis wird gegen die konfigurierten Personen, Schalter und Helfer geprüft. 
\- die default-Konfiguration wird in die leere Gesamtkonfiguration eingelesen
\- die konfigurierten Schalter werden nacheinander geprüft und deren Konfiguration in die Konfiguration integriert. \(Sollte der exclusive Modus eines Schalters aktiviert sein, wird das Einlesen beendet und mit dem Einstellen der Thermostate begonnen.\) 
\- Der Status der Personen wird ausgewertet und wenn es zu diesem einen Eintrag im Befehlsfeld gibt, wird dieser Eintrag in die Basiskonfiguration integriert.
\- die Thermostate werden entsprechend der Konfiguration eingestellt
)
- The automation is triggered on every status change event and the start of HA. The trigger event is checked against the configured people, switches and helpers.
- the default configuration is read into the empty command variable
- The configured switches are checked one after the other and their configuration is merged with the command variable. \(If the exclusive mode of a switch is activated, the reading is ended and the setting of the thermostats begins.\)
- The status of the people is evaluated and if there is an entry for them in the command field, this entry is integrated into the command variable.
- the thermostats are set according to the command variable

#### What do I mean by "command set"
commands sets are setted in Blueprint configuration as JSON-Objects. A command set consists of zero to as many command elements as necessary. A command element is a key-value pair. The key identifies the device, while the value contains the actual command 

```{<key>:<value>, <key>:<value>. ...}```
* The **key** is a unique substring from the thermostat's "friendly_name", starting from the beginning of the word.  
In my case the "friendly_name" consists of an abbreviation for the room, a number if the room has several radiators and the device type.
For example, in the living room the two thermostats are called  
  - LR_1_thermostate
  - LR_2_thermostate  

  So I can use "LR_1" as a unique key to identify the thermostate.
* The **value** indicates what should be done, for example set the target temperatur on Thermostate 1 in Living room to 24°C.
```{"LR_1":24}``` 

#### how the merging of command configurations works 
if you have two commands for the same device, first is checked which command is later in sequence list. If command is the same, the highest program parameter wins (alphanumeric sorted). If you want to change that note a "!" as first sign of parameter.
| frank      | fiona | winner | justification|
| ----------- | ----------- | ----------- | ----------- |
|```{"LR_1":"20"}```|```{"LR_1":"22"}```|fiona| 22 is warmer than 20
|```{"LR_1":"off"}```|```{"LR_1":"22"}```|fiona| temperature comes later in sequence
|```{"LR_1":"WP>abc"}```|```{"LR_1":"WP>cba"}```|fiona| cba is higher than abc if alphanumeric sorted
|```{"LR_1":"P>3"}```|```{"LR_1":"P>1"}```|frank| 3 is higher than 1 if alphanumeric sorted
|```{"LR_1":"!20"}```|```{"LR_1":"22"}```|frank| 22 is warmer than 20 but "!" overrides it

## Configuration
[comment]: <> (wenn aktiviert, werden Angaben zum Blueprint als Persistent Notification angezeigt. Mindestens zwei Notifications sollten pro Aufruf gesetzt werden, wenn das Blueprint fehlerfrei läuft. die erste Notification zeigt die eingelesene Konfiguration, die zweite Notification zeigt das Resultat vor dem Setzen der Thermostate. Weitere Notifications werden beim Setzen der Thermostate erzeugt)
+ ***debug-mode [default=off]***  
If activated, information about the blueprint is displayed as a persistent notification. At least two notifications should be set per call if the blueprint runs without errors. The first notification shows the configuration read in, the second notification shows the result of the evaluation before the thermostats were set. Further notifications are generated when the thermostats are set

[comment]: <> (hier setzt du die Personen deren Status du berücksichtigen willst. Du kannst entweder direkt die person-entity oder einen Text-Helfer angeben der den Status als Text enthält. Das "friendly_name" - Attribut wird als Suchschlüssel in der Konfiguration verwendet)
+ ***person-state entities [optional]***  
here you set the people whose status you want to take into account. You can either specify the person entity directly or a text helper that contains the status as text. The "friendly_name" attribute is used as a search key in the configuration

[comment]: <> (bitte selektiere hier alle Heizungsgeräte die du steuern willst. Das "friendly_name" - Attribut sollte dabei mit dem Kürzel beginnen, welches du in in deiner Konfiguration verwendest)
+ ***heating devices [optional]***  
Please select all heating devices that you want to control here. The "friendly_name" attribute should start with the unique abbreviation that you use in your configuration

  ### Base Configuration

[comment]: <> (gib hier Name-Wert Paare an, die angeben welchen Zustand deine Thermostate als default annehmen sollen )

* ***default command [default= {} ]***  
In a json object, enter name-value pairs that indicate which state your thermostats should assume as default  
e.g.```{"KI":20, "LR_1":"off"}```

[comment]: <> (eine Komma-separierte Liste von Befehlen die steuern welcher Befehl gewinnt wenn beim Zusammenfügen der Befehlsvariable zwei Befehle für ein Gerät abgearbeitet werden sollen. Der weiter rechts stehende Befehl gewinnt.  )

* ***sequence [default= "off,boost,T,WP,P" ]***  
a comma-separated list of commands that control which command wins if two commands are to be processed for a device when the command variable is combined. The command further to the right wins.  

| Command      | Description | Example |
| ----------- | ----------- | ----------- |
| off      | turn device off       | ```{"LR_1":"off"}```
| boost>[2.Command]     | turn on boost mode and then do the second command<br>(not impemented yet)       | ```{"LR_1":"boost>22"}```<br>```{"LR_1":"boost>"WP>noOneHome"}```
| T   | represents a Number or a number representing string as target temperature       | ```{"KI":"20"}```<br>```{"KI":20}```<br>```{"KI":20.5}```<br>```{"KI":"20.5"}```<br>```{"KI":"20,5"}```<br>
|WP>[key]|use weekplan given by key| ```{"LR_1":"WP>noOneHome"}```
|P>[key]|only for Homematic devices (HM, HmIP) set weekprogram to the by key given number<br>key=[1-3]| ```{"LR_1":"P>1"}```


### store in Statefile
not implemented in the moment<br>
for use add following lines to your ha configuration.yaml
```
shell_command:
  write_to_file: /bin/bash -c "echo '{{txt}}' >> {{file}}"
  read_from_file: /bin/bash -c "cat {{file}}"
  ```

[comment]: <> (Pfad zu einer Text Datei welche Daten aus dem Blueprint zwischenspeichert um sie bei einem späteren Durchlauf nutzen zu können. Stelle sicher, dass die Datei existiert und für HA schreibbar und lesbar ist oder HA in dem Ordner Schreibrechte hat  )
* ***state file path: [default= "" ]***  
Path to a text file that caches data from the blueprint so that it 
can be used in a later run. Make sure the file exists and is writable
and readable by HA or HA has write permissions in the folder

### Weekplan Configuration
[comment]: <> (ein verschachteltes JSON-Objekt, 
jeder Hauptschlüssel stellt einen eigenen Wochenplan dar. 
Jeder Wochenplan hat 7 Schlüssel \( Zahlen 1-7 \) 1=Montag, jeder Schüssel 
ist eine Liste mit Paaren aus Zeit Befehl oder ein Verweis auf 
einen anderen Wochentag oder ein Verweis auf einen anderen Wochentag 
in einem anderen Wochenplan)

* ***Weekplans: [optional]***  
a nested JSON object, each master key represents its own weekly schedule. Each weekly schedule has 7 keys (numbers 1-7) 1=Monday, each key is a list of pairs from **\<time>-\<command>**, sorting is not necessary or a reference to another day of the week or a reference to another day of the week in another weekly schedule

```
{
"noOneHome":                        
  {
  "1":["06:00-20", "20:00-15"],     
  "2":"*",  
  "3":["09:00-22", "22:00-off"],
  "4":"1",
  "6":"*",     
  "7":"7:someOneHome"     
  },
"someOneHome":
  {
  "7":["09:00-22", "22:00-off"]
  }  
}
```
means:  
if weekplan "noOneHome" is active
- on Monday set temperature to 20°C at 6:00 and to 15°C at 20:00
- on Tuesday use the same settings like the last configurated day, in this case Monday 
- on Wednesday set temperature to 22°C at 9:00 and turn off device at 22:00
- on Thursday use the same settings like Monday
- on Friday nothing is configurated -> nothing happens temperature is still 15°C
- on Saturday, use the same settings as the last configured day, in this case Thursday, which is linked to Monday
- on Sunday, use the configuration of sunday in weekplan "someOneHome"

* ***Weekplan-Timer: [optional]***  
a timer entity, controled by this blueprint for triggering the next switching point of the weekplans   

### Command line
[comment]: <> (ein verschachteltes JSON-Objekt, 
jeder Hauptschlüssel stellt eine Person dar. 
Jede Person ist ein Objekt dessen Schlüssel gleich einem möglichen Statuswert sind.
Jeder Status entspricht einem Objekt wie unter Base Configuration, default beschrieben.
)

in nested JSON object, 
each master key represents a person. 
Each person is an object whose keys are equal to a possible status value.
Each status corresponds to an object as described under Base Configuration, default.

```
{
"frank":                        
  {
  "home":{"LR_1":"20", "KI":"18"},     
  "not_home":{"LR_1":"15", "KI":"15"},   
  "tv":{"LR_1":"21"},
  "homeoffice":{"LR_1":"WP>noOneHome", "KI":"15"}
  },
"fiona":
  {
  "home":{"LR_1":"21", "KI":"20"},
  "tv":{"LR_1":"22"}
  }  
}
```
### switch 1 - switch 5
[comment]: <> (ein verschachteltes JSON-Objekt, 
jeder Hauptschlüssel stellt eine Person dar. 
Jede Person ist ein Objekt dessen Schlüssel gleich einem möglichen Statuswert sind.
Jeder Status entspricht einem Objekt wie unter Base Configuration, default beschrieben.
)

* ***switch entity: [optional]***  
a boolean or schedule helper

* ***on-cmd: [default={} ]***  
Commands that are executed when the switch is turned on
* ***exclusive Mode (on): [default=false ]***  
if true the merging of commands ends here 
* ***off-cmd: [default={} ]***  
Commands that are executed when the switch is turned on
* ***exclusive Mode (off): [default=false ]***  
if true the merging of commands ends here 
