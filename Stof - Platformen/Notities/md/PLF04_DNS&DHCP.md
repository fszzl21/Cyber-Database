# Cheat Sheet – Week 04: Core Network Services (DHCP & DNS)

> Geschreven door: Faisal Anwari

---

## 1. Linux DHCP server installeren

```bash
sudo dnf install dhcp-server
```

**Tip:** Na installatie vind je een uitgebreid voorbeeldbestand op `less /usr/share/doc/dhcp*/dhcpd.conf.example`. Dit is handig als startpunt of ter referentie voor de syntax.

---

## 2. Linux DHCP server configureren

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Minimale configuratie:

```
option domain-name "(domeinnaam)";
option domain-name-servers (DNS-servernaam of IP);

default-lease-time 600;
max-lease-time 7200;

subnet (netwerkadres) netmask (subnetmask) {
    range (start-IP) (eind-IP);
    option routers (gateway-IP);
    option broadcast-address (broadcast-IP);
}

# Voorkomt waarschuwingen in de logs voor interfaces zonder scope:
subnet (ander-subnet) netmask (subnetmask) {
}
```

**Voorbeeld voor bmc.example:**

```
option domain-name "bmc.example";
option domain-name-servers rocky1.bmc.example;

default-lease-time 600;
max-lease-time 7200;

subnet 192.168.15.0 netmask 255.255.255.0 {
    range 192.168.15.100 192.168.15.150;
    option routers 192.168.15.254;
    option broadcast-address 192.168.15.255;
}
```

**Tip:** Elke `option`- en `range`-regel moet eindigen met een puntkomma (`;`). Dit is de meest voorkomende reden waarom `dhcpd` niet start.

### Relevante bestanden en mappen (DHCP Linux)

| Pad | Beschrijving |
|-----|--------------|
| `/etc/dhcp/dhcpd.conf` | Configuratiebestand van de DHCP-server |
| `/usr/share/doc/dhcp*/dhcpd.conf.example` | Voorbeeldconfiguratiebestand |
| `/var/lib/dhcpd/dhcpd.leases` | Database van uitgegeven IP-adressen |

---

## 3. Linux DHCP server configuratie controleren, starten en firewall

```bash
dhcpd -cf /etc/dhcp/dhcpd.conf
sudo firewall-cmd --add-service=dhcp
sudo firewall-cmd --runtime-to-permanent
sudo systemctl enable --now dhcpd
```

**Tip:** `dhcpd -cf` controleert de syntaxis van het configuratiebestand zonder de service te starten. Doe dit altijd voor je de service herstart, zodat je weet of een fout in de config zit.

---

## 4. Linux DHCP leases bekijken

```bash
tail /var/lib/dhcpd/dhcpd.leases
systemctl status dhcpd -l
```

**Tip:** Het leases-bestand toont welke IP-adressen zijn uitgedeeld, aan welk MAC-adres en tot wanneer. Dit is de snelste manier om te controleren of de DHCP-server daadwerkelijk adressen uitgeeft.

---

## 5. Windows DHCP server installeren

### Via Server Manager (GUI op DC)

1. Open **Server Manager**
2. Kies **Manage → Add Roles and Features**
3. Selecteer de rol **DHCP Server**
4. Voltooi de wizard en autoriseer de DHCP-server in Active Directory via de melding die na installatie verschijnt

### Via PowerShell (op Core server of DC)

