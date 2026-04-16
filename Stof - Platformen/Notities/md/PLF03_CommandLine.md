# Cheat Sheet – Week 03: Command Line (Linux Shell & Windows Server Core)

> Geschreven door: Faisal Anwari

---

## 1. Help & documentatie opzoeken (Linux)

```bash
(commando) --help
man (commando)
man -k (trefwoord)
sudo mandb
```

Navigeren in een manpage: spatie of `f` = volgende pagina, `b` = vorige pagina, `q` = afsluiten.

**Tip:** Gebruik `man -k` om een commando te vinden als je de exacte naam niet weet. Gebruik `sudo mandb` eenmalig na een nieuwe installatie om de manpage-database aan te vullen — anders geeft `man -k` geen resultaten.

---

## 2. Navigeren door het bestandssysteem (Linux)

```bash
pwd
cd (map)
cd ~
cd ..
cd /
cd /etc
ls
ls -r (map)
```

**Tip:** `pwd` vertelt je altijd waar je staat. Als je verdwaald raakt in een diepe mapstructuur is `cd ~` de snelste weg terug naar je home directory.

### Relevante paden

| Pad | Beschrijving |
|-----|--------------|
| `/` | Root van het bestandssysteem |
| `/home/(gebruiker)/` | Home directory van een gebruiker |
| `~` | Snelkoppeling naar eigen home directory |
| `/etc/` | Configuratiebestanden van het systeem |
| `/usr/share/doc/` | Documentatie van geinstalleerde pakketten |
| `/var/log/` | Logbestanden van het systeem |

---

## 3. Bestandsbeheer (Linux)

```bash
touch (bestand)
mkdir (map)
mkdir -p (pad/naar/map)
cp (bron) (doel)
cp -r (bronmap) (doelmap)
cp -v (bron) (doel)
mv (bron) (doel)
rm (bestand)
rm -r (map)
rm -rf (map)
rmdir (lege map)
```

**Tip:** `cp -r` kopieert mappen inclusief alle bestanden en submappen (recursief). Zonder `-r` werkt `cp` alleen op losse bestanden. Gebruik `rm -rf` met uiterste voorzichtigheid — er is geen prullenbak op de command line.

---

## 4. Output redirection (Linux)

```bash
(commando) > (bestand)
(commando) 2> (foutbestand)
(commando) > (uitvoer) 2> (fouten)
(commando) >& (bestand)
(commando) < (invoerbestand)
(commando1) | (commando2)
```

### Voorbeelden

```bash
ls /etc > ~/etc.lst
cp -r /etc ~/backup 2> ~/backuperrors.log
cp -rv /etc ~/backup > ~/backup.lst 2> ~/backuperrors.log
cat ~/backuperrors.log | more
```

**Tip:** Kanaal `1` is standaard uitvoer (wat goed gaat), kanaal `2` is foutuitvoer (wat mis gaat). Door ze apart om te leiden kun je foutmeldingen in een eigen logbestand opslaan zonder dat ze de normale uitvoer vervuilen.

---

## 5. Bestanden weergeven en doorzoeken (Linux)

```bash
cat (bestand)
more (bestand)
less (bestand)
grep (zoekterm) (bestand)
grep -i (zoekterm) (bestand)
find (startmap) -name "(bestandsnaam)"
find (startmap) -name "*(patroon)*"
wc -l (bestand)
```

### Voorbeelden

```bash
cat /etc/hosts
cat /etc/hosts -n
grep started /var/log/messages
grep -i error /var/log/messages
find /usr/share/doc -name "*linux*"
find /usr/share/doc -name "README*" | wc -l
```

**Tip:** Linux is hoofdlettergevoelig. `grep error` vindt "error" maar niet "Error". Gebruik `grep -i` voor hoofdletterongevoelig zoeken. Met `| wc -l` tel je het aantal resultaten in plaats van ze allemaal te tonen.

---

## 6. Omgevingsvariabelen (Linux)

