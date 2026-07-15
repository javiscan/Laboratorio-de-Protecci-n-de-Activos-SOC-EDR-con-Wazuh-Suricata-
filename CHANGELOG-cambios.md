# 📝 CHANGELOG de cambios — Lab Protección de Activos

Registro documental de todos los cambios aplicados sobre los activos del laboratorio (gestión de cambios / BAU). Una fila por cambio. Referencia los runbooks y las evidencias.

| Fecha | Activo | Cambio | Runbook | Snapshot previo | Resultado | Evidencia |
|---|---|---|---|---|---|---|
| AAAA-MM-DD | UbuntuServer (.20) | Ejemplo: parche de seguridad `openssl` | RB-PATCH-001 | `pre-parche` | OK | evidencias/3.2-apt-security.png |
| | | | | | | |

## Convenciones
- **Resultado:** `OK` · `Rollback` · `Pendiente`.
- Toda fila con `Rollback` debe abrir una incidencia para replanificar (ver `runbook-parcheo.md`).
- Nombra las evidencias por reto: `evidencias/<reto>-<descripcion>.png`.
- Referencia la técnica MITRE cuando el cambio sea consecuencia de una alerta del EDR.

## Historial
- **v1.0** — Estructura inicial del laboratorio (7 VMs, red `lab` 10.20.0.0/24, Wazuh + Suricata + MFA + runbooks).
