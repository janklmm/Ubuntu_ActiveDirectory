# Ubuntu 22.04 in eine Active Directory Domain einbinden

[Active Directory](#active-directory)
[Fehler AD](#probleme)

## Erstellen der Statischen IP & IPv6 deaktivieren

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
Es ist wichtig, das sich an die Syntax gehalten wird, sollte es nicht passen, werden Fehler bei der Ausgabe angezeigt. Sollte das der Fall sein, müssen die Leerzeichen angepasst werden. 

Hier wird folgendes Eingetragen:
"#" steht für ein Kommentar


```bash
network:
  version: 2
  renderer: networkd
  ethernets:
   #je nachdem was für eine NetzwerkID vergeben wurde 0, 1, 2 etc
   eth0:
     link-local: [ipv4]
     dhcp4: no
     addresses: [192.168.10.11/24]  # Die IP, des Clients
     routes:                        # Routes anstatt Gateway
       - to: default
         via: 192.168.10.5
     nameservers:
       addresses:
       - 192.168.10.5    # IP des DNS
       search:
       - "xxx.local"
```
Wenn die Datei fertig geschrieben ist, mit STRG+X schließen. 
Es wird gefragt ob man die Datei speichern will, das mit "Y" bestätigen, als nächstes wird gefragt unter welchem Namen die Datei gespeichert werden soll. Das mit Enter bestätigen.

IPv6 deaktivieren
```bash
sudo nano /etc/netplan/99-disable-link-local.yaml
```
```bash
network:
  version: 2
  renderer: networkd
  ethernets:
   #je nachdem was für eine NetzwerkID vergeben wurde 0, 1, 2 etc
   eth0:
     link-local: []
```
Speichern und schließen
```bash
sudo nano /etc/sysctl.d/99-ipv6-disable.conf

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
Speichern und schließen


Zum Überprüfen ob die Einstellungen korrekt sind, wird folgender Befehl benutzt
```bash
sudo netplan try
```
Sollten hier keine Fehler auftreten, kann mit 
```bash
sudo netplan apply
```
der neu Netplan wird erstellt.

Der Ubuntu Client, sollte jetzt neugestartet werden. Entweder über das Terminal mit ** reboot ** oder über die GUI

Nach dem Neustart kann über das Terminal mit ** ip a ** die IP Adresse abgefragt werden, oder über die GUI Einstellungen -> Netzwerk -> auf das Zahnrad

<details>
<summary>Bilder IP Config</summary>
  
![Screenshot 2024-06-24 203219](https://github.com/blvkf0rest/ubuntuad/assets/74656799/b1da95a2-968f-473f-aa1f-d4a8b1fc987a)

![Screenshot 2024-06-24 203358](https://github.com/blvkf0rest/ubuntuad/assets/74656799/400d4d87-8cd7-42b7-8880-5e6c1f58e260)

</details>

## Fehler
Syntax Fehler:
```bash
ERROR:root:/etc/netplan/01-netcfg.yaml:3:12: Invalid YAML: inconsistent indentation: renderer: networkd
                                                                                             ^           
```
Bedeutet
 es, dass die Zeile bei "renderer: networkd" falsch gesetzt ist. 
Beispiel: 
```bash
network:
  version: 2
   renderer: networkd 
  ethernets:
**.... **
```
Es ist ein Leerzeichen zuviel!

Berechtigungs Fehler:
```bash
** (process:xxxx): WARNING **: ... : Permissions for /etc/netplan/01-netcfg.yaml are too open. Netplan configuration should NOT be accessible by others.
```

Um diesen Fehler zu beheben nutzen wir chmod
```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
```
Die Datei hat jetzt folgende Berechtigung: rw- --- ---

[chmod zum selber nachlesen](https://www.linode.com/docs/guides/modify-file-permissions-with-chmod/)

# Active Directory

Um Ubuntu in eine Windows Domain einzubinden, werden noch einige zusätzliche Pakete benötigt. 

```bash
sudo apt-get install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```
Dieser Befehl fragt die Domain an, ob sie existiert oder nicht.
```bash
realm discover xxx.local
```
Der Domain beitreten:
```bash
realm join xxx.local
```
Man wird aufgefordert ein Passwort einzugeben, es ist das Passwort eines Domain-Admins. 
Nach der eingabe, wird man aufgefordert das Lokale Passwort einzugeben. 

Sollte die Eingabe BEIDER Passwörter erfolgreich sein, passiert nichts. 

Um zu Überprüfen, ob man in der Domain ist und man zugriff hat, kann man mit
```bash
id xxx@xxx.local
```
einen Benutzer der Domain abfragen, bekommt man eine Positive Antwort sieht es folgendermaßen aus:
<details>
<summary>id abfrage</summary>

![Screenshot 2024-06-24 205541](https://github.com/blvkf0rest/ubuntuad/assets/74656799/720a85f2-618b-4d46-9bd0-2e987431e78a)

</details>

Der Lokale Benutzer kann nun abgemeldet werden. 
Im Anmeldebildschirm hat man die Möglichkeit, einen neuen Benutzer hinzuzufügen indem man auf "not listed?" klickt. 

Hier kann man sich dann mit dem Domain-Admin oder einen anderem Domain User anmelden. 
<details>
<summary>Domain anmeldung</summary>

![Screenshot 2024-06-24 210352](https://github.com/blvkf0rest/ubuntuad/assets/74656799/701dc40c-e2c1-485d-a3fa-9b685e975233)

</details>

Willkommen in der Domain!

Für den HostA Eintrag ist folgender Befehl zu benutzen:
```bash
 hostnamectl set-hostname CLIENTNAME.DOMAIN.local
```
Danach wird der Service neugestartet
```bash
service sssd restart
```

Danach ist der HostA Eintrag im DNS vom AD zu finden. 


# Probleme

nslookup funktioniert über Ubuntu nicht. 
![Screenshot 2024-06-24 215629](https://github.com/blvkf0rest/ubuntuad/assets/74656799/1b360a39-d480-46bc-94b2-5a704cb84001)

über den AD geht es.

![Screenshot 2024-06-24 215712](https://github.com/blvkf0rest/ubuntuad/assets/74656799/90384654-4bbf-4631-9b5f-8ad105fa6575)


```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
