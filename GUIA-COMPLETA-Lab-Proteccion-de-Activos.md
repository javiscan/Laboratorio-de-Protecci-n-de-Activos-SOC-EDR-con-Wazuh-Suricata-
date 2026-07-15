# Guía completa — Laboratorio de Protección de Activos (SOC / EDR)

**Ver los activos → configurar la defensa → atacar → registrar toda la telemetría → visualizar → generar informe**

Laboratorio práctico en VirtualBox orientado al puesto de **Técnico/a de Intervención de Protección de Activos** (Sphera-TI). Usa tus máquinas virtuales reales y equivalentes open source de las herramientas comerciales (Cortex/EDR → **Wazuh**, DUO/MFA → **PAM+TOTP**, IDS → **Suricata**). Entorno **100% aislado**.

Objetivo del laboratorio, en tres actos:
1. **Configurar la defensa** para que los ataques se detecten/contengan (firewall, agentes EDR, IDS, MFA, respuesta activa).
2. **Atacar desde Kali** y comprobar que **todo queda registrado** (logs de endpoint, telemetría de red, alertas).
3. **Visualizar** en el dashboard de Wazuh y **generar un informe** con las evidencias.

---

## 1. Inventario de VMs y roles (tus máquinas reales)

| VM (tu VirtualBox) | Rol en el lab | IP fija | Recursos sugeridos |
|---|---|---|---|
| **pfSense-2** | Gateway + firewall (NAT a Internet) | 10.20.0.1 | 1 GB RAM |
| **kali-linux-2025.3** | Atacante / analista | 10.20.0.5 | 2 GB RAM |
| **Debian 12 - Wazuh** | **SIEM / EDR — Manager + Dashboard** (núcleo) | 10.20.0.10 | 4 GB RAM (mínimo) |
| **Windows 10 Markus** | Endpoint corporativo con **agente Wazuh (EDR)** | 10.20.0.15 | 2-4 GB RAM |
| **UbuntuServer** | Activo a proteger: **parcheo + MFA + agente Wazuh** | 10.20.0.20 | 2 GB RAM |
| **Metasploitable2** | Activo **vulnerable** (a escanear) | 10.20.0.30 | 512 MB-1 GB |
| **Debian 13 - Suricata** | Sensor **IDS de red** (telemetría → Wazuh) | 10.20.0.40 | 2 GB RAM |

> Las demás VMs (OWASP-Broken, CyberOps, Windows Server 2019, etc.) quedan de reserva. Si tu equipo va justo de RAM, **arranca por acto**: para el Módulo de vulnerabilidades no necesitas Windows; para EDR sí. No hace falta tenerlas todas encendidas a la vez.

**Prioridad de arranque (por RAM):** pfSense → Debian-Wazuh → el activo del módulo que estés haciendo → Kali. Windows y Suricata solo cuando toque su módulo.

---

## 2. Arquitectura del laboratorio

```
                 INTERNET (simulado por NAT de VirtualBox)
                          |  WAN DHCP
                  ┌───────┴────────┐
                  │   pfSense-2    │   GATEWAY + FIREWALL
                  │  10.20.0.1     │
                  └───────┬────────┘
                red interna `lab` — 10.20.0.0/24 (aislada)
   ┌──────────┬───────────┼───────────┬──────────────┬──────────────┐
   │          │           │           │              │              │
┌──┴───┐  ┌───┴────┐  ┌───┴────┐  ┌───┴─────┐  ┌─────┴──────┐  ┌────┴──────┐
│ KALI │  │ WAZUH  │  │WINDOWS │  │ UBUNTU  │  │METASPLOIT. │  │ SURICATA  │
│ .5   │  │ .10    │  │ 10 .15 │  │SERVER.20│  │  .30       │  │  .40      │
│atacan│  │SIEM/EDR│  │agente  │  │agente + │  │activo vuln.│  │IDS de red │
│analis│  │Dashboard│ │Wazuh   │  │MFA+parch│  │(a escanear)│  │→ Wazuh    │
└──┬───┘  └───▲────┘  └───┬────┘  └────┬────┘  └─────┬──────┘  └────┬──────┘
   │          │           │            │             │              │
   │ ataca    │  telemetría (agentes + Suricata + logs)             │
   └──────────┴───────────┴────────────┴─────────────┴──────────────┘
                          │
              Wazuh Dashboard (https://10.20.0.10)
        alertas · EDR · FIM · IDS · vulnerabilidades · MITRE ATT&CK
```

**La idea clave:** Wazuh (10.20.0.10) es el **punto central** donde converge TODA la telemetría — logs de los endpoints (agentes), alertas de red (Suricata) y eventos del firewall. Desde ahí se ve y se genera el informe.

---

## 3. Configuración de red de cada máquina  ← (requisito 1: ver las máquinas y su config)

En **todas** las VMs: `Configuración → Red → Adaptador 1 → Red interna`, nombre **`lab`**.
- pfSense necesita **2 adaptadores**: Adaptador 1 = **NAT** (WAN, salida a Internet); Adaptador 2 = **Red interna `lab`** (LAN).

