# Ubuntu 22.04 in eine Active Directory Domain einbinden


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
       addresses: [192.168.10.5]    # IP des DNS
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
der neuu Netplan erstellt werden.

## Fehler
Syntax Fehler:
```bash
ERROR:root:/etc/netplan/01-netcfg.yaml:3:12: Invalid YAML: inconsistent indentation: renderer: networkd
                                                                                             ^           
```
Bedeutet es, dass die Zeile bei "renderer: networkd" falsch gesetzt ist. 
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

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
