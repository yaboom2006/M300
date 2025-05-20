# Modul 300 Dokumentation

- [Modul 300 Dokumentation](#modul-300-dokumentation)
  - [Projektidee](#projektidee)
  - [VPC und Subnetz erstellen](#vpc-und-subnetz-erstellen)
  - [Security Groups erstellen](#security-groups-erstellen)
  - [Schlüsselpaar erstellen](#schlüsselpaar-erstellen)
  - [Elastic IPs erstellen](#elastic-ips-erstellen)
  - [Cloud Init erstellen](#cloud-init-erstellen)
  - [Startvorlage erstellen](#startvorlage-erstellen)

## Projektidee
Luca hatte die Idee eine Wetter-Webseite zu entwerfen, auf welcher man das Wetter abrufen kann. Ich wollte jedoch nicht das gleiche machen und habe darum überlegt was man sonst noch machen könnte. Ich habe ChatGPT um Hilfe gebeten, welche mir die Idee mit den Benzinpreisen gab.
Da ich Motorrad fahre fand ich das perfekt und vor allem auch nützlich, weil wir sowieso immer auf Tankstellen suche sind und dabei auch gerne den einen oder anderen Franken sparen würden.

## VPC und Subnetz erstellen
Als erstes habe ich das VPC erstellt welches ich für dieses Modul verwenden werde. Ich habe ganz standard ein /24 Netz genommen mit der Netz-IP 192.168.1.0. Der Name für das VPC lautet "M300 VPC".

## Security Groups erstellen
Die Security Groups sind ein sehr wichtiger bestandteil einer AWS Umgebung. Hier habe ich zwei neue Groups erstellt, eine für den Webserver und eine für den Datenbankserver. Beide Server haben ausgehend keine Einschränkungen, nur eingehend. Beim Webserver sind die Ports 80, 443 und 22 offen. Der Datenbankserver hat nur Port 22 und 1433 offen. Das werde ich jedoch anpassen müssen wenn ich merke dass etwas mit der Kommunikation zwischen den beiden Servern nicht funktioniert.

## Schlüsselpaar erstellen
Um den Zugriff auf die Instanzen ein bisschen zu sichern habe ich ein RSA Key erstellt und bei AWS Importiert.

## Elastic IPs erstellen
Um zu verhindern, dass die Server immer eine neue IP nach einem Neustart bekommen habe ich zwei Elastic IPs erstellt.

## Cloud Init erstellen
Das Cloud Init Script ist eher optional aber ich habe auch schon imemr damit gearbeitet darum dachte ich, mache ich das jetz auch so. Das Script holt und installiert alle neuen Updates. Zusätzlich installiert es curl, wget, git, htop, unzip, net-tools und ca-certificates.
```yaml
#cloud-config
package_update: true
package_upgrade: true
packages:
  - curl
  - wget
  - git
  - htop
  - unzip
  - net-tools
  - ca-certificates

runcmd:
  - timedatectl set-timezone Europe/Zurich
  - mkdir -p /var/log/init-setup
  - echo "Init-Script wurde ausgeführt am $(date)" > /var/log/init-setup/info.log

final_message: "EC2-Instanz ist vollständig vorbereitet und auf dem aktuellen Stand."
```

## Startvorlage erstellen
Das Cloud Init habe ich dann in eine Startvorlage gegeben. Ich werde zwei Vorlagen machen, eine für den Webserver und eine für den Datenbankserver. So kann ich immer wenn ich etwas neu machen muss, ganz schnell einen Server wieder starten.