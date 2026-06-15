### Intro 

Bash is een script taal. GEEN programmeer taal!

**Wat is het verschil dan?** 
Verschil tussen script en programmeer taal is dat het niet nodig is om bij scripting de code te compilen om het te draaien.

**Waarvoor gebruikt?**
- Terminal te besturen
-  Processen automatiseren
- Data filteren

**Voorbeeld script executies.**
```
./script.sh < optionele argumenten>
```
of 
```
sh script.sh <optionele argumenten>
```

## Conditionele Exectutie (If statements)

If statements bepaalt het flow van de script door verschillende condities te bereiken. Als je een specifieke waarde hebt en een if statement die eraan voldoet  dan wordt alleen die geexecuteerd en de andere afgewezen.

Voorbeelden hiervan zijn if, else, then en fi.

**Conditionele statements is wel een van de belangerijktste onderdeel van een script.**
## Shebang
Een shebang is een lijn die altijd top van de script gaat. Het verteld de terminal van "Dit is een script. Ga naar dit bestand om het script te vertalen."

**Voorbeelden van shebangs.**

Een bash script laten draaien
```
#!/bin/bash 
```

Een python script draaien
```
#!/usr/bin/env python
```

