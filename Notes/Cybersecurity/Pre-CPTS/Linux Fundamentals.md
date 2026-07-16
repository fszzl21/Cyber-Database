## Cheatsheet
| Command                    | Description                                |
| -------------------------- | ------------------------------------------ |
| ssh (sshuser)@(IP address) | Met een apparaat op host afstand verbinden |
| Cell 1, Row 2              | Cell 1, Row 2                              |
[[Bekende Commands]]
#### Wat is het? 
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

