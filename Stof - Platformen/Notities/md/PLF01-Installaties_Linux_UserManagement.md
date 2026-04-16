# Linux Fundament en User Management + Windows VM aanmaken

> Geschreven door : Faisal Anwari
---

## Belangrijke bestanden & mappen
```
/etc/passwd          # Gebruikersaccounts
/etc/shadow          # Versleutelde wachtwoorden
/etc/group           # Groepen
/etc/sudoers         # Sudo-rechten configuratie
/etc/skel/           # Standaard home-directory inhoud (wordt gekopieerd bij useradd -m)
/home/(gebruiker)/   # Home-directories van gebruikers
/root/               # Home-directory van root
```

---

> Honorable mention :P

## VMware — VM aanmaken

**Windows:**
1. VMware → New Virtual Machine → Custom
2. "I will install the operating system later"
3. Sla op buiten OneDrive (bijv. `C:\VMs`)
4. Koppel ISO achteraf via VM Settings → CD/DVD

**Linked clone maken:**
1. Sysprep draaien in de VM: `C:\Windows\System32\sysprep\sysprep.exe`
   - Kies OOBE → vink "Generalize" aan → Shutdown
2. In VMware: rechtermuisklik op VM → Manage → Clone
3. Base/parent VM nooit meer opstarten!

> 💡 Linked clones delen de basisdisk met de parent — dit bespaart veel schijfruimte bij meerdere VM's van hetzelfde OS.

---

## Package management (DNF)
```bash
# Updaten
sudo dnf upgrade

# Pakket installeren
sudo dnf install (pakket)
sudo dnf install (pakket1) (pakket2)      # Meerdere tegelijk

# Pakket verwijderen (wildcards mogelijk)
sudo dnf remove (pakket)
sudo dnf remove iwl*-firmware             # Voorbeeld wildcard

# Groepen bekijken en installeren
sudo dnf group list
sudo dnf group info (groepsnaam)
sudo dnf group install "(groepsnaam)"
```

> 💡 Gebruik wildcards met `*` om meerdere gerelateerde pakketten in één keer te verwijderen, bijv. alle WiFi-firmware op een server die dat toch nooit nodig heeft.

---

## Groepen aanmaken
```bash
sudo groupadd (groepsnaam)
```

---

## Gebruikers aanmaken
```bash
# Gebruiker aanmaken met home-directory
sudo useradd -m -g users -G (groep1),(groep2) (gebruikersnaam)

# Wachtwoord instellen
sudo passwd (gebruikersnaam)

# Inloggen als andere gebruiker
su (gebruikersnaam)
```

> 💡 `-m` zorgt dat de home-directory aangemaakt wordt. Zonder `-m` heeft de gebruiker geen eigen map, wat problemen geeft bij inloggen.

---

## Wachtwoordbeleid instellen
```bash
sudo chage -M 40 -W 7 -m 1 (gebruikersnaam)   # max / waarschuwing / min dagen
sudo chage -l (gebruikersnaam)                  # Instellingen bekijken
```

> 💡 In een bedrijfsomgeving is wachtwoordbeleid verplicht vanuit beveiligingsstandaarden (bijv. ISO 27001). Met `chage` stel je dit per gebruiker in, maar via Group Policy (Windows) of PAM (Linux) doe je dit organisatiebreed.

---

## Account locken / unlocken
```bash
sudo usermod -L (gebruikersnaam)    # Locken
sudo usermod -U (gebruikersnaam)    # Unlocken
```

> 💡 Lock een account als iemand uit dienst gaat maar je de data nog niet wilt verwijderen. In `/etc/shadow` herken je een vergrendeld account aan een `!` voor het wachtwoord.

---

## Gebruiker verwijderen
```bash
sudo userdel -r (gebruikersnaam)    # -r verwijdert ook home-directory
```

> 💡 Zonder `-r` blijft de home-directory staan. Handig als je eerst een backup wilt maken van de bestanden van de gebruiker.

---

## Sudo-rechten geven
```bash
# Optie 1: gebruiker toevoegen aan wheel-groep (aanbevolen)
sudo usermod -aG wheel (gebruikersnaam)

# Optie 2: sudoers bestand aanpassen
sudo visudo                          # Opent nano als editor
# Voeg toe: (gebruikersnaam) ALL=(ALL) ALL
```

> 💡 Geef sudo-rechten alleen aan mensen die het echt nodig hebben. Via de wheel-groep is het eenvoudiger te beheren dan losse regels in sudoers — je hoeft alleen groepslidmaatschap aan te passen.

---

## SSH verbinding vanuit Windows

1. IP-adres opzoeken in Linux: rechtsboven op netwerkicoon → Wired Settings → tandwiel
2. PuTTY openen → vul IP-adres in → Open → inloggen met gebruikersnaam + wachtwoord

> 💡 In de praktijk beheer je servers bijna altijd via SSH — je logt zelden fysiek in op een server. Leer PuTTY of Windows Terminal goed kennen.
