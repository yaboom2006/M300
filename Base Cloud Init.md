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