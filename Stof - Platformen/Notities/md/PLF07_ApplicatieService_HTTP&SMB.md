# Cheat Sheet – Week 07: Applicatieservices HTTP & SMB

> Geschreven door: Faisal Anwari 

---

## 1. Apache Webserver installeren (Rocky Linux)

### Installatie & Service starten

```bash
sudo dnf install httpd
sudo systemctl enable --now httpd
```
 

### Firewall openzetten voor HTTP

```bash
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

**Waarom?** Rocky Linux blokkeert standaard alle inkomende poorten. Zonder deze stap is je webserver onbereikbaar van buitenaf, ook al draait Apache zelf prima.

---

## 2. Website-directory aanmaken & rechten instellen

### Directory aanmaken en HTML-pagina plaatsen

```bash
sudo mkdir /var/www/(domeinnaam)
sudo nano /var/www/(domeinnaam)/index.html
```

### Bestandsrechten instellen (Apache mag lezen, niet schrijven)

```bash
sudo chown -R (eigenaar):(eigenaar) /var/www/
sudo chmod -R 755 /var/www/
```

**Waarom?** Met `755` heeft de eigenaar lees/schrijf/uitvoer-rechten, maar de Apache-user (`apache`) valt onder "anderen" en krijgt alleen leesrechten. Schrijfrechten voor Apache zouden een beveiligingsrisico zijn.

### Relevante bestanden & mappen

| Pad | Beschrijving |
|-----|--------------|
| `/var/www/` | Standaard webroot van Apache |
| `/var/www/(domeinnaam)/index.html` | Hoofdpagina van de website |
| `/etc/httpd/conf/httpd.conf` | Hoofdconfiguratie van Apache |
| `/etc/httpd/conf.d/` | Map voor Virtual Host configuratiebestanden |
| `/var/log/httpd/error_log` | Logbestand met Apache-foutmeldingen |

---

## 3. DNS CNAME record toevoegen

```bash
sudo nano /var/named/(domeinnaam).db
```

Voeg toe aan het zonebestand:

```
www    IN    CNAME    @
```

Vergeet niet het **serienummer** te verhogen, dan de zone herladen:

```bash
sudo named-checkzone (domeinnaam) /var/named/(domeinnaam).db
sudo systemctl reload named
```

**Waarom?** Een CNAME koppelt `www.bmc.example` aan de bestaande A-record van de server, zodat beide namen naar hetzelfde IP-adres verwijzen zonder dat je twee A-records hoeft bij te houden.

---

## 4. Virtual Host (vhost) aanmaken

```bash
sudo nano /etc/httpd/conf.d/(domeinnaam).conf
```

Inhoud van het configuratiebestand:

```apacheconf
<VirtualHost *:80>
    ServerName www.(domeinnaam)
    ServerAlias (domeinnaam)
    DocumentRoot /var/www/(domeinnaam)
</VirtualHost>
```

### Configuratie testen en Apache herstarten

```bash
apachectl configtest
sudo systemctl restart httpd
```

**Waarom?** Met `apachectl configtest` controleer je de syntaxis voordat je herstart. Op een productieserver kan een fout in de configuratie alle andere sites offline halen.

---

## 5. Webroot verplaatsen naar `/plf/www` (SELinux oefening)

### Nieuwe directorystructuur aanmaken

```bash
sudo mkdir -p /plf/www/html /plf/www/cgi-bin
sudo chmod -R 755 /plf
```

### Apache-configuratie aanpassen

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Gebruik in nano **Ctrl+W** om te zoeken, **Alt+R** om te zoeken én te vervangen.
Vervang alle vermeldingen van `/var/www` door `/plf/www`.

```bash
sudo systemctl restart httpd
```

**Waarom?** Door de webroot te verplaatsen naar een niet-standaardpad simuleer je een echte situatie waarbij SELinux ingrijpt — dit leert je hoe je beveiligingsconflicten structureel oplost.

### Testpagina aanmaken

```bash
sudo nano /plf/www/html/test.html
```

---

## 6. SELinux: Bestandscontext corrigeren

### Huidige context bekijken

```bash
ls -Z /var/www
ls -Z /plf/www
```

### Tijdelijke fix: context handmatig instellen

```bash
sudo chcon -R -t httpd_sys_content_t /plf/www
```

**Waarom?** SELinux controleert niet alleen bestandsrechten (`chmod`), maar ook beveiligingslabels (contexts). Apache mag alleen bestanden lezen met het label `httpd_sys_content_t`.

### Toegestane HTTP-poorten bekijken

```bash
sudo semanage port -l | grep http
```

### Context terugzetten naar standaard (om probleem te reproduceren)

```bash
sudo restorecon -R /plf/www
```

### Structurele fix: context permanent vastleggen

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/plf/www(/.*)?"
sudo restorecon -R -v /plf/www
```

