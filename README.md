# Ubuntu 22.04 in eine Active Directory Domain einbinden

[Active Directory](#active-directory)

[Fehler AD](#probleme)

## Installation wichtiger Software


```bash
sudo apt-get install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

## Sudo für Domain-Admins

Führt man einen Sudo-Befehl als Domain-Admin ohne Sudo rechte ausführen, kommt folgende Meldung:

"linux@xxx.local ist nicht in der sudoers-Datei. Dieser Vorfall wird gemeldet."

Aus diesen Grund müssen vorbereitungen getroffen werden.
```bash
sudo nano /etc/sudoers
```
```bash
# User privilege specification
root                   ALL=(ALL:ALL) ALL
%LinuxAdmins@xxx.local ALL=(ALL:ALL) ALL
```
Wenn die Datei fertig geschrieben ist, mit STRG+X schließen. 
Es wird gefragt ob man die Datei speichern will, das mit "Y" bestätigen, als nächstes wird gefragt unter welchem Namen die Datei gespeichert werden soll. Dies mit Enter bestätigen. 
(So werden alle Datein, wenn sie mit 'Nano' geöffnet wurden, gespeichert)

Auf dem AD brauchen wir eine Sicherheitsgruppe mit GENAU dem selben Namen:

"LinuxAdmins"

In diese Gruppe kommen dann die Domain-Admins

Damit ist die Vorbereitung der Sudo-Rechte fertig


## Erstellen der Statischen IP & IPv6 deaktivieren

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
Es ist wichtig, das sich an die Syntax gehalten wird, sollte es nicht passen, werden Fehler bei der Ausgabe angezeigt. Sollte das der Fall sein, müssen die Leerzeichen angepasst werden. 

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
     routes:                        # Routes anstatt Gateway,
       - to: default                # da Gateway von Netplan nicht genutzt wird und fehler auftreten können
         via: 192.168.10.5
     nameservers:
       addresses:
       - 192.168.10.5    # IP des DNS
       search:
       - "xxx.local"
```
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
Hier kann eine fehlermeldung erscheinen, dieses kann ignoriert werden. [Klick-Bug](https://ubuntuforums.org/showthread.php?t=2495406&p=14179792#post14179792)
```bash
WARNING:root:Cannot call Open vSwitch: ovsdb-server.service is not running.
```

Sollten hier KEINE (bis auf den oben genannten) Fehler auftreten, kann mit 
```bash
sudo netplan apply
```
der neue Netplan erstellt werden.

```bash
sudo nano /etc/systemd/resolved.conf
#DNS=xxx.xxx.xxx.xxx
(#FallbackDNS=8.8.8.8)

sudo systemctl restart systemd-resolved

______nicht nutzen_______
(sudo nano /etc/resolv.conf)
(search xxx.local)
```
```bash
sudo systemctl status systemd-timesyncd
sudo nano /etc/systemd/timesyncd.conf

NTP=xxx.xxx.xxx.xxx #AD IP
```
```bash
sudo systemctl restart systemd-timesyncd
```

Nach dem Neustart kann über das Terminal mit ** ip a ** die IP Adresse abgefragt werden, oder über die GUI Einstellungen -> Netzwerk -> auf das Zahnrad

<details>
<summary>Bilder IP Config</summary>
  
![Screenshot 2024-06-24 203219](https://github.com/blvkf0rest/ubuntuad/assets/74656799/b1da95a2-968f-473f-aa1f-d4a8b1fc987a)

![Screenshot 2024-06-24 203358](https://github.com/blvkf0rest/ubuntuad/assets/74656799/400d4d87-8cd7-42b7-8880-5e6c1f58e260)

</details>

## Fehler

<details>
  <summary>Lösung</summary>
  
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

</details>

# Active Directory

Überprüfen des aktuellen Host Namen
```bash
hostnamectl
```
Setzen des Host Namen
```bash
sudo hostnamectl set-hostname xxx.xxx.local
```

Dieser Befehl fragt die Domain ab, ob sie existiert oder nicht.
```bash
realm discover xxx.local
```
Dieser Befehl fragt die Domain ab, ob sie existiert oder nicht.
```bash
realm discover xxx.local --verbose
```
Unterschied zwischen --verbose und ohne
<details>
<summary>VERBOSE</summary>
  
![verbose](https://github.com/blvkf0rest/ubuntuad/assets/74656799/5f3f409b-3ab4-45e0-899c-23fbcae6cbe3) ![ohneverbose](https://github.com/blvkf0rest/ubuntuad/assets/74656799/116cc614-f5f5-4ed5-9b5b-9b16072183e3)

</details>


Der Domain beitreten:
```bash
sudo realm join --user=USERNAME -v xxx.local
```
DOMAIN PASSWORT EINGEBEN 

Überprüfen, ob man in der Domain ist und man zugriff hat, kann man mit
```bash
sudo realm list
```
```bash
id xxx@xxx.local
```
Willkommen in der Domain!

## Erstellen des HomeOrdners

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
Aktivieren des HomeOrdners. (Mit Tab navigieren, mit Leertaste auswählen, mit Enter beenden)
```bash
sudo pam-auth-update
```
<details>
<summary>mkhomedir</summary>

  ![Screenshot 2024-06-25 015600](https://github.com/blvkf0rest/ubuntuad/assets/74656799/90ce55dd-10e2-4391-940c-c34e306a351a)
</details>

Der HostA sowie der Reverse Eintrag sind nun im AD hinterlegt 

Wechseln auf den Domain Nutzers
```bash
su - linux@jan.local
```
Danach sollte das Terminal so aussehen:

linux@jan.local@xxxx:~$

Jeden Domain Nutzer erlauben sich mit dem Ubuntu Client zu verbinden:
```bash
sudo realm permit --all
oder es zu verbieten
sudo realm deny --all
```



# Probleme


nslookup funktioniert anscheinend über Ubuntu nicht. 
(Problem: die /etc/resolv. sollte nicht bearbeitet werden da sie sich nach jedem neustart selber schreibt)

![Screenshot 2024-06-24 215629](https://github.com/blvkf0rest/ubuntuad/assets/74656799/1b360a39-d480-46bc-94b2-5a704cb84001)

nslookup über AD ohne probleme

![Screenshot 2024-06-24 215712](https://github.com/blvkf0rest/ubuntuad/assets/74656799/90384654-4bbf-4631-9b5f-8ad105fa6575)





## Quellen

 - [Server-World](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=realmd)
 - [Netplan](https://netplan.readthedocs.io/en/stable/)
 - [Youtube](https://www.youtube.com/watch?v=dQw4w9WgXcQ)
 - [PowerAdm](https://poweradm.com/join-ubuntu-debian-active-directory-domain/#:~:text=In%20order%20to%20join%20a%20Linux%20host%20to,-U%20k.muller%20poweradm.com%20Enter%20the%20domain%20user%20password.)



_____________________________________________
## Alt, Test, nicht nutzen

altes zeug

Informationen für das AD vorbereiten
```bash
sudo nano /etc/realmd.conf
```
```bash
[active-directory]
os-name = Ubuntu GNU/Linux
os-version = xx.xx (Versions Name) #welche Version gerade gentutz wird
```
Für den HostA Eintrag ist folgender Befehl zu benutzen:
```bash
 systemctl status sssd
```
Danach wird der Service neugestartet
```bash
service sssd restart
```
```bash
_______ALT NICHT NUTZEN
cat /etc/resolv.conf
nameserver xxx.xxx.xxx.xxx
search fritz.box

sudo nano /etc/resolv.conf
nameserver xxx.xxx.xxx.xxx
search xxx.local
_______
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
