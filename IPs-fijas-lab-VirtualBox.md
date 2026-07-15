# IPs fijas — Lab Protección de Activos (red interna `lab`, 10.20.0.0/24)

Todas las VMs con **Adaptador 1 → Red interna, nombre `lab`** (pfSense además lleva Adaptador 1 = NAT para la WAN). IPs fijas por sistema operativo.

## Plan de direccionamiento

| VM | IP fija | Gateway | DNS |
|---|---|---|---|
| pfSense-2 (LAN) | 10.20.0.1/24 | — | — |
| Kali | 10.20.0.5/24 | 10.20.0.1 | 10.20.0.1 |
| Debian 12 - Wazuh | 10.20.0.10/24 | 10.20.0.1 | 10.20.0.1 |
| Windows 10 Markus | 10.20.0.15/24 | 10.20.0.1 | 10.20.0.1 |
| UbuntuServer | 10.20.0.20/24 | 10.20.0.1 | 10.20.0.1 |
| Metasploitable2 | 10.20.0.30/24 | 10.20.0.1 | 10.20.0.1 |
| Debian 13 - Suricata | 10.20.0.40/24 | 10.20.0.1 | 10.20.0.1 |

## pfSense-2 (por consola)
Opción **2) Set interface IP** → LAN `10.20.0.1/24`, DHCP server: **No**. WAN (NAT) en DHCP. En la GUI: Interfaces → WAN → desmarcar "Block private networks".

## Kali (NetworkManager)
```bash
ip -brief link      # ver interfaz (eth0)
sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 10.20.0.5/24 ipv4.gateway 10.20.0.1 ipv4.dns 10.20.0.1
sudo nmcli con down "Wired connection 1" && sudo nmcli con up "Wired connection 1"
```

## Debian (Wazuh y Suricata) — /etc/network/interfaces o netplan
Si usa ifupdown (`/etc/network/interfaces`):
```
auto eth0
iface eth0 inet static
    address 10.20.0.10        # (Suricata: 10.20.0.40)
    netmask 255.255.255.0
    gateway 10.20.0.1
    dns-nameservers 10.20.0.1
```
```bash
sudo systemctl restart networking
```

## UbuntuServer (netplan)
```bash
ls /etc/netplan/    # editar el .yaml
```
```yaml
network:
  version: 2
  ethernets:
    enp0s3:                 # pon el nombre real (ip -brief link)
      dhcp4: no
      addresses: [10.20.0.20/24]
      routes:
        - to: default
          via: 10.20.0.1
      nameservers:
        addresses: [10.20.0.1, 8.8.8.8]
```
```bash
sudo chmod 600 /etc/netplan/*.yaml && sudo netplan apply
```

## Metasploitable2 (/etc/network/interfaces)
```
auto eth0
iface eth0 inet static
    address 10.20.0.30
    netmask 255.255.255.0
    gateway 10.20.0.1
```
```bash
sudo /etc/init.d/networking restart
```

## Windows 10 (PowerShell como Administrador)
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.20.0.15 -PrefixLength 24 -DefaultGateway 10.20.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.20.0.1
```

## Verificación
Arranca pfSense primero. Luego, desde Kali: `ping 10.20.0.10`, `ping 10.20.0.20`, `ping 10.20.0.30`. Toma snapshot `red-ok` de cada VM cuando respondan.