**Waarom?** `chcon` is tijdelijk — bij een SELinux-relabel (b.v. na `restorecon`) gaat de context verloren. Met `semanage fcontext` leg je de regel permanent vast in de SELinux-database.

### SELinux-fouten analyseren

```bash
sudo grep http /var/log/audit/audit.log | audit2why
sudo tail -n 25 /var/log/httpd/error_log
```

### Relevante bestanden & mappen (SELinux)

| Pad | Beschrijving |
|-----|--------------|
| `/plf/www/html/` | Verplaatste webroot |
| `/plf/www/cgi-bin/` | CGI-scripts directory |
| `/var/log/audit/audit.log` | SELinux audit-logboek |
| `/var/log/httpd/error_log` | Apache foutmeldingen |

---

## 7. IIS installeren op Windows Server

### Via Server Manager (GUI)

1. Open **Server Manager**
2. Klik op **Manage → Add Roles and Features**
3. Kies **Web Server (IIS)**
4. Voeg toe: **Security-opties** (Basic Auth, Windows Auth, IP Restrictions) en **Management Service**
5. Voltooi de wizard

**Waarom?** De Management Service is nodig voor beheer op afstand (via IIS Manager of Remote IIS). Zonder deze rol kun je IIS alleen lokaal beheren.

### IIS installeren via PowerShell (alternatief)

```powershell
Import-Module ServerManager
Add-WindowsFeature Web-Server -IncludeAllSubFeature
```

### Remote management inschakelen op Core server

```powershell
Set-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\WebManagement\Server `
  -Name EnableRemoteManagement -Value 1
Net Stop WMSVC
Net Start WMSVC
```

**Waarom?** Een Windows Core server heeft geen GUI. Remote management via de bovenstaande registry-instelling maakt het mogelijk om IIS vanaf een andere machine te beheren.

---

## 8. IIS beveiligen (Default Web Site)

### IP-restrictie instellen (GUI)

1. Open **IIS Manager → Default Web Site → IP Address and Domain Restrictions**
2. Stel **Default access** in op **Deny**
3. Voeg toe: **Allow** `192.168.2.0/255.255.255.0`
4. Voeg toe: **Deny** `192.168.2.2`

**Waarom?** Met een "deny-all" standaard + expliciete allow-regels zorg je dat alleen bekende netwerken toegang krijgen — dit heet een whitelist-aanpak en is veiliger dan een blacklist.

### Authenticatie instellen (GUI)

1. **IIS Manager → Default Web Site → Authentication**
2. Schakel **Basic Authentication** in, stel realm in op `Basic authentication`
3. Daarna: schakel **Basic Authentication** uit en schakel **Windows Authentication** in

**Waarom?** Basic Authentication stuurt het wachtwoord (base64-gecodeerd maar *niet* versleuteld) mee met elk request. Windows Authentication gebruikt Kerberos of NTLM en stuurt nooit het wachtwoord zelf over het netwerk.

### Logging uitbreiden (GUI)

1. **IIS Manager → Default Web Site → Logging**
2. Klik op **Select Fields** en activeer **alle beschikbare velden**

---

## 9. Windows Shared Folder aanmaken

### Via GUI (op DC)

1. Maak de map aan: `C:\data\share1`
2. Rechtsklik → **Properties → Security** tab:
   - **Users**: Read & Execute
   - **Administrators**: Full Control
3. Rechtsklik → **Properties → Sharing** tab → **Advanced Sharing**
4. Vink aan: **Share this folder**, naam: `share1`
5. Klik op **Permissions**: stel **Read** in voor **Everyone**

**Waarom?** Er zijn twee lagen rechten: NTFS-rechten (op het bestandssysteem) en Share-rechten (op het netwerk). De meest beperkende combinatie wint — daarom mag een administrator op de share volledig, maar worden ze op netwerkniveau teruggebracht naar Read.

### Via PowerShell (op Core server)

```powershell
New-SmbShare -Name "share1" -Path "C:\data\share1" -FullAccess "Administrators" -ReadAccess "Users"
Get-SmbShare
```

**Waarom?** Op een Core server zonder GUI is PowerShell de enige manier. `Get-SmbShare` toont alle gedeelde mappen, inclusief de verborgen admin-shares zoals `C$` en `ADMIN$`.

### Logfile kopiëren naar share (Core)

```powershell
Copy-Item "C:\inetpub\logs\LogFiles\W3SVC1\(logbestand)" "C:\data\share1\"
```

---

## 10. Samba installeren en configureren (Rocky Linux)

### Installatie & Service starten

```bash
sudo dnf install samba*
sudo systemctl enable --now smb nmb
```

**Waarom?** Samba bestaat uit meerdere daemons: `smbd` verzorgt file sharing en authenticatie, `nmbd` regelt NetBIOS-naamresolutie zodat Windows de server kan vinden op naam.

### Originele config bewaren en nieuwe aanmaken

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf~
sudo nano /etc/samba/smb.conf
```

