# 🧪 Práctica guiada — Laboratorio de Protección de Activos (SOC / EDR)

Retos organizados por acto. Cada reto indica **dificultad**, **objetivo** y la **evidencia** que debes entregar (captura, archivo o log). Sube tus evidencias a una carpeta `evidencias/` y regístralas en `CHANGELOG-cambios.md`.

**Escala de dificultad:** ⭐ básico · ⭐⭐ intermedio · ⭐⭐⭐ avanzado
**Puntuación total:** 100 pts. Aprobado ≥ 60 pts. La rúbrica del instructor está en `RUBRICA-evaluacion.md`.

> **Antes de empezar:** monta la red según `IPs-fijas-lab-VirtualBox.md`, verifica conectividad desde Kali (`ping 10.20.0.10/.20/.30`) y toma snapshot `red-ok` de cada VM.

---

## 🏁 Ranking de progreso

| Nivel | Puntos | Perfil |
|---|---|---|
| 🥉 Bronce | 60-74 | Sabe operar el SIEM y detectar lo básico. |
| 🥈 Plata | 75-89 | Detecta, correlaciona y contiene con evidencias. |
| 🥇 Oro | 90-100 | Domina detección + respuesta + reporte con calidad profesional. |

---

## ACTO 1 — Configurar la defensa (30 pts)

### Reto 1.1 — Red aislada operativa · ⭐ · 5 pts
**Objetivo:** dejar las 7 VMs en la red interna `lab` con IP fija y salida a Internet vía pfSense.
**Evidencia:** captura de `ping` correcto desde Kali a `.10`, `.20` y `.30` + snapshot `red-ok`.

### Reto 1.2 — Wazuh Manager + Dashboard vivo · ⭐ · 5 pts
**Objetivo:** verificar que `wazuh-manager`, `wazuh-indexer` y `wazuh-dashboard` están activos y acceder a `https://10.20.0.10`.
**Evidencia:** salida de `systemctl status wazuh-*` + captura del login del dashboard.

### Reto 1.3 — Agentes EDR desplegados · ⭐⭐ · 8 pts
**Objetivo:** instalar el agente Wazuh en Windows 10 (.15) y UbuntuServer (.20) apuntando al manager.
**Evidencia:** captura de *Agents* con ambos en estado **Active**.

### Reto 1.4 — Suricata → Wazuh · ⭐⭐ · 7 pts
**Objetivo:** que las alertas de red de Suricata (.40) aparezcan dentro de Wazuh (leyendo `eve.json`).
**Evidencia:** captura de una alerta de Suricata visible en el dashboard de Wazuh.

### Reto 1.5 — FIM + respuesta activa + MFA · ⭐⭐⭐ · 5 pts
**Objetivo:** activar syscheck (FIM) sobre `/etc`, la respuesta activa `firewall-drop` para fuerza bruta SSH, y MFA (PAM+TOTP) en el activo Ubuntu.
**Evidencia:** fragmento de `ossec.conf` con `<syscheck>` y `<active-response>` + login SSH que pide contraseña **y** código de 6 dígitos. Snapshot `defensa-ok`.

---

## ACTO 2 — Atacar y comprobar que TODO queda registrado (40 pts)

### Reto 2.1 — Escaneo de vulnerabilidades · ⭐⭐ · 10 pts
**Objetivo:** desde Kali, `nmap -sV --script vuln 10.20.0.30` contra Metasploitable.
**Evidencia:** `scan_metasploitable.txt` + captura de la alerta de escaneo detectada por Suricata en Wazuh. Identifica **2 CVE** y su CVSS.

### Reto 2.2 — Fuerza bruta SSH contenida · ⭐⭐⭐ · 12 pts
**Objetivo:** `hydra` contra SSH del activo Ubuntu (.20) y comprobar que la **respuesta activa banea** la IP de Kali.
**Evidencia:** alerta de fuerza bruta (reglas 5710/5712/5763) + evento `firewall-drop` con la IP `10.20.0.5` baneada. Explica por qué "el ataque no llega".

### Reto 2.3 — Reconocimiento contra el endpoint Windows · ⭐⭐ · 8 pts
**Objetivo:** `nmap -A 10.20.0.15` y localizar el evento en Wazuh + su técnica **MITRE ATT&CK**.
**Evidencia:** captura del evento con la técnica MITRE mapeada (ej. T1046 Network Service Discovery).

### Reto 2.4 — Cambio no autorizado (FIM) · ⭐ · 10 pts
**Objetivo:** `sudo touch /etc/cambio_no_autorizado.txt` en el Ubuntu y capturar la alerta de integridad.
**Evidencia:** captura de la alerta FIM (added) con ruta y timestamp.

---

## ACTO 3 — Visualizar y reportar (30 pts)

### Reto 3.1 — Threat Hunting con filtros · ⭐⭐ · 10 pts
**Objetivo:** en el dashboard, aislar el ataque filtrando por `data.srcip: 10.20.0.5` y `rule.level >= 10`.
**Evidencia:** captura del panel filtrado mostrando solo la actividad del atacante.

### Reto 3.2 — Gestión de vulnerabilidades: parcheo + rollback · ⭐⭐⭐ · 12 pts
**Objetivo:** aplicar el ciclo de `runbook-parcheo.md` en el activo Ubuntu (snapshot → inventario → parche de seguridad → verificación → rollback si falla).
**Evidencia:** salida de `apt list --upgradable | grep -i security` antes/después + registro en `CHANGELOG-cambios.md`.

### Reto 3.3 — Informe de detección y defensa · ⭐⭐ · 8 pts
**Objetivo:** redactar tu informe con las evidencias de los actos 2 y 3, siguiendo `runbook-respuesta-EDR.md`.
**Evidencia:** informe (PDF o DOCX) comparado con el ejemplo `INFORME-Deteccion-y-Defensa.docx`.

---

## ✅ Checklist de entrega

- [ ] Carpeta `evidencias/` con capturas nombradas por reto (ej. `2.2-firewall-drop.png`).
- [ ] `CHANGELOG-cambios.md` actualizado con cada ejercicio.
- [ ] Informe final de detección y defensa.
- [ ] Autoevaluación con la escala de ranking (Bronce/Plata/Oro).
