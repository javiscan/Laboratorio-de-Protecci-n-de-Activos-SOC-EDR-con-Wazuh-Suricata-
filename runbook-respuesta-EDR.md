# Runbook — Respuesta a una alerta del EDR (Wazuh)

**ID:** RB-EDR-001 · **Versión:** 1.0 · **Alcance:** alertas del dashboard Wazuh sobre endpoints protegidos.

## 1. Detección
- Fuente: dashboard Wazuh (Security events / MITRE ATT&CK).
- Datos de la alerta: agente afectado, nivel (rule level), técnica MITRE, IP origen, timestamp.

## 2. Triage / Clasificación
- [ ] ¿Nivel >= 10 (alta severidad)? → prioridad alta.
- [ ] ¿Falso positivo conocido? → documentar y ajustar regla.
- [ ] ¿Afecta a un activo crítico? → escalar.
- [ ] Correlacionar con FIM (¿hubo cambio de integridad asociado?).

## 3. Contención
- [ ] Aislar el endpoint de la red si hay compromiso confirmado.
- [ ] Bloquear la IP origen (firewall / iptables) si es un ataque externo.
- [ ] Deshabilitar la cuenta comprometida si aplica.

## 4. Erradicación
- [ ] Eliminar el artefacto malicioso / revertir el cambio no autorizado.
- [ ] Aplicar el parche si la causa fue una vulnerabilidad (ver RB-PATCH-001).

## 5. Recuperación
- [ ] Restaurar el servicio/endpoint a estado saludable.
- [ ] Verificar ausencia de nuevas alertas.

## 6. Cierre y lecciones aprendidas
- [ ] Registrar en `CHANGELOG-cambios.md` / ticket.
- [ ] Ajustar reglas de detección o hardening para prevenir recurrencia.
- [ ] Reporte a las áreas correspondientes.