Inhoud van de nieuwe `smb.conf`:

```ini
[global]
workgroup = (WERKGROEP)
server string = Samba Server %v
netbios name = (SERVERNAAM)
security = user
map to guest = bad user
dns proxy = no

#==================== Share Definitions ======================
[secure]
path = /home/secure
valid users = @smbgrp
guest ok = no
writable = yes
browsable = yes
```

**Waarom?** Door de originele config te hernoemen met `~` heb je altijd een backup als je configuratie niet werkt. De `[global]` sectie bepaalt het gedrag van de hele server; de `[secure]` sectie definieert de gedeelde map.

### Gebruiker en groep aanmaken

```bash
sudo useradd (gebruikersnaam)
sudo groupadd smbgrp
sudo usermod -a -G smbgrp (gebruikersnaam)
sudo smbpasswd -a (gebruikersnaam)
```

**Waarom?** Samba heeft een apart wachtwoordbestand naast het Linux-wachtwoord. `smbpasswd -a` voegt de gebruiker toe aan de Samba-database — zonder dit stap kan de gebruiker niet inloggen via het netwerk.

### Rechten op de gedeelde directory instellen

```bash
sudo mkdir /home/secure
sudo chown -R (gebruikersnaam):smbgrp /home/secure/
sudo chmod -R 0770 /home/secure/
sudo chcon -t samba_share_t /home/secure/
```

**Waarom?** `chmod 0770` geeft de eigenaar én groepsleden volledige rechten, maar blokkeert iedereen buiten de groep. De `chcon` regel is noodzakelijk voor SELinux — zonder het juiste label `samba_share_t` zal SELinux de toegang blokkeren.

### Firewall openzetten voor Samba

```bash
sudo firewall-cmd --add-service=samba
sudo firewall-cmd --runtime-to-permanent
```

### Samba services herstarten

```bash
sudo systemctl restart smb nmb
```

### Samba-configuratie controleren

```bash
testparm
```

**Waarom?** `testparm` controleert `/etc/samba/smb.conf` op syntaxisfouten en toont de resulterende shares — vergelijkbaar met `apachectl configtest` voor Apache.

### Relevante bestanden & mappen (Samba)

| Pad | Beschrijving |
|-----|--------------|
| `/etc/samba/smb.conf` | Hoofd-configuratiebestand van Samba |
| `/etc/samba/smb.conf~` | Backup van de originele configuratie |
| `/home/secure/` | De gedeelde directory |

---

## 11. Samba-share benaderen vanuit Windows (client)

1. Open **Windows Explorer**
2. Rechtsklik op **This PC** → **Map network drive...**
3. Voer als sharenaam in: `\\(servernaam)\secure`
4. Deselecteer **Reconnect at sign-in**
5. Selecteer **Connect using different credentials**
6. Voer de credentials in van de Samba-gebruiker (`(gebruikersnaam)` + Samba-wachtwoord)

**Waarom?** Als de Windows-client lid is van een Windows-domein, zal Windows automatisch proberen in te loggen met de domeinaccount. Door "different credentials" te kiezen, forceer je het gebruik van de Samba-gebruiker op de Linux-server.
