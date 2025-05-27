# Modul 300 Dokumentation

- [Modul 300 Dokumentation](#modul-300-dokumentation)
  - [AWS Umgebung](#aws-umgebung)
    - [Projektidee](#projektidee)
    - [VPC und Subnetz erstellen](#vpc-und-subnetz-erstellen)
    - [Security Groups erstellen](#security-groups-erstellen)
    - [Schlüsselpaar erstellen](#schlüsselpaar-erstellen)
    - [Elastic IPs erstellen](#elastic-ips-erstellen)
    - [Cloud Init erstellen](#cloud-init-erstellen)
    - [Startvorlage erstellen](#startvorlage-erstellen)
  - [Webserver](#webserver)
    - [System updaten, Node.js und Nginx installieren](#system-updaten-nodejs-und-nginx-installieren)
    - [Nginx installieren](#nginx-installieren)
    - [Backend aufsetzen](#backend-aufsetzen)
    - [pm2 installlieren und konfigurieren](#pm2-installlieren-und-konfigurieren)
  - [HTTPS Zertifikat erstellen](#https-zertifikat-erstellen)

## AWS Umgebung

### Projektidee
Luca hatte die Idee eine Wetter-Webseite zu entwerfen, auf welcher man das Wetter abrufen kann. Ich wollte jedoch nicht das gleiche machen und habe darum überlegt was man sonst noch machen könnte. Ich habe ChatGPT um Hilfe gebeten, welche mir die Idee mit den Benzinpreisen gab.
Da ich Motorrad fahre fand ich das perfekt und vor allem auch nützlich, weil wir sowieso immer auf Tankstellen suche sind und dabei auch gerne den einen oder anderen Franken sparen würden.

### VPC und Subnetz erstellen
Als erstes habe ich das VPC erstellt welches ich für dieses Modul verwenden werde. Ich habe ganz standard ein /24 Netz genommen mit der Netz-IP 192.168.1.0. Der Name für das VPC lautet "M300 VPC".

### Security Groups erstellen
Die Security Groups sind ein sehr wichtiger bestandteil einer AWS Umgebung. Hier habe ich zwei neue Groups erstellt, eine für den Webserver und eine für den Datenbankserver. Beide Server haben ausgehend keine Einschränkungen, nur eingehend. Beim Webserver sind die Ports 80, 443 und 22 offen. Der Datenbankserver hat nur Port 22 und 1433 offen. Das werde ich jedoch anpassen müssen wenn ich merke dass etwas mit der Kommunikation zwischen den beiden Servern nicht funktioniert.

### Schlüsselpaar erstellen
Um den Zugriff auf die Instanzen ein bisschen zu sichern habe ich ein RSA Key erstellt und bei AWS Importiert.

### Elastic IPs erstellen
Um zu verhindern, dass die Server immer eine neue IP nach einem Neustart bekommen habe ich zwei Elastic IPs erstellt.

### Cloud Init erstellen
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

### Startvorlage erstellen
Das Cloud Init habe ich dann in eine Startvorlage gegeben. Ich werde zwei Vorlagen machen, eine für den Webserver und eine für den Datenbankserver. So kann ich immer wenn ich etwas neu machen muss, ganz schnell einen Server wieder starten.

## Webserver

### System updaten, Node.js und Nginx installieren
```bash
sudo apt update && sudo apt upgrade -y
```

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### Nginx installieren
```bash
sudo apt install -y nginx
```

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Backend aufsetzen
```bash
mkdir ~/benzin-backend
cd ~/benzin-backend
npm init -y
npm install express axios cors dotenv
```

```bash
touch .env
sudo nano .env
```

Folgenden Text einfügen:
```
TANKERKOENIG_API_KEY=60787b7e-69a4-ffed-1956-951a460c1b61
```

index.js Datei erstellen:
```bash
touch index.js
sudo nano index.js
```

Code:
```js
require('dotenv').config();
const express = require('express');
const axios = require('axios');
const cors = require('cors');

const app = express();
app.use(cors());

app.get('/api/preise', async (req, res) => {
  const { lat, lng, type = 'e5', radius = 5 } = req.query;

  if (!lat || !lng) {
    return res.status(400).json({ error: 'lat und lng sind erforderlich' });
  }

  try {
    const response = await axios.get('https://creativecommons.tankerkoenig.de/json/list.php', {
      params: {
        lat,
        lng,
        rad: radius,
        sort: 'dist',
        type,
        apikey: process.env.TANKERKOENIG_API_KEY
      }
    });

    res.json(response.data);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'API-Anfrage fehlgeschlagen' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server läuft auf Port ${PORT}`));
```

Um den Server zu starten:
```bash
node index.js
```

### pm2 installlieren und konfigurieren
```bash
sudo npm install -g pm2
```

pm2 konfigurieren dass das Backend im Hintergrund läuft:
```bash
pm2 start index.js --name benzin-backend
```

Zum  check:
```bash
pm2 list
```

Das sollte folgendes ausgeben:
```
┌─────┬─────────────────┬──────┬────────┬─────┬───────────┐
│ id  │ name            │ mode │ status │ cpu │ memory    │
├─────┼─────────────────┼──────┼────────┼─────┼───────────┤
│ 0   │ benzin-backend  │ fork │ online │ ... │ ...       │
└─────┴─────────────────┴──────┴────────┴─────┴───────────┘
```

Damit das Backend auch immer automatisch nach einem Neustart des Servers wieder startet:
```bash
pm2 startup
```

Das gibt mir folgenden Befehl aus, welcher ich auch direkt kopiren und 1:1 ausführen muss:
```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

Zum Schluss, um alles zu speichern:
```bash
pm2 save
```

## HTTPS Zertifikat erstellen
Hierzu habe ich bei Duckdns eine gratis Domäne registriert. Der Name der Domände lautet "yassinbenzin.duckdns.org". Sie zeigt auf die öffentliche IP meines Webservers.

Nacher musste ich die Nginx Konfigurationsdatei anpassen.

Vorher:
```bash
server {
    listen 443 ssl;
    server_name 52.204.171.169;
    ssl_certificate /etc/letsencrypt/live/yassinbenzin.duckdns.org/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yassinbenzin.duckdns.org/privkey.pem; # managed by Certbot
```
Nacher:
```bash
server {
    listen 443 ssl;
    server_name yassinbenzin.duckdns.org;
    ssl_certificate /etc/letsencrypt/live/yassinbenzin.duckdns.org/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yassinbenzin.duckdns.org/privkey.pem; # managed by Certbot
```

Anschliessend das Zertifikat erstellen lassen:
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yassinbenzin.duckdns.org
```