# Runbook — Parcheo de seguridad de un activo

**ID:** RB-PATCH-001 · **Versión:** 1.0 · **Propietario:** Equipo de Protección de Activos
**Alcance:** aplicación de parches de seguridad en servidores Linux (activos productivos).

## 1. Requisitos previos (pre-checks)
- [ ] Ventana de cambio aprobada y comunicada.
- [ ] **Snapshot / backup** del activo tomado y verificado (rollback garantizado).
- [ ] Inventario de parches pendientes revisado y priorizado por CVSS.
- [ ] Servicio en estado saludable antes del cambio (baseline de disponibilidad).
- [ ] Registro del cambio abierto en `CHANGELOG-cambios.md`.

## 2. Ejecución
```bash
# 2.1 Actualizar índice de paquetes
sudo apt-get update

# 2.2 Listar solo parches de seguridad pendientes
apt list --upgradable 2>/dev/null | grep -i security

# 2.3 Simulación (dry-run) antes de aplicar
sudo unattended-upgrade --dry-run -d

# 2.4 Aplicar parches de seguridad
sudo unattended-upgrade -d
```

## 3. Verificación post-cambio
- [ ] `sudo apt-get -s upgrade | grep -i secur` → sin parches de seguridad pendientes.
- [ ] Servicios críticos activos: `systemctl status <servicio>`.
- [ ] Prueba funcional del servicio (respuesta esperada).
- [ ] Sin alertas nuevas en el EDR (Wazuh) tras el cambio.

## 4. Rollback (si la verificación falla)
```bash
# Opción A - revertir un paquete concreto
apt-cache policy <paquete>
sudo apt-get install <paquete>=<version-anterior>
sudo apt-mark hold <paquete>

# Opción B - restaurar la snapshot "pre-parche" en VirtualBox
```
- [ ] Confirmar que el servicio vuelve al estado saludable tras el rollback.

## 5. Cierre
- [ ] Actualizar `CHANGELOG-cambios.md` con resultado (OK / rollback) y evidencias.
- [ ] Comunicar cierre de la ventana de cambio.
- [ ] Si hubo rollback: abrir incidencia para replanificar el parche.