```bash
echo $USER
echo $HOSTNAME
echo $HOME
echo $PATH
set
(VARIABELENAAM)=(waarde)
export (VARIABELENAAM)
```

### Voorbeeld: variabele aanmaken en gebruiken

```bash
BACKUPDIR=~/backup
LOGBESTAND=~/error.log
cp -r /etc $BACKUPDIR 2> $LOGBESTAND
```

**Tip:** Variabelen die je aanmaakt zijn alleen beschikbaar in de huidige shell-sessie. Met `export` maak je ze beschikbaar voor alle processen die vanuit die sessie worden gestart (kindprocessen). Zonder `export` ziet een script de variabele niet.

### Relevante configuratiebestanden voor shell-opstartscripts

| Bestand | Wanneer geladen | Voor wie |
|---------|----------------|---------|
| `/etc/profile` | Login shell | Alle gebruikers, alle shells |
| `~/.bash_profile` | Login shell | Specifieke gebruiker, alleen bash |
| `~/.profile` | Login shell | Specifieke gebruiker, alle shells |
| `/etc/bashrc` | Non-login shell | Alle gebruikers, alleen interactieve bash |
| `~/.bashrc` | Non-login shell | Specifieke gebruiker, alleen interactieve bash |

---

## 7. Procesbeheer (Linux)

```bash
ps aux
top
pgrep (procesnaam)
kill (PID)
```

**Tip:** `ps aux` geeft een momentopname van alle draaiende processen. `top` toont processen live en ververst automatisch. Verlaat `top` met `q`. Het PID (Process ID) heb je nodig om een proces te stoppen met `kill`.

---

## 8. Shell scripting – Opstartscripts aanpassen

### Welkom-melding alleen voor root (login shell)

Voeg toe aan `/root/.bash_profile`:

```bash
sudo nano /root/.bash_profile
```

```bash
echo "Welkom, super(wo)man!"
```

**Tip:** `/root/.bash_profile` wordt alleen geladen bij het inloggen als root via een login shell (SSH, console). Dit bestand bestaat mogelijk nog niet — nano maakt het automatisch aan.

### Inhoud van /etc/redhat-release tonen voor root (alle shells)

Voeg toe aan `/root/.bashrc`:

```bash
sudo nano /root/.bashrc
```

```bash
cat /etc/redhat-release
```

### PATH uitbreiden voor alle gebruikers

Voeg toe aan `/etc/profile`:

```bash
sudo nano /etc/profile
```

```bash
export PATH=$PATH:/opt/microsoft/powershell/7
```

**Tip:** `/etc/profile` wordt geladen voor alle gebruikers bij een login shell. Door het bestaande `$PATH` te behouden met `$PATH:` voeg je het nieuwe pad toe in plaats van de hele variabele te overschrijven.

### todo.txt check voor gewone gebruikers (niet root, alle shells)

Voeg toe aan `/etc/bashrc`:

```bash
sudo nano /etc/bashrc
```

```bash
if [ "$USER" != "root" ]; then
    if [ -f ~/todo.txt ]; then
        cat ~/todo.txt
    else
        echo "Je hebt nog geen todo.txt! Maak deze z.s.m. aan."
    fi
fi
```

**Tip:** `/etc/bashrc` wordt geladen bij elke interactieve shell (ook non-login). De `if [ -f ... ]` constructie controleert of een bestand bestaat. De `$USER != "root"` check zorgt dat root deze melding niet ziet.

---

## 9. Windows Server Core – Basisconfiguratie

### sconfig starten

```
sconfig
```

Relevante opties in sconfig:

| Optie | Functie |
|-------|---------|
| 2 | Computernaam wijzigen |
| 1 | Domeinlidmaatschap instellen |
| 8 | Netwerkinstellingen |
| 4 | Remote management |

**Tip:** sconfig is een tekst-menu dat vrijwel alles regelt voor de initiele configuratie van een Core server. Gebruik het als startpunt voordat je naar PowerShell of netsh gaat.

