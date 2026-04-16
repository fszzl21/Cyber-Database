# Cheat Sheet – Week 06: Inleiding Systeembeveiliging

> Geschreven door: Faisal Anwari  

---

## 1. Windows bestandsrechten – NTFS permissies instellen (GUI)

1. Rechtsklik op een map → **Properties → Security**
2. Klik op **Edit** om rechten aan te passen
3. Voeg een gebruiker of groep toe via **Add**
4. Stel de gewenste standaard permissie in:
   - **Full Control** – alles inclusief rechten wijzigen
   - **Modify** – lezen, schrijven, verwijderen
   - **Read & Execute** – lezen en uitvoeren
   - **Read** – alleen lezen
   - **Write** – alleen schrijven

**Tip:** NTFS-rechten en share-rechten werken samen. De meest beperkende combinatie van beide wint. Stel NTFS-rechten zo gedetailleerd mogelijk in en houd share-rechten simpel (bijv. iedereen "Read"). Zo gelden de juiste rechten zowel lokaal als via het netwerk.

---

## 2. Windows bestandsrechten – Share aanmaken (GUI op DC of server met GUI)

1. Rechtsklik op map → **Properties → Sharing → Advanced Sharing**
2. Vink aan: **Share this folder**
3. Geef de share een naam
4. Klik op **Permissions** en stel de share-rechten in
5. Klik **OK** om op te slaan

**Tip:** Deny heeft altijd voorrang op Allow, maar gebruik Deny zo min mogelijk. Het AGDLP-principe (Account → Global group → Domain Local group → Permissions) maakt rechtenbeheer overzichtelijk en schaalbaar: ken rechten toe aan Domain Local groepen, voeg Global groepen toe aan die Domain Local groepen, en zet gebruikers in de Global groepen.

---

## 3. Windows bestandsrechten – Share aanmaken op Core server via MMC

1. Open op de grafische machine **Win+R → mmc**
2. Kies **File → Add/Remove Snap-in → Shared Folders**
3. Kies **Another Computer** en vul de naam van de Core server in
4. Klik op **Shares → rechtsklik → New Share** en volg de wizard
5. Sla de MMC op via **File → Save** voor hergebruik

Firewall op Core server openzetten voor bestandsdeling:

```cmd
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
```

Shares opvragen op Core server:

```cmd
net share
```

**Tip:** De Shared Folders wizard in MMC stelt standaard alleen share-rechten in, niet NTFS-rechten. Stel NTFS-rechten achteraf apart in via de eigenschappen van de map in Windows Verkenner, of gebruik PowerShell met `Set-Acl`.

---

## 4. Windows – Share aanmaken en rechten instellen via PowerShell

```powershell
New-Item -Name (mapnaam) -ItemType Directory
New-SmbShare -Name "(sharenaam)" -Path "(pad)" -FullAccess "Administrators" -ReadAccess "(groep)"
Get-SmbShare
Get-SmbShareAccess -Name "(sharenaam)"
```

