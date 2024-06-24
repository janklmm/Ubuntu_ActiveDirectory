# Ubuntu 22.04 in eine Active Directory Domain einbinden

[Active Directory](#active-directory)
[Fehler AD](#probleme)

## Installation wichtiger Software


```bash
sudo apt-get install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```


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

```bash
cat /etc/resolv.conf
nameserver xxx.xxx.xxx.xxx
search fritz.box

sudo nano /etc/resolv.conf
nameserver xxx.xxx.xxx.xxx
search xxx.local
```
```bash
sudo systemctl status systemd-timesyncd
sudo nano /etc/systemd/timesyncd.conf

NTP=xxx.xxx.xxx.xxx
```
```bash
sudo systemctl restart systemd-timesyncd
```

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

Dieser Befehl fragt die Domain an, ob sie existiert oder nicht.
```bash
realm discover xxx.local
```
Informationen für das AD vorbereiten
```bash
sudo nano /etc/realmd.conf
```
```bash
[active-directory]
os-name = Ubuntu GNU/Linux
os-version = xx.xx (Versions Name) #welche Version gerade gentutz wird
```

Der Domain beitreten:
```bash
realm join -U USERNAME xxx.local
```
DOMAIN PASSWORT EINGEBEN 

Host in eine OU hhinzufügen
```bash
sudo realm join --verbose --user=Linux --computer-ou="OU=Ubuntu,OU=HQ,DC=jan,DC=local" jan.local
```
Überprüfen, ob man in der Domain ist und man zugriff hat, kann man mit
```bash
sudo realm list
```
```bash
id xxx@xxx.local
```
einen Benutzer der Domain abfragen, bekommt man eine Positive Antwort sieht es folgendermaßen aus:
<details>
<summary>id abfrage</summary>

![Screenshot 2024-06-24 205541](https://github.com/blvkf0rest/ubuntuad/assets/74656799/720a85f2-618b-4d46-9bd0-2e987431e78a)

</details>

Automatisches erstellen des HomeOrdners

```bash
sudo bash -c "cat > /usr/share/pam-configs/mkhomedir" <<EOF
Name: activate mkhomedir
Default: yes
Priority: 900
Session-Type: Additional
Session:
required pam_mkhomedir.so umask=0022 skel=/etc/skel
EOF
```
Aktivieren des HomeOrdners
```bash
sudo pam-auth-update
```
<details>
<summary>mkhomedir</summary>


</details>

Hier kann man sich dann mit dem Domain-Admin oder einen anderem Domain User anmelden. 
<details>
<summary>Domain anmeldung</summary>

![Screenshot 2024-06-24 210352](https://github.com/blvkf0rest/ubuntuad/assets/74656799/701dc40c-e2c1-485d-a3fa-9b685e975233)

</details>

Willkommen in der Domain!

Für den HostA Eintrag ist folgender Befehl zu benutzen:
```bash
 systemctl status sssd
```
Danach wird der Service neugestartet
```bash
service sssd restart
```

Danach ist der HostA Eintrag im DNS vom AD zu finden. 

Wechseln auf den Domain Nutzers
```bash
su - linux@jan.local
```
Danach sollte das Terminal so aussehen:

linux@jan.local@ubnt22:~$

Jeden Domain Nutzer erlauben sich mit dem Ubuntu Client zu verbinden:
```bash
sudo realm permit --all
oder es zu verbieten
sudo realm deny --all
```

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