```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

### DHCP-beheertools installeren op GUI-server (voor remote beheer)

```powershell
Install-WindowsFeature -Name RSAT-DHCP
```

**Tip:** Een Windows DHCP-server moet worden geautoriseerd in Active Directory voordat hij adressen uitgeeft. Zonder autorisatie heeft de server een rood kruisje in de DHCP-beheerconsole en doet hij niets. Dit beschermt het netwerk tegen onbevoegde DHCP-servers.

---

## 6. Windows DHCP server configureren (via MMC of DHCP-console)

1. Open **DHCP** via Server Manager of via `mmc.exe` met de DHCP-snap-in
2. Verbind met de juiste server
3. Rechtsklik op **IPv4 → New Scope**
4. Vul in:
   - IP-range: bijv. `192.168.14.100` t/m `192.168.14.199`, subnetmasker `/24`
   - Optie **003 Router**: gateway-IP
   - Optie **006 DNS Server**: IP van de DNS-server
   - Optie **015 Domain Name**: bijv. `bmc.test`
5. Activeer de scope aan het einde van de wizard

**Tip:** De opties 003, 006 en 015 zijn gedefinieerd in RFC 2132 en vertellen de client niet alleen een IP-adres, maar ook gateway, DNS-server en domeinnaam. Zonder deze opties werkt DHCP maar heeft de client geen werkende naam­resolutie of routing.

---

## 7. DHCP testen op Windows client

```cmd
ipconfig /renew Ethernet1
ipconfig /all
```

**Tip:** `ipconfig /all` toont bij elk adres ook de DHCP-server die het heeft uitgegeven. Zo controleer je in een netwerk met meerdere DHCP-servers (Linux en Windows) welke server welke client heeft bediend.

---

## 8. Windows DNS – Round Robin instellen (DNS Manager)

1. Open **DNS Manager** op de Domain Controller
2. Navigeer naar de gewenste zone, bijv. `bmc.test`
3. Maak drie A-records aan met dezelfde naam maar verschillende IP-adressen:
   - `home` → `1.0.0.1`
   - `home` → `1.0.0.2`
   - `home` → `1.0.0.3`

Testen op de client:

```cmd
ipconfig /flushdns
ping home.bmc.test
ipconfig /flushdns
ping home.bmc.test
```

**Tip:** DNS Round Robin werkt doordat de DNS-server bij elke query de volgorde van de A-records roteert. De DNS-cache op de client bewaart het antwoord, daarom is `ipconfig /flushdns` nodig tussen de tests om steeds een verse query te forceren.

---

## 9. Linux DNS server (BIND) installeren

```bash
sudo dnf install bind bind-utils
```

**Tip:** `bind` is de daadwerkelijke DNS-server (ook wel `named` genoemd). `bind-utils` levert de testgereedschappen `dig`, `nslookup` en `host`. Installeer beide altijd tegelijk.

---

## 10. Linux DNS server configureren: named.conf

```bash
sudo nano /etc/named.conf
```

Pas de `options`-sectie aan:

```
options {
    listen-on port 53 { 127.0.0.1; (server-IP); };
    allow-query     { localhost; (netwerk/prefix); };
    allow-transfer  { localhost; (slave-DNS-IP); };
};
```

Voeg onderaan een zone-definitie toe:

```
zone "(domeinnaam)" IN {
    type master;
    file "/var/named/(domeinnaam).db";
    notify yes;
};
```

**Tip:** `listen-on` bepaalt op welke IP-adressen de DNS-server luistert. `allow-query` bepaalt wie vragen mag stellen. Gebruik nooit `0.0.0.0` maar `any` als je alle adressen wilt toestaan. Een vergeten subnet hier is de meest voorkomende reden waarom clients geen antwoord krijgen terwijl de server zelf wel werkt.

### Relevante bestanden en mappen (DNS Linux)

| Pad | Beschrijving |
|-----|--------------|
| `/etc/named.conf` | Hoofdconfiguratie van BIND |
| `/var/named/` | Map met zone-databasebestanden |
| `/var/named/named.empty` | Leeg zone-bestand als startpunt |

---

## 11. Linux DNS forward zone aanmaken

```bash
sudo cp /var/named/named.empty /var/named/(domeinnaam).db
sudo nano /var/named/(domeinnaam).db
```

Structuur van een zone-bestand:

```
$TTL 300