Firewall volledig uitzetten (alleen in oefenomgeving):

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled false
```

**Tip:** `Get-SmbShare` toont alle actieve shares inclusief verborgen admin-shares zoals `C$` en `ADMIN$`. Met `Get-SmbShareAccess` zie je de share-rechten per share. Dit zijn andere rechten dan de NTFS-rechten op de onderliggende map.

---

## 5. Linux bestandsrechten – Overzicht notaties

### Symbolisch

```
drwxrwx--- 2 root sales 4096 jan 1 00:00 /data/sales
```

| Positie | Betekenis |
|---------|-----------|
| `d` | directory (`-` = bestand, `l` = link) |
| `rwx` | rechten eigenaar (user) |
| `rwx` | rechten groep |
| `---` | rechten overigen (others) |

| Recht | Op bestand | Op directory |
|-------|-----------|--------------|
| `r` | inhoud lezen | bestandenlijst opvragen |
| `w` | inhoud wijzigen | bestanden/submappen aanmaken |
| `x` | uitvoeren | naar de map navigeren (cd) |

### Numeriek (octaal)

| Getal | Rechten |
|-------|---------|
| 7 | rwx |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |
| 0 | --- |

---

## 6. Linux bestandsrechten – Eigenaar en groep instellen

```bash
sudo chown (gebruiker) (bestand of map)
sudo chown (gebruiker):(groep) (bestand of map)
sudo chown -R root:sales /data/sales
sudo chgrp (groep) (bestand of map)
```

**Tip:** `chown -R` past eigenaar en groep recursief toe op een map en alle bestanden daarin. Vergeet dit niet als je een mappenstructuur inricht — anders hebben bestanden die al bestonden nog de oude eigenaar.

---

## 7. Linux bestandsrechten – Permissies instellen

```bash
sudo chmod 755 (map)
sudo chmod 770 (map)
sudo chmod 600 (bestand)
sudo chmod -R 755 /data
sudo chmod u+rw,g+rw,o-rwx (bestand)
```

### Veelgebruikte combinaties

| Waarde | Rechten | Typisch gebruik |
|--------|---------|----------------|
| `755` | rwxr-xr-x | Publieke map of script |
| `770` | rwxrwx--- | Gedeelde map voor een groep |
| `700` | rwx------ | Prive map van een gebruiker |
| `644` | rw-r--r-- | Normaal bestand |
| `600` | rw------- | Privesleutel of wachtwoordbestand |

```bash
ls -l (pad)
ls -la (pad)
```

**Tip:** Begin altijd met `ls -l` om de huidige rechten te zien voordat je `chmod` of `chown` uitvoert. Zo weet je wat je wijzigt en voorkom je het per ongeluk wegnemen van bestaande rechten.

### Relevante paden

| Pad | Beschrijving |
|-----|--------------|
| `/data/` | Voorbeeldhoofdmap uit de opdracht |
| `/data/sales/` | Map voor de salesgroep |
| `/data/to1-project/` | Map voor projectgroep to1 |
| `/data/to2-project/` | Map voor projectgroep to2 |
| `/gegevens/` | Tweede mapstructuur uit de opdracht |

---

## 8. Linux – Gebruikers en groepen aanmaken

```bash
sudo useradd (gebruikersnaam)
sudo passwd (gebruikersnaam)
sudo groupadd (groepsnaam)
sudo usermod -a -G (groep) (gebruiker)
su - (gebruiker)
id (gebruiker)
```

**Tip:** De `-a` bij `usermod -a -G` staat voor "append". Zonder `-a` worden alle bestaande groepslidmaatschappen overschreven en is de gebruiker alleen nog lid van de opgegeven groep. Dit is een veelgemaakte fout.

---

## 9. Linux – Bijzondere rechten: setuid, setgid en sticky bit

### setuid – uitvoeren als eigenaar van het bestand

```bash
sudo chmod u+s (bestand)
sudo chmod 4755 (bestand)
```

Voorbeeld: `/usr/bin/sudo` draait altijd als root, ongeacht wie het aanroept.

Bestanden met setuid en root als eigenaar zoeken:

```bash
sudo find / -user root -perm -4000
```

**Tip:** Het setuid-bit op een bestand met root als eigenaar is een beveiligingsrisico als het in verkeerde handen valt. Zoek regelmatig naar dergelijke bestanden en verwijder onnodige exemplaren direct.

### setgid – uitvoeren als groep van het bestand, of groepseigendom in map

```bash
sudo chmod g+s (map)
sudo chmod 2770 (map)
```

**Tip:** Setgid op een map zorgt dat nieuwe bestanden die erin worden aangemaakt de groep van de map erven in plaats van de primaire groep van de maker. Dit is handig voor gedeelde projectmappen waar meerdere gebruikers bestanden moeten kunnen lezen.

### sticky bit – alleen eigenaar mag bestand verwijderen

```bash
sudo chmod +t (map)
sudo chmod 1777 (map)
```

**Tip:** Het sticky bit wordt klassiek gebruikt op `/tmp`. Iedereen mag bestanden aanmaken in `/tmp`, maar niemand mag elkaars bestanden verwijderen. Gebruik dit op gedeelde schrijfbare mappen om te voorkomen dat gebruikers elkaars werk verwijderen.

---

## 10. Linux – Symbolic link aanmaken

```bash
ln -s (doelbestand) (naam-van-link)
ln -s /data/topsecret.txt ~/secretpassage.txt
ls -la (map)
```

**Tip:** Een symbolic link is een verwijzing naar een ander bestand. Rechten op de link zelf hebben geen betekenis — wat telt zijn de rechten op het doelbestand. Schrijven via de link wijzigt het originele bestand direct.

---

## 11. SSH – Status controleren en inloggen

```bash
sudo systemctl status sshd
ssh (gebruiker)@(host)
ssh (gebruiker)@localhost
exit
```

**Tip:** Test SSH-verbindingen altijd eerst vanuit een aparte terminal voordat je de lopende sessie sluit. Als de configuratie een fout bevat en de server herstart, kun je jezelf buitensluiten.

---

## 12. SSH – sshd_config aanpassen

```bash
sudo nano /etc/ssh/sshd_config
```

### Belangrijke instellingen in sshd_config

| Instelling | Waarde | Betekenis |
|------------|--------|-----------|
| `Port` | `22` | Standaard poort |
| `PermitRootLogin` | `no` | Root-login uitschakelen |
| `PasswordAuthentication` | `no` | Wachtwoordlogin uitzetten na instellen van sleutels |
| `MaxAuthTries` | `6` | Max aantal inlogpogingen |

Na elke aanpassing de service herstarten:

```bash
sudo systemctl restart sshd
```

**Tip:** Zet `PermitRootLogin no` in zodra je een normaal gebruikersaccount hebt dat kan inloggen. Root-login via SSH is een van de meest aangevallen configuraties. Stel dit in als een van de eerste beveiligingsmaatregelen op een nieuwe server.

### Relevante bestanden (SSH)

| Pad | Beschrijving |
|-----|--------------|
| `/etc/ssh/sshd_config` | Configuratie van de SSH-server |
| `/etc/ssh/ssh_config` | Configuratie van de SSH-client (systeemwijd) |
| `~/.ssh/config` | Configuratie van de SSH-client (per gebruiker) |
| `~/.ssh/known_hosts` | Opgeslagen fingerprints van eerder verbonden servers |
| `~/.ssh/authorized_keys` | Publieke sleutels die mogen inloggen op dit account |
| `~/.ssh/id_ecdsa` | Private sleutel van de gebruiker |
| `~/.ssh/id_ecdsa.pub` | Publieke sleutel van de gebruiker |
| `/etc/issue` | Bericht dat wordt getoond voor het inloggen |
| `/etc/motd` | Bericht dat wordt getoond na het inloggen (Message of the Day) |

---

## 13. SSH – Sleutelpaar genereren en kopiëren (public key authenticatie)

### Sleutelpaar genereren (op de client, als gewone gebruiker)

```bash
ssh-keygen -t ecdsa
```

Accepteer de standaard locatie. Kies geen passphrase voor scriptgebruik.

### Publieke sleutel naar de server kopiëren

```bash
ssh-copy-id (gebruiker)@(host)
ssh-copy-id (gebruiker)@localhost
```

### Inloggen met sleutel testen

```bash
ssh (gebruiker)@(host)
```

Als het goed is, wordt er geen wachtwoord gevraagd.

### Rechten controleren op de sleutelbestanden

```bash
ls -al ~/.ssh/
```

Vereiste rechten:

| Bestand | Rechten |
|---------|---------|
| `~/.ssh/` | `700` (drwx------) |
| `~/.ssh/id_ecdsa` | `600` (-rw-------) |
| `~/.ssh/id_ecdsa.pub` | `644` (-rw-r--r--) |
| `~/.ssh/authorized_keys` | `600` (-rw-------) |

**Tip:** OpenSSH weigert te werken als de rechten op de `.ssh`-map of sleutelbestanden te ruim zijn. Als key-authenticatie niet werkt terwijl de sleutels er wel staan, controleer dan als eerste de rechten met `ls -al ~/.ssh/`.

---

## 14. SELinux – Status opvragen en modus wisselen

```bash
getenforce
sestatus
sudo setenforce 0
sudo setenforce 1
```

| Modus | Beschrijving |
|-------|-------------|
| `Enforcing` | SELinux blokkeert overtredingen actief |
| `Permissive` | SELinux logt overtredingen maar blokkeert niet |
| `Disabled` | SELinux volledig uit (niet aanbevolen) |

**Tip:** Gebruik `setenforce 0` (Permissive) tijdelijk om te testen of een probleem door SELinux wordt veroorzaakt. Als het probleem dan verdwijnt, weet je dat je een context of boolean moet aanpassen in plaats van SELinux permanent uit te zetten.

---

## 15. SELinux – Permanente modus instellen

```bash
sudo nano /etc/selinux/config
```

Pas de regel aan:

```
SELINUX=enforcing
```

of:

```
SELINUX=permissive
```

Actief na een reboot.

**Tip:** De instelling in `/etc/selinux/config` bepaalt de modus na een reboot. `setenforce` verandert de modus alleen tijdelijk voor de lopende sessie. Beide zijn nodig: `setenforce` voor onmiddellijke aanpassingen, de config voor permanente instellingen.

### Relevant bestand

| Pad | Beschrijving |
|-----|--------------|
| `/etc/selinux/config` | Permanente SELinux-configuratie |

---

## 16. SELinux – Contexts bekijken en aanpassen

### Context van bestanden bekijken

```bash
ls -Z (pad)
ls -lZ (pad)
```

### Context tijdelijk aanpassen

```bash
sudo chcon -t (context_type) (bestand of map)
sudo chcon -R -t httpd_sys_content_t /plf/www
sudo chcon --reference=(referentiepad) -R (doelpad)
```

### Context permanent vastleggen

```bash
sudo semanage fcontext -a -t (context_type) "(pad)(/.*)?"
sudo restorecon -R -v (pad)
```

**Tip:** `chcon` is tijdelijk — bij een SELinux-relabel gaat de instelling verloren. Gebruik altijd `semanage fcontext` gevolgd door `restorecon` voor een permanente oplossing. Dit is de juiste werkwijze bij een niet-standaard map voor een dienst zoals Apache of Samba.

---

## 17. SELinux – Fouten analyseren en booleans beheren

### Overtredingen zoeken in logbestanden

```bash
sudo ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts today
grep "SELinux is preventing" /var/log/messages
sudo grep http /var/log/audit/audit.log | audit2why
```

### Booleans bekijken en instellen

```bash
sudo semanage boolean -l
sudo setsebool (boolean_naam) on
sudo setsebool -P (boolean_naam) on
```

**Tip:** De `-P` vlag bij `setsebool` maakt de wijziging permanent (Persistent). Zonder `-P` geldt de boolean alleen tot de volgende reboot.

---

## 18. SELinux – Gebruikers en contexten

### Beschikbare SELinux users en rollen bekijken

```bash
sudo dnf install setools-console
seinfo -u
seinfo -r
```

### Context van de huidige gebruiker bekijken

```bash
id -Z
```

### Nieuwe gebruiker aanmaken met een confined SELinux label

```bash
sudo useradd -Z staff_u (gebruikersnaam)
sudo passwd (gebruikersnaam)
```

Na een reboot en inloggen als die gebruiker:

```bash
id -Z
```

Verwachte output bevat: `staff_u:staff_r:staff_t`

### Aangemelde gebruikers en hun SELinux labels bekijken

```bash
sudo semanage login -l
```

**Tip:** Een "confined" gebruiker zoals `staff_u` heeft beduidend minder rechten dan de standaard `unconfined_u`. Dit is het principe van "least privilege": een gebruiker krijgt alleen de rechten die strikt noodzakelijk zijn. Dit beperkt de schade als een account wordt gecompromitteerd.
