# Projektanforderungen – Tankpreise-Webseite

## Projekt
- Webanwendung zur Abfrage von aktuellen Benzinpreisen
- Frontend zeigt Standort und günstigste Tankstellen
- Backend ruft externe API auf und liefert gefilterte Daten
- Möglichkeit zur späteren Benutzeranmeldung (Favoriten etc.)

---

## Infrastruktur

### AWS-Grundlagen
- [ ] AWS-Zugang erstellen
### Netzwerk
- [ ] VPC + Subnets einrichten (optional, für sauberes Netzwerk)
- [ ] Security Groups definieren:
  - [ ] EC2-Webserver: Ports 80, 443, 22
  - [ ] EC2-Datenbank: Nur intern erreichbar für Webserver

---

## Server

### Webserver (EC2)
- [ ] EC2-Instanz (Ubuntu 22.04)
- [ ] Nginx installieren & konfigurieren
- [ ] Node.js + npm installieren
- [ ] Firewall konfigurieren (ufw oder AWS-Security Group)
- [ ] Domain (optional) einrichten + auf Server-IP zeigen
- [ ] HTTPS-Zertifikat mit Let's Encrypt einrichten

### Datenbankserver (separat oder RDS)
- [ ] EC2 oder RDS-Instanz mit PostgreSQL / MySQL / MongoDB
- [ ] Benutzer & Rollen mit sicheren Passwörtern
- [ ] Nur interne Verbindungen vom Webserver zulassen

---

## Software & Backend

### Backend (Node.js)
- [ ] Projektstruktur aufsetzen
- [ ] Routen/API-Endpunkte definieren