Luego, IP fija dentro de cada VM (el método por SO está en `IPs-fijas-lab-VirtualBox.md`):

| VM | Adaptador VirtualBox | IP / máscara | Gateway | DNS |
|---|---|---|---|---|
| pfSense-2 | 1: NAT · 2: interna `lab` | LAN 10.20.0.1/24 | — | — |
| Kali | interna `lab` | 10.20.0.5/24 | 10.20.0.1 | 10.20.0.1 |
| Debian 12 - Wazuh | interna `lab` | 10.20.0.10/24 | 10.20.0.1 | 10.20.0.1 |
| Windows 10 Markus | interna `lab` | 10.20.0.15/24 | 10.20.0.1 | 10.20.0.1 |
| UbuntuServer | interna `lab` | 10.20.0.20/24 | 10.20.0.1 | 10.20.0.1 |
| Metasploitable2 | interna `lab` | 10.20.0.30/24 | 10.20.0.1 | 10.20.0.1 |
| Debian 13 - Suricata | interna `lab` | 10.20.0.40/24 | 10.20.0.1 | 10.20.0.1 |

**Configurar la IP en Windows 10** (Panel de control → Red → Adaptador → Propiedades → IPv4): IP `10.20.0.15`, máscara `255.255.255.0`, puerta de enlace `10.20.0.1`, DNS `10.20.0.1`. O por PowerShell (admin):
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.20.0.15 -PrefixLength 24 -DefaultGateway 10.20.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.20.0.1
```

**Verificación de red (hazlo antes de seguir):** desde Kali, `ping 10.20.0.10`, `ping 10.20.0.20`, `ping 10.20.0.30`. Todos deben responder. En pfSense, activa la salida a Internet (Interfaces → WAN → desmarcar "Block private networks").

> **Snapshot ahora:** cuando la red funcione, toma una instantánea de cada VM llamada `red-ok`. Es tu punto de retorno.

---

## ACTO 1 — Configurar la defensa (para que el ataque quede registrado/contenido)

### 4. Wazuh Manager + Dashboard (el cerebro del lab)
Tu VM **Debian 12 - Wazuh** ya lo trae. Enciéndela y verifica que el servicio está activo:
```bash
sudo systemctl status wazuh-manager wazuh-dashboard wazuh-indexer
```
Accede al dashboard: `https://10.20.0.10` (usuario `admin`, la contraseña que configuraste al instalar). Si necesitas reinstalar:
```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

### 5. Desplegar agentes EDR en los activos
**En Windows 10 (10.20.0.15)** — PowerShell como Administrador:
```powershell
msiexec.exe /i wazuh-agent-4.9.0-1.msi /q WAZUH_MANAGER="10.20.0.10" WAZUH_AGENT_NAME="win10-markus"
NET START WazuhSvc
```
**En UbuntuServer (10.20.0.20):**
```bash
sudo WAZUH_MANAGER="10.20.0.10" apt-get install -y wazuh-agent
sudo systemctl enable --now wazuh-agent
```
En el dashboard → *Agents*, ambos deben figurar **Active**. Eso es tu EDR desplegado en los puestos.

### 6. Suricata como sensor de red → enviando a Wazuh
Tu VM **Debian 13 - Suricata** ya lo trae. Asegúrate de que escribe `eve.json` y que el **agente Wazuh** de esa VM lo lee:
```bash
# En Debian-Suricata: instalar el agente Wazuh apuntando al manager
sudo WAZUH_MANAGER="10.20.0.10" apt-get install -y wazuh-agent
# Decirle al agente que recoja el eve.json de Suricata:
sudo tee -a /var/ossec/etc/ossec.conf >/dev/null <<'EOF'
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
EOF
sudo systemctl restart wazuh-agent suricata
```
Con esto, **las alertas de red de Suricata aparecen dentro de Wazuh** (telemetría de red centralizada).

### 7. Activar FIM y respuesta activa (para "que no llegue el ataque")
En el **manager** (Debian-Wazuh), `/var/ossec/etc/ossec.conf`:
- **FIM** (integridad de ficheros):
```xml
<syscheck>
  <directories check_all="yes" realtime="yes">/etc,/usr/bin</directories>
</syscheck>
```
- **Respuesta activa** (Wazuh banea automáticamente la IP atacante ante fuerza bruta):
```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5710,5712,5763</rules_id>   <!-- reglas de fuerza bruta SSH -->
  <timeout>600</timeout>
