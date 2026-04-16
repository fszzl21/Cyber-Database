# PLF03 Active Directory & Windows Gebruikers — Cheat Sheet

> Geschreven door: Faisal Anwari

---

## Belangrijke mappen & locaties
```
C:\Windows\System32\sysprep\   # Sysprep tool
\\(server)\Homefolders\        # Centrale homefolders share
\\(server)\Roaming\            # Roaming profielen share
\\(server)\Mandatory\          # Mandatory profielen share
```

---

## Domain Controller promoveren

**GUI:**
1. Server Manager → Manage → Add Roles and Features
2. Role-based installation → selecteer server
3. Vink **Active Directory Domain Services** aan
4. Na installatie: klik gele uitroepteken → "Promote this server to a domain controller"
5. Nieuwe forest → domeinnaam invullen (bijv. `bedrijf.local`)
6. DNS Server aangevinkt laten, DSRM-wachtwoord instellen → installeer

> 💡 Noteer het DSRM-wachtwoord op een veilige plek — je hebt het zelden nodig maar het is cruciaal bij herstel.

**Tweede DC / subdomein toevoegen:**
- Zelfde stappen, maar kies **Add a new domain to an existing forest**
- Zorg dat de tweede DC de eerste DC als primaire DNS gebruikt
- Test eerst: `ping (domeinnaam)`

---

## Server naam & vast IP instellen

**GUI:**
1. Rechtermuisklik Start → System → Rename this PC
2. Netwerk: Control Panel → Network → Ethernet → Properties → IPv4
   - Vul vast IP in, bijv. `192.168.1.10`
   - DNS: IP van de DC (of eigen IP als dit de DC zelf is)

> 💡 Een DC heeft altijd een vast IP nodig — DHCP zorgt voor problemen bij AD en DNS.

---

## Windows client aan domein toevoegen

**GUI:**
1. Start → System → Rename this PC (Advanced) → Change
2. Member of: **Domain** → domeinnaam invullen
3. Inloggen met domeinbeheerder → herstart
4. Controleer daarna in ADUC onder **Computers**

---

## RSAT installeren op Windows 10/11

**GUI:**
1. Settings → Apps → Optional Features → Add a feature → zoek `RSAT`
2. Installeer minimaal:
   - AD Domain Services Tools
   - DHCP Server Tools
   - DNS Server Tools
   - Group Policy Management Tools
   - Server Manager

> 💡 Met RSAT beheer je servers vanuit je eigen werkplek zonder in te loggen op de server zelf.

---

## Organizational Units (OU) aanmaken

**GUI (ADUC of ADAC):**
1. Rechtermuisklik op domein of bovenliggende OU → New → Organizational Unit
2. Laat "Protect from accidental deletion" aangevinkt

**PowerShell:**
```powershell
# Directe OU onder domein
New-ADOrganizationalUnit -Name "Medewerkers" -Path "DC=bedrijf,DC=local"

# OU binnen een andere OU
New-ADOrganizationalUnit -Name "IT" -Path "OU=Medewerkers,DC=bedrijf,DC=local"
```

> 💡 OU's gebruik je om gebruikers, computers en printers logisch te groeperen én om Group Policies gericht toe te passen.

---

## Gebruikers aanmaken & beheren

**GUI (ADUC):**
- Rechtermuisklik op OU → New → User
- Logon naam: `voornaam.achternaam`

**Template user kopiëren:**
- Rechtermuisklik op template user → Copy → nieuwe naam invullen
- Zet template altijd op **Disabled** als hij niet gebruikt wordt
- Disabled account herkenbaar aan pijltje (↓) op het icoon

**PowerShell (bulk import via CSV):**
```powershell
Import-Csv "gebruikers.csv" | New-ADUser -Enabled $true `
  -AccountPassword (ConvertTo-SecureString "Wachtwoord1!" -AsPlainText -Force)
```

**Gebruiker verplaatsen naar andere OU:**
- GUI: rechtermuisklik → Move → kies doel-OU
- PowerShell:
```powershell
Move-ADObject -Identity "CN=Jan Jansen,OU=Oud,DC=bedrijf,DC=local" `
  -TargetPath "OU=Nieuw,DC=bedrijf,DC=local"
```

---

## Homefolders & profielen instellen

**Shares aanmaken:**
1. Maak mappen aan op de server
2. Rechtermuisklik → Properties → Sharing → Advanced Sharing
3. Geef **Authenticated Users** Full Control

**Homefolder koppelen (ADUC):**
1. Selecteer gebruiker(s) → Properties → tab **Profile**
2. Home folder → Connect → schijfletter kiezen
3. Pad: `\\(server)\Homefolders\%username%`

**Roaming profiel instellen:**
- Profile path: `\\(server)\Roaming\%username%`

> 💡 `%username%` wordt automatisch vervangen door de inlognaam van elke gebruiker — zo krijgt iedereen automatisch een eigen map.

---

## Delegation of Control

> Gebruik dit om bijvoorbeeld een teamleider zelf wachtwoorden te laten resetten zonder volledige beheerdersrechten te geven.

**GUI (ADUC):**
1. Rechtermuisklik op OU → Delegate Control
2. Voeg gebruiker of groep toe
3. Kies taken (bijv. wachtwoorden resetten, gebruikers aanmaken)

---

## Handige PowerShell commando's
```powershell
# Alle gebruikers in een OU opvragen
Get-ADUser -Filter * -SearchBase "OU=Medewerkers,DC=bedrijf,DC=local"

# Gebruiker opzoeken
Get-ADUser -Identity "jan.jansen" -Properties *

# Wachtwoord resetten
Set-ADAccountPassword -Identity "jan.jansen" -Reset `
  -NewPassword (ConvertTo-SecureString "NieuwWw1!" -AsPlainText -Force)

# Account in-/uitschakelen
Disable-ADAccount -Identity "jan.jansen"
Enable-ADAccount  -Identity "jan.jansen"

# Gebruiker aan groep toevoegen
Add-ADGroupMember -Identity "IT-Beheer" -Members "jan.jansen"
```
