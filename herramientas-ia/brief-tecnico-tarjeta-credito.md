# Brief Técnico: Gestión de Datos de Tarjeta de Crédito

## 1. Introducción

Este documento técnico describe los datos clave relacionados con la gestión de tarjetas de crédito, incluyendo el pago mínimo, la fecha límite de pago, el rango de facturación y los mecanismos para actualizar saldos en cuentas de tarjeta de crédito o cuentas bancarias asociadas.

---

## 2. Datos Clave de la Tarjeta de Crédito

### 2.1 Pago Mínimo (`pago_minimo`)

- **Descripción:** Monto mínimo que el titular debe pagar antes de la fecha límite para evitar cargos por mora.
- **Tipo de dato:** `DECIMAL(12, 2)`
- **Reglas de negocio:**
  - Generalmente equivale al 1.5% al 5% del saldo total adeudado, o un monto fijo mínimo establecido por la entidad emisora (p. ej., $50.00 MXN).
  - Si el saldo total es menor al mínimo fijo, el pago mínimo es igual al saldo total.
  - Se recalcula al cierre de cada ciclo de facturación.
- **Ejemplo:** `pago_minimo = 350.00`

### 2.2 Fecha Límite de Pago (`fecha_limite_pago`)

- **Descripción:** Fecha máxima en que el titular debe realizar al menos el pago mínimo para evitar intereses moratorios y cargos adicionales.
- **Tipo de dato:** `DATE` (formato `YYYY-MM-DD`)
- **Reglas de negocio:**
  - Típicamente se ubica entre 20 y 25 días después del cierre del período de facturación.
  - Si la fecha límite cae en día inhábil (fin de semana o día festivo), se recorre al siguiente día hábil.
  - Se debe notificar al titular con al menos 14 días de anticipación (regulación estándar).
- **Ejemplo:** `fecha_limite_pago = 2026-05-15`

### 2.3 Rango de Facturación (`rango_facturacion`)

- **Descripción:** Período de tiempo que abarca el ciclo de facturación activo, definido por una fecha de inicio y una fecha de corte.
- **Campos relacionados:**
  - `fecha_inicio_facturacion` — `DATE`: Primer día del ciclo de facturación.
  - `fecha_corte_facturacion` — `DATE`: Último día del ciclo; a partir de este día se genera el estado de cuenta.
- **Reglas de negocio:**
  - El ciclo estándar dura entre 28 y 31 días calendario.
  - Todas las transacciones realizadas dentro del rango de facturación se incluyen en el estado de cuenta del período.
  - Las transacciones posteriores al corte se acumulan en el siguiente ciclo.
- **Ejemplo:**
  - `fecha_inicio_facturacion = 2026-04-16`
  - `fecha_corte_facturacion = 2026-05-15`

---

## 3. Estructura de Datos — Modelo Relacional

```sql
CREATE TABLE tarjeta_credito (
  id_tarjeta          BIGINT PRIMARY KEY AUTO_INCREMENT,
  numero_tarjeta      VARCHAR(19) NOT NULL,          -- Enmascarado: **** **** **** 1234
  id_cuenta_bancaria  BIGINT NOT NULL,               -- FK a tabla cuentas_bancarias
  saldo_actual        DECIMAL(12, 2) NOT NULL DEFAULT 0.00,
  limite_credito      DECIMAL(12, 2) NOT NULL,
  saldo_disponible    DECIMAL(12, 2) NOT NULL,
  pago_minimo         DECIMAL(12, 2) NOT NULL,
  fecha_limite_pago   DATE NOT NULL,
  fecha_inicio_facturacion DATE NOT NULL,
  fecha_corte_facturacion  DATE NOT NULL,
  tasa_interes_anual  DECIMAL(5, 2) NOT NULL,        -- Porcentaje anual (p. ej., 36.00)
  estado              ENUM('ACTIVA', 'BLOQUEADA', 'CANCELADA') DEFAULT 'ACTIVA',
  fecha_creacion      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 4. Flujo de Actualización de Saldos

### 4.1 Proceso de Pago

Cuando se registra un pago hacia la tarjeta de crédito, se deben actualizar los siguientes campos de forma atómica (transacción):

1. **Reducción del saldo actual:**
   ```sql
   UPDATE tarjeta_credito
   SET saldo_actual = saldo_actual - :monto_pago,
       saldo_disponible = saldo_disponible + :monto_pago,
       fecha_actualizacion = NOW()
   WHERE id_tarjeta = :id_tarjeta;
   ```

2. **Débito en la cuenta bancaria de origen:**
   ```sql
   UPDATE cuentas_bancarias
   SET saldo = saldo - :monto_pago,
       fecha_actualizacion = NOW()
   WHERE id_cuenta = :id_cuenta_bancaria;
   ```

3. **Registro de transacción:**
   ```sql
   INSERT INTO transacciones (
     id_tarjeta, id_cuenta_bancaria, tipo, monto, descripcion, fecha_transaccion
   ) VALUES (
     :id_tarjeta, :id_cuenta_bancaria, 'PAGO', :monto_pago, 'Pago de tarjeta de crédito', NOW()
   );
   ```

### 4.2 Validaciones Previas al Pago

| Validación | Descripción |
|---|---|
| Saldo suficiente en cuenta | El saldo en cuenta bancaria debe ser >= monto del pago |
| Monto mínimo | El pago debe ser >= `pago_minimo` para evitar cargos por mora |
| Fecha límite | Si `fecha_actual > fecha_limite_pago`, aplicar cargo por pago tardío |
| Estado de tarjeta | La tarjeta debe estar en estado `ACTIVA` |
| Monto máximo | El pago no debe exceder el saldo total adeudado (`saldo_actual`) |

### 4.3 Recálculo al Cierre de Ciclo de Facturación

Al alcanzar la `fecha_corte_facturacion`, el sistema debe:

1. Calcular el nuevo `pago_minimo` con base en el saldo acumulado.
2. Establecer la nueva `fecha_limite_pago` (fecha de corte + 20 días hábiles).
3. Generar el estado de cuenta para el período.
4. Iniciar el nuevo ciclo: actualizar `fecha_inicio_facturacion` y `fecha_corte_facturacion`.
5. Aplicar intereses si el saldo no fue liquidado en su totalidad.

```sql
UPDATE tarjeta_credito
SET pago_minimo = GREATEST(saldo_actual * 0.015, 50.00),
    fecha_limite_pago = DATE_ADD(fecha_corte_facturacion, INTERVAL 20 DAY),
    fecha_inicio_facturacion = DATE_ADD(fecha_corte_facturacion, INTERVAL 1 DAY),
    fecha_corte_facturacion = DATE_ADD(fecha_corte_facturacion, INTERVAL 30 DAY),
    fecha_actualizacion = NOW()