@   IN  SOA  (primaire-NS). hostmaster.(domeinnaam). (
                (YYYYMMDD##)  ; Serial
                3M            ; Refresh
                1M            ; Retry
                30D           ; Expire
                1M )          ; Minimum TTL (negatieve cache)

; Nameserver record
    IN  NS  (primaire-NS).

; A-records
(primaire-NS kortform)  IN  A  (IP-adres-server)
(hostnaam)              IN  A  (IP-adres)

; Alias
www                     IN  CNAME  (hostnaam).
```

**Voorbeeld voor bmc.example:**

```
$TTL 300

@   IN  SOA  rocky1.bmc.example. hostmaster.bmc.example. (
                2024010601    ; Serial
                3M            ; Refresh
                1M            ; Retry
                30D           ; Expire
                1M )          ; Minimum

    IN  NS  rocky1.bmc.example.

rocky1  IN  A  192.168.15.1
dc1     IN  A  192.168.14.1
core1   IN  A  192.168.14.4
```

**Tip:** Let op de punt aan het einde van elke Fully Qualified Domain Name (FQDN) zoals `rocky1.bmc.example.`. Zonder die punt wordt de domeinnaam er automatisch achter geplakt, wat leidt tot `rocky1.bmc.example.bmc.example.` — een veelgemaakte fout die moeilijk te zien is maar direct fout gaat.

Het serienummer in het formaat `YYYYMMDD##` maakt het makkelijk bij te houden wanneer de laatste wijziging was. Verhoog het getal bij elke aanpassing, anders weten slave DNS-servers niet dat er iets veranderd is.

---

## 12. Linux DNS reverse zone aanmaken

Voeg toe aan `/etc/named.conf`:

```bash
sudo nano /etc/named.conf
```

```
zone "(omgekeerde-octetten).in-addr.arpa" IN {
    type master;
    file "/var/named/(bestandsnaam).db";
};
```

**Voorbeeld voor 192.168.15.0/24:**

```
zone "15.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/192.168.15.db";
};
```

Maak het zone-bestand aan:

```bash
sudo nano /var/named/192.168.15.db
```

```
$TTL 300

@   IN  SOA  rocky1.bmc.example. hostmaster.bmc.example. (
                2024010601
                3M
                1M
                30D
                1M )

    IN  NS  rocky1.bmc.example.

1   IN  PTR  rocky1.bmc.example.
4   IN  PTR  core1.bmc.example.
254 IN  PTR  vyos.bmc.example.
```

**Tip:** In een reverse zone schrijf je alleen het laatste octet van het IP-adres als naam van het PTR-record. Dus voor `192.168.15.1` schrijf je `1 IN PTR rocky1.bmc.example.`. De rest van het adres zit al opgesloten in de zone-naam `15.168.192.in-addr.arpa`.

Reverse DNS is nuttig voor traceroute (toont hostnamen), voor logging (leesbare namen in logbestanden) en wordt door sommige services gecontroleerd als extra authenticatie.

---

## 13. Linux DNS configuratie controleren

```bash
sudo -u named named-checkconf /etc/named.conf
sudo -u named named-checkzone (domeinnaam) /var/named/(domeinnaam).db
sudo -u named named-checkconf -z
```

**Tip:** `sudo -u named` voert de controle uit als de `named`-gebruiker. Dit heeft als voordeel dat rechtenfouten (bijv. het zone-bestand is niet leesbaar voor `named`) direct zichtbaar worden als "permission denied" in de output. Als je het als root uitvoert, zie je deze rechtenfouten niet.
Ook als je error krijgt van permission denied probeer dan het volgende commands.
```bash
chwon named:named /var/named/(domeinnaam).db
chmod 640 /var/named/(domeinnaam).db
```
---

## 14. Linux DNS firewall, opstarten en herladen

```bash
sudo firewall-cmd --add-service=dns
sudo firewall-cmd --runtime-to-permanent
sudo systemctl enable --now named
sudo systemctl reload named
sudo systemctl status named
```

**Tip:** Gebruik `reload` in plaats van `restart` om een gewijzigde configuratie actief te maken. Bij `reload` worden bestaande verbindingen niet verbroken. `restart` stopt de service volledig en start hem opnieuw, wat een korte onderbreking veroorzaakt.

---

## 15. Linux DNS testen

```bash
dig +noall +answer (hostnaam) @127.0.0.1
dig +noall +answer (hostnaam) @(server-IP)
dig +noall +answer -x (IP-adres) @127.0.0.1
nslookup (hostnaam)
host (hostnaam)
```

**Tip:** Met `@(IP-adres)` geef je aan welke DNS-server je wilt bevragen. Dit is essentieel voor testen: zo weet je zeker dat jouw server antwoordt en niet de DNS-server van het besturingssysteem. Test altijd eerst op `@127.0.0.1` (lokaal) en daarna op het externe IP-adres.

---

## 16. Linux DNS client instellen

```bash
sudo nano /etc/resolv.conf
```

```
search (domeinnaam)
nameserver 127.0.0.1
```

Volgorde van naamresolutie bekijken:

```bash
grep hosts /etc/nsswitch.conf
```

De regel `hosts: files dns` betekent: kijk eerst in `/etc/hosts`, dan pas in DNS.

**Tip:** Na het aanpassen van de netwerkinterface-instellingen kan `/etc/resolv.conf` automatisch worden overschreven door NetworkManager. Controleer het bestand altijd opnieuw nadat je het interface hebt gedeactiveerd en geactiveerd.
