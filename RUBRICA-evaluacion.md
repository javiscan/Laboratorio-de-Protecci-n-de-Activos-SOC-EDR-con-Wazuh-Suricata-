# 📊 Rúbrica de evaluación — Lab Protección de Activos (uso del instructor)

Evaluación por **competencias** alineada al puesto de Técnico/a de Intervención de Protección de Activos. Total **100 pts**. Aprobado ≥ 60. Correlaciona con los retos de `PRACTICA.md`.

## 1. Ponderación por competencia

| Competencia | Retos asociados | Peso |
|---|---|---|
| Configuración de la defensa (hardening, agentes, IDS, MFA) | 1.1–1.5 | 30 % |
| Detección y telemetría (SIEM/EDR, IDS, FIM, MITRE) | 2.1–2.4, 3.1 | 40 % |
| Respuesta, contención y gestión de vulnerabilidades | 2.2, 3.2 | 20 % |
| Reporte y documentación profesional | 3.3, CHANGELOG | 10 % |

## 2. Niveles de desempeño por reto

Para **cada** reto, puntuar según:

| Nivel | Criterio | % del valor del reto |
|---|---|---|
| Excelente | Objetivo cumplido, evidencia clara y correctamente etiquetada, explicación técnica correcta. | 100 % |
| Competente | Objetivo cumplido, evidencia presente pero con detalles menores incompletos. | 75 % |
| En desarrollo | Objetivo parcialmente cumplido o evidencia ambigua/insuficiente. | 40 % |
| No logrado | Sin evidencia válida o resultado incorrecto. | 0 % |

## 3. Criterios de calidad transversales (aplican a toda la entrega)

- **Trazabilidad:** cada evidencia está nombrada por reto y referenciada en `CHANGELOG-cambios.md`.
- **Reproducibilidad:** los pasos permiten repetir el resultado (comandos, IPs, snapshots).
- **Correlación:** el alumno relaciona alerta ↔ ataque ↔ técnica MITRE ↔ respuesta.
- **Higiene de laboratorio:** entorno aislado, snapshots antes de cambios, rollback probado.
- **Redacción del informe:** claridad, evidencias, impacto y recomendaciones accionables.

## 4. Escala de ranking final

| Puntos | Ranking | Descriptor |
|---|---|---|
| 90-100 | 🥇 Oro | Detección + respuesta + reporte de nivel profesional. |
| 75-89 | 🥈 Plata | Detecta, correlaciona y contiene con evidencias sólidas. |
| 60-74 | 🥉 Bronce | Opera el SIEM y detecta lo básico. |
| < 60 | No aprobado | Repetir actos con evidencias faltantes. |

## 5. Penalizaciones

- −5 pts: atacar/escanear fuera de las VMs del laboratorio.
- −5 pts: cambios sin snapshot previo (riesgo de no-rollback).
- −3 pts: evidencias sin etiquetar o no referenciadas.

## 6. Plantilla de acta (rellenar por alumno)

```
Alumno:  __________________________   Fecha: __________
Acto 1 (30):  ____   Acto 2 (40): ____   Acto 3 (30): ____
Penalizaciones: ____   TOTAL: ____ /100   Ranking: __________
Observaciones: ______________________________________________
```