### Netwerk instellen via netsh

```cmd
netsh interface ipv4 show interface
netsh interface ipv4 set address name=(id) source=static address=(IP) mask=(subnetmask) gateway=(gateway-IP)
netsh interface ipv4 add dnsserver name=(id) address=(DNS-IP) index=1
```

**Tip:** Zoek eerst het interface-ID op met `show interface` voordat je de overige netsh-commando's uitvoert. Het ID is een getal of een naam zoals "Ethernet".

---

## 10. Windows Server Core – Computernaam & Domein (PowerShell)

### Naar PowerShell switchen vanuit cmd

```cmd
powershell
```

### Computernaam wijzigen en herstarten

```powershell
Rename-Computer (naam)
Restart-Computer
```

### Domein toevoegen

```powershell
Add-Computer -DomainName (domeinnaam) -Credential (domein\gebruiker)
Restart-Computer
```

**Tip:** Na het toevoegen aan het domein moet je herstarten voor de wijziging actief wordt. Log daarna in met `(gebruiker)@(domeinnaam)` om expliciet het domeinaccount te gebruiken in plaats van het lokale account.

### PS Remoting inschakelen

```powershell
Enable-PSRemoting
```

**Tip:** PS Remoting is nodig als je de Core server op afstand wilt beheren via PowerShell vanaf een andere machine. Zonder dit werkt `Enter-PSSession` niet.

---

## 11. Windows Server Core – Firewall & Remote beheer

### Remote beheer via netsh inschakelen

```cmd
netsh advfirewall firewall set rule group="windows remote management" new enable=yes
netsh advfirewall set currentprofile settings remotemanagement enable
```

### Firewall volledig aan of uit

```cmd
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off
```

### Remote Desktop inschakelen via sconfig

Kies in sconfig optie **4** (Remote Management) en optie **7** (Remote Desktop).

**Tip:** De firewall op een Core server blokkeert standaard remote beheer. Zet je de firewall tijdelijk uit voor de initiële configuratie, zet hem daarna altijd terug aan en configureer alleen de specifieke regels die je nodig hebt.

### Verbinding maken met Core server via PowerShell (vanaf een andere machine)

```powershell
Enter-PSSession -ComputerName (servernaam)
```

---

## 12. Windows Server Core – Rollen installeren (PowerShell)

### Beschikbare rollen bekijken

```powershell
Get-WindowsFeature
```

### Active Directory Domain Services installeren

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Promoveren tot DC in bestaand domein

```powershell
Install-ADDSDomainController -DomainName "(domeinnaam)" -Credential $(Get-Credential)
```

### DNS-rol installeren

```powershell
Install-WindowsFeature DNS -IncludeManagementTools
```

**Tip:** Voeg altijd `-IncludeManagementTools` toe aan `Install-WindowsFeature`. Zonder deze vlag installeer je de rol maar niet de bijbehorende beheerhulpmiddelen, waardoor je de rol niet lokaal kunt beheren.

---

## 13. Windows Server Core – Bestandsdeling via cmd

```cmd
net view \\(servernaam)
net use (stationsletter): \\(servernaam)\(sharenaam)
```

**Tip:** Met `net use` koppel je een netwerk-share aan een stationsletter (bijv. `D:`). Dit is het command-line equivalent van "Map network drive" in Windows Verkenner.

---

## 14. MMC met snap-ins instellen (op client of DC, remote beheer)

1. Start via **Win+R**: typ `mmc`
2. Kies **File → Add/Remove Snap-in**
3. Voeg toe: **Active Directory Users and Computers**, **Computer Management**, of **DNS**
4. Kies bij elk snap-in voor een andere computer en vul de servernaam in

**Tip:** Via MMC met snap-ins beheer je een Core server alsof je er grafisch op zit, maar dan vanaf een machine met GUI. Installeer hiervoor RSAT (Remote Server Administration Tools) op de Windows 10 client.