</active-response>
```
```bash
sudo systemctl restart wazuh-manager
```

### 8. MFA en el activo (UbuntuServer) — equivalente a DUO
```bash
sudo apt-get install -y libpam-google-authenticator
google-authenticator          # escanea el QR con Google Authenticator; guarda códigos de recuperación
echo "auth required pam_google_authenticator.so" | sudo tee -a /etc/pam.d/sshd
sudo sed -i 's/^#\?KbdInteractiveAuthentication.*/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart ssh
```
Prueba desde Kali: `ssh usuario@10.20.0.20` → pide **contraseña + código de 6 dígitos**. MFA operativo.

> **Snapshot `defensa-ok`** cuando todo esto esté configurado.

---

## ACTO 2 — Atacar y comprobar que TODO queda registrado

Desde **Kali (10.20.0.5)**. Cada ataque genera telemetría en un plano distinto; abajo se indica **dónde** se registra.

### 9.1 Escaneo de vulnerabilidades (activo Metasploitable)
```bash
nmap -sV --script vuln 10.20.0.30 -oN scan_metasploitable.txt
```
→ **Suricata** detecta el escaneo (alertas en Wazuh) · el informe local queda para el Módulo de vulnerabilidades.

### 9.2 Fuerza bruta SSH contra el activo (UbuntuServer)
```bash
sudo apt-get install -y hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.20.0.20 -t 4
```
→ **Wazuh** (agente del Ubuntu) registra los intentos fallidos y, por la **respuesta activa**, **banea la IP de Kali** (el ataque "no llega"). Verás la alerta y el `firewall-drop`.

### 9.3 Actividad sospechosa en el endpoint Windows
Desde Kali, escaneo/objetivo Windows:
```bash
nmap -A 10.20.0.15
```
→ **Wazuh** (agente Windows) y **Suricata** registran la actividad. Mapeo a **MITRE ATT&CK** en el dashboard.

### 9.4 Cambio no autorizado (prueba de FIM)
En el activo Ubuntu: `sudo touch /etc/cambio_no_autorizado.txt`
→ **Wazuh FIM** genera alerta de integridad al instante.

---

## ACTO 3 — Visualizar la telemetría y generar el informe

### 10. Visualizar en el dashboard de Wazuh (https://10.20.0.10)
- **Security events / Threat Hunting** → todas las alertas (fuerza bruta, escaneos, MITRE).
- **Integrity monitoring (FIM)** → los cambios en ficheros vigilados.
- **Vulnerabilities** → CVEs detectados en los agentes.
- **Agents** → estado de cada endpoint protegido (Active).
- Filtra por `agent.name`, `rule.level >= 10`, `data.srcip: 10.20.0.5` para aislar el ataque.

Qué demuestras aquí (los 3 actos): **defensa configurada** (agentes + respuesta activa) → **ataque contenido y registrado** (banned + alertas) → **visibilidad centralizada** (todo en un panel).

### 11. Generar el informe
- **Rápido:** en el dashboard, *Reporting* → genera un PDF del módulo (Security events, FIM o Vulnerabilities) con los filtros del ataque.
- **Documental:** registra el ejercicio en `CHANGELOG-cambios.md` y responde según `runbook-respuesta-EDR.md`.
- Captura de evidencias para el informe: alerta de fuerza bruta + `firewall-drop`, alerta FIM, informe de vulnerabilidades, login SSH con MFA.

---

## 12. Gestión de vulnerabilidades: parcheo con runbook + rollback (activo Ubuntu)
Sigue `runbook-parcheo.md`. Resumen del ciclo:
```bash
# 1) Snapshot "pre-parche" en VirtualBox (rollback garantizado)
# 2) Inventario
sudo apt-get update && apt list --upgradable 2>/dev/null | grep -i security
# 3) Aplicar solo parches de seguridad
sudo apt-get install -y unattended-upgrades && sudo unattended-upgrade -d
# 4) Verificar / si falla -> rollback:
#    sudo apt-get install <paquete>=<version-anterior>  |  o restaurar snapshot
```
Registra el cambio en `CHANGELOG-cambios.md`.

---

## 13. Subir a GitHub (para que cualquiera lo estudie)
Desde la carpeta del lab (clic derecho → "Git Bash Here"):
```bash
git init
git add .
git commit -m "Lab Proteccion de Activos: EDR Wazuh + IDS Suricata + vuln mgmt + parcheo/rollback + MFA + runbooks"
git branch -M main
git remote add origin https://github.com/javiscan/NOMBRE-DEL-REPO.git
git push -u origin main
```
Crea antes el repo vacío en https://github.com/new y haz el push con tu cuenta.

---

## Solución de problemas
| Síntoma | Causa | Solución |
|---|---|---|
| Agente no aparece "Active" | IP del manager mal / puertos 1514-1515 | Revisar `ossec.conf` del agente y firewall pfSense |
| Suricata no manda a Wazuh | Falta el `<localfile>` del eve.json | Añadirlo en `ossec.conf` del agente y reiniciar |
| Wazuh dashboard no carga | Servicios caídos / poca RAM | `systemctl status wazuh-*`; darle 4 GB a la VM |
| Respuesta activa no banea | Reglas o timeout mal | Verificar `rules_id` y `<active-response>` |
| Windows sin IP | Config manual mal | Usar el bloque PowerShell del punto 3 |
| SSH no pide TOTP | Falta reiniciar ssh / PAM | `sudo systemctl restart ssh` |

---

*Guía orientada al puesto de Técnico de Intervención de Protección de Activos. Basada en VMs reales. Autor: Javier Scandura — Ciberseguridad Blue Team.*