WHERE id_tarjeta = :id_tarjeta;
```

---

## 5. Puntos de Datos Relacionados — Referencia Completa

| Campo | Tipo | Descripción |
|---|---|---|
| `id_tarjeta` | BIGINT | Identificador único de la tarjeta |
| `numero_tarjeta` | VARCHAR(19) | Número enmascarado de la tarjeta |
| `id_cuenta_bancaria` | BIGINT | Cuenta bancaria vinculada para débitos |
| `saldo_actual` | DECIMAL(12,2) | Saldo adeudado en el período actual |
| `limite_credito` | DECIMAL(12,2) | Límite máximo de crédito otorgado |
| `saldo_disponible` | DECIMAL(12,2) | Crédito disponible para uso (`limite - saldo_actual`) |
| `pago_minimo` | DECIMAL(12,2) | Monto mínimo requerido para el período |
| `fecha_limite_pago` | DATE | Fecha máxima para realizar el pago mínimo |
| `fecha_inicio_facturacion` | DATE | Inicio del ciclo de facturación vigente |
| `fecha_corte_facturacion` | DATE | Fin del ciclo; fecha de generación del estado de cuenta |
| `tasa_interes_anual` | DECIMAL(5,2) | Tasa de interés anual aplicada al saldo no pagado |
| `estado` | ENUM | Estado operativo de la tarjeta |

---

## 6. Consideraciones de Seguridad y Cumplimiento

- **PCI-DSS:** El número completo de tarjeta (`PAN`) nunca debe almacenarse en texto plano. Usar tokenización o enmascaramiento.
- **Auditoría:** Todas las actualizaciones de saldo deben registrarse en una tabla de auditoría con `timestamp`, usuario responsable y tipo de operación.
- **Transacciones atómicas:** Los débitos en cuenta bancaria y los abonos a la tarjeta deben ejecutarse dentro de una misma transacción de base de datos para garantizar consistencia (`ACID`).
- **Notificaciones:** El sistema debe generar alertas automáticas al titular cuando: (a) se acerque la `fecha_limite_pago` (7 días antes), (b) el pago mínimo no haya sido registrado, o (c) el saldo disponible sea inferior al 10% del límite de crédito.

---

## 7. Glosario

| Término | Definición |
|---|---|
| **Pago mínimo** | Monto menor aceptado por la entidad emisora en un ciclo de facturación |
| **Fecha límite de pago** | Fecha tope para efectuar el pago y evitar recargos |
| **Rango de facturación** | Período comprendido entre la fecha de inicio y la fecha de corte de un ciclo |
| **Saldo disponible** | Diferencia entre el límite de crédito y el saldo adeudado |
| **Fecha de corte** | Fecha en que cierra el ciclo de facturación y se genera el estado de cuenta |
| **Tasa moratoria** | Interés adicional aplicado por pagos realizados después de la fecha límite |

---

*Documento generado el 2026-04-27 | Versión 1.0 | Equipo de Ingeniería Financiera*
