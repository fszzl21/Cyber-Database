## Cheatsheet

| Command                                                                       | Description                                                                                                                       |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| ssh (sshuser)@(IP address)                                                    | Met een apparaat op host afstand verbinden                                                                                        |
| find (directory) -name (bestand die je zoekt) 2>/dev/null \| grep (zoekwoord) | Zoekt op in een directory met naam van de bestand die je zoekt en filtert de error weg en zoekt lijnen die de zoekwoord bevatten. |
[[Bekende Commands]]

### Belangerijke Folders

```
/etc/passwd
```
Bevat belangrijke gegevens over gebruikers in het systeem.

```
/etc/shadow
```
Bevat wachtwoord hashes. 
## Wat is het? 
Linux is naast windows en MacOS een [Operating system]. Het is vooral bekend om robust, flexibel en de open-sourceness.

Linux maakt gebruik van de shell. Dat is de main manier hoe je rond veel servers die linux systemen gebruiken rond kan komen. 

**Voorbeeld van een prompt**
```console
<username>@<hostname><current working directory>$
```

Bij prompt is het belangrijk of je een # of * weet. 

- (#) is prompt met privileges 
- ($) is prompt zonder privileges

### SSH
SSH (Secure Shell) is een protocol waar je veilig commands en en acties kan uitvoeren op remote computers op Linux gebaseerde hosts en servers.

om te verbinden gebruik je het volgende command.
```
irawnaf@host[/home]$ ssh <sshuser>@[IP address]
```

### File Descriptor en Redirections
File Descriptor is een getal dat uniek is aan een bestand of ander input/output bron, met als functie om elk onderdeel in een unix systeem te identificeren en connecties bij te volgen. 

Er zijn 3 soorten FD's 
1. Input = STDIN - 0
2. Output = STDOUT - 1
3. Output Error = STDERR - 2

**Redirections**
Ook is het mogelijk om de resultaten van de output in een bestand kan redirecten met 
```
find /etc/ -name shadow 2>/dev/null > bestandsnaam.txt
```

Ook kan je STOUT en STDERR in aparte bestanden zetten

```
find /etc/ -name shadow 2> sterr.txt 1> stdout.txt
```