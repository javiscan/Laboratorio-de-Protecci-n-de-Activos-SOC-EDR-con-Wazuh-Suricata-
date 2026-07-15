# Laboratorio de Protección de Activos (SOC / EDR) — Wazuh · Suricata · MFA · Vuln Mgmt

Laboratorio práctico de **operaciones de protección de activos** en VirtualBox, orientado al puesto de Técnico/a de Intervención de Protección de Activos. Demuestra el ciclo completo: **configurar la defensa → atacar → registrar toda la telemetría → visualizar → generar informe**, con equivalentes open source de las herramientas comerciales (Cortex/EDR → Wazuh, DUO/MFA → PAM+TOTP, IDS → Suricata).

> Autor: **Javier Scandura** — Analista de Ciberseguridad (Blue Team). Entorno 100% aislado (red interna de VirtualBox).

## Arquitectura (red interna `lab` 10.20.0.0/24)

```
INTERNET (NAT) ── pfSense-2 (10.20.0.1, firewall)
                        │  red interna `lab`
  Kali .5 (atacante) · Wazuh .10 (SIEM/EDR) · Windows10 .15 (agente)
  UbuntuServer .20 (agente+MFA+parcheo) · Metasploitable2 .30 (vuln)
  Debian-Suricata .40 (IDS de red → Wazuh)
```
Wazuh (10.20.0.10) es el punto central donde converge toda la telemetría (endpoints + red + firewall) y desde donde se visualiza y se genera el informe.

## Qué demuestra (mapa al puesto)

| Área del puesto | Implementación |
|---|---|
| EDR (Cortex/Trellix) | Wazuh Manager + agentes (Windows + Linux) |
| IDS / telemetría de red | Suricata → Wazuh |
| Gestión de vulnerabilidades | Nmap/OpenVAS + priorización CVSS |
| Parcheo y rollback | apt/unattended-upgrades + runbook + snapshot |
| MFA (DUO) | PAM + TOTP en SSH |
| FIM | Wazuh syscheck |
| Respuesta y contención | Wazuh active response (auto-ban) |
| Gestión de cambios (BAU) | Runbooks + change log |

## Contenido

| Archivo | Descripción |
|---|---|
| `GUIA-COMPLETA-Lab-Proteccion-de-Activos.md` | Guía paso a paso (empieza aquí). |
| `IPs-fijas-lab-VirtualBox.md` | Config de red e IP fija por máquina. |
| `runbook-parcheo.md` | Procedimiento validado de parcheo con rollback. |
| `runbook-respuesta-EDR.md` | Respuesta a alertas del EDR (ciclo NIST). |
| `CHANGELOG-cambios.md` | Registro documental de cambios. |

## Tecnologías
`Wazuh` · `Suricata` · `PAM + TOTP` · `Nmap/OpenVAS` · `apt/unattended-upgrades` · `pfSense` · `Kali` · `Windows` · `Metasploitable` · `VirtualBox`

## Aviso legal
Uso educativo. Metasploitable es intencionadamente vulnerable: no exponer a redes reales. Ataques únicamente contra las VMs propias del laboratorio aislado.
