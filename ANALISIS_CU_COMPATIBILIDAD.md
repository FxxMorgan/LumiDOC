# Análisis de Compatibilidad: Plataforma Luminova vs Unidades de Control (CU/ACU/ME/Gateway)

> **Fecha:** 16 de febrero de 2026  
> **Versión:** 1.0  
> **Alcance:** Backend + Frontend de la plataforma Luminova

---

## Tabla de Contenidos

1. [Resumen Ejecutivo](#1-resumen-ejecutivo)
2. [Estado Actual de la Plataforma](#2-estado-actual-de-la-plataforma)
3. [Requisitos por Tipo de Dispositivo](#3-requisitos-por-tipo-de-dispositivo)
4. [Análisis de Brechas (Gap Analysis)](#4-análisis-de-brechas-gap-analysis)
5. [Soporte de Variables Genéricas (No Solo Fases Eléctricas)](#5-soporte-de-variables-genéricas-no-solo-fases-eléctricas)
6. [Cambios Principales Necesarios](#6-cambios-principales-necesarios)
7. [Nuevo Modelo de Datos Propuesto](#7-nuevo-modelo-de-datos-propuesto)
8. [Nuevo Protocolo UDP Propuesto](#8-nuevo-protocolo-udp-propuesto)
9. [Orden de Implementación Recomendado](#9-orden-de-implementación-recomendado)

---

## 1. Resumen Ejecutivo

La plataforma Luminova actualmente **solo soporta monitoreo pasivo de variables eléctricas trifásicas (fases R/S/T)**. Para ser compatible con todo el ecosistema de dispositivos (CU, CU2, CU3, ACU, ME, Gateway), se requieren cambios profundos en:

| Área | Estado Actual | Estado Requerido |
|------|---------------|------------------|
| Recepción de datos | Solo lectura eléctrica trifásica | Variables genéricas (nivel, temperatura, agua, diesel, etc.) |
| Control de actuadores | ❌ No implementado | ✅ Envío de comandos bidireccional |
| Gateway como intermediario | ❌ Solo existe como tipo en enum | ✅ Lógica completa de routing |
| Módulos de expansión (ME) | ❌ No existe concepto | ✅ Registro jerárquico ACU → ME |
| Tipos de control | ❌ Solo plataforma (lectura) | ✅ Plataforma, botonera, sensor, estadístico |
| Periféricos/sensores | ❌ No soportado | ✅ Variables físicas arbitrarias |

---

## 2. Estado Actual de la Plataforma

### 2.1 Modelo de Datos Actual

```
User (admin/cliente)
  └── Empalme (punto de medición)
       ├── dispositivos[] (embebidos, tipo "medidor_trifasico")
       └── Lecturas[] (time series: faseR, faseS, faseT)
            └── Cada fase: voltaje, corriente, potencia, energía, frecuencia, factorPotencia
```

### 2.2 Protocolo UDP Actual

```
[empalmeId] [V_R] [I_R] [P_R] [E_R] [F_R] [FP_R] [V_S] [I_S] [P_S] [E_S] [F_S] [FP_S] [V_T] [I_T] [P_T] [E_T] [F_T] [FP_T] [señal]
```
- **20 valores** separados por espacios
- **Solo variables eléctricas**: voltaje, corriente, potencia, energía, frecuencia, factor de potencia
- **Solo trifásico**: siempre 3 fases (R, S, T)
- **Sin identificación de tipo de dispositivo** en el paquete

### 2.3 Capacidades Existentes

| Funcionalidad | Implementada | Detalle |
|---|---|---|
| Recepción UDP | ✅ | Batch processing, rate limiting |
| Time Series MongoDB | ✅ | Colección optimizada con índices |
| WebSocket tiempo real | ✅ | Socket.io con rooms por empalme |
| Alertas por umbrales | ✅ | Voltaje, corriente, frecuencia, FP |
| Dashboard + gráficas | ✅ | Agregaciones por período, costos eléctricos |
| CRUD dispositivos | ✅ | Pero desconectado del flujo real |
| Tipo `actuador` en enum | ✅ | Pero sin lógica de control |
| Tipo `gateway` en enum | ✅ | Pero sin lógica de routing |
| Envío de comandos | ❌ | No existe |
| Variables no eléctricas | ❌ | No soportado |
| Jerarquía gateway→CU→ME | ❌ | No existe |

---

## 3. Requisitos por Tipo de Dispositivo

### 3.1 CU — Unidad de Control Original (Alumbrado Público)

| Requisito | Detalle |
|---|---|
| **Medición** | Variables eléctricas (3 canales: voltaje, corriente, potencia, energía, frecuencia, FP) |
| **Control** | Encendido/apagado de luminarias |
| **Modos de control** | Estadístico (horario/timer) + Plataforma (Maestro remoto) |
| **Comunicación** | Vía Gateway (LoRa) → Plataforma (WiFi/GSM) |
| **Aplicación** | II (Iluminación Interior), IE (Iluminación Exterior) |

**Gap:** La plataforma recibe datos pero NO puede enviar comandos de encendido/apagado. No tiene concepto de control estadístico (schedulers/horarios).

### 3.2 CU2 — Unidad de Control Roller Screen

| Requisito | Detalle |
|---|---|
| **Medición** | Variables eléctricas del motor del roller |
| **Control** | Subir/bajar/detener roller screen |
| **Modos de control** | Botonera (Maestro local) + Plataforma (remoto) |
| **Comunicación** | Vía Gateway (LoRa) → Plataforma (WiFi/GSM) |
| **Aplicación** | RS (Roller Screen) |

**Gap:** No hay UI ni backend para comandos de tipo posicional (subir/bajar/detener). No hay concepto de "botonera como maestro" donde la plataforma registra el estado pero no lo controla siempre.

### 3.3 CU3 — Unidad de Control de Baños

| Requisito | Detalle |
|---|---|
| **Medición** | Variables eléctricas + potencialmente sensores de movimiento |
| **Control** | Encendido/apagado iluminación y ventilación |
| **Modos de control** | Sensor de movimiento + Botonera (Maestro) + Plataforma |
| **Comunicación** | Vía Gateway (LoRa) → Plataforma (WiFi/GSM) |
| **Aplicación** | Baños/servicios |

**Gap:** No hay recepción de datos de sensores de movimiento. No hay concepto de prioridad de control (sensor > botonera > plataforma).

### 3.4 ACU — Unidad de Control Avanzada

| Requisito | Detalle |
|---|---|
| **Medición** | 3 mediciones de variables eléctricas independientes |
| **Actuadores** | 3 actuadores controlables |
| **Control** | Solo plataforma |
| **Expansión** | Compatible con Módulos de Expansión (ME) |
| **Periféricos** | Compatible con sensores de otras variables físicas |
| **Comunicación** | Gateway + WiFi/GSM (fusión: ACU = Gateway + CU) |
| **Aplicación** | Cualquier instalación compleja |

**Gap:** El modelo actual solo soporta 1 lectura trifásica por empalme. La ACU necesita 3 mediciones independientes + 3 actuadores + enlace a MEs + periféricos arbitrarios.

### 3.5 ME — Módulo de Expansión

| Requisito | Detalle |
|---|---|
| **Medición** | 3 mediciones de variables eléctricas |
| **Actuadores** | 3 actuadores |
| **Control** | Solo plataforma (a través de la ACU padre) |
| **Comunicación** | Exclusivamente a través de la ACU (no tiene comunicación directa) |
| **Dependencia** | Siempre vinculado a una ACU |

**Gap:** No existe concepto de jerarquía padre-hijo entre dispositivos. No hay routing de comandos ACU → ME.

### 3.6 Gateway

| Requisito | Detalle |
|---|---|
| **Función** | Intermediario entre CUs (LoRa) y Plataforma (WiFi/GSM) |
| **Comunicación descendente** | LoRa hacia las CUs |
| **Comunicación ascendente** | WiFi o GSM hacia la plataforma |
| **Gestión** | Registro de CUs conectadas, retransmisión de datos y comandos |

**Gap:** El Gateway no existe como entidad funcional. No hay lógica de routing, ni registro de qué CUs están detrás de cada Gateway.

---

## 4. Análisis de Brechas (Gap Analysis)

### 4.1 Backend — Brechas Críticas

| # | Brecha | Impacto | Prioridad |
|---|--------|---------|-----------|
| B1 | **No hay bidireccionalidad** — solo recibe datos, no envía comandos | No se puede controlar ningún actuador | 🔴 Crítica |
| B2 | **Modelo de lectura rígido** — solo faseR/faseS/faseT con campos eléctricos fijos | No se pueden recibir datos de agua, diesel, temperatura, nivel, etc. | 🔴 Crítica |
| B3 | **Sin jerarquía de dispositivos** — no hay relación Gateway→CU→ME | No se puede representar la topología real | 🔴 Crítica |
| B4 | **Sin modelo de Actuador operativo** — tipo existe pero sin lógica | Imposible operar relés, motores, etc. | 🔴 Crítica |
| B5 | **Sin concepto de "modo de control"** — estadístico, botonera, sensor, plataforma | No se sabe quién tiene el control en cada momento | 🟠 Alta |
| B6 | **EmpalmeId = DispositivoId** — sin distinción real | No se pueden tener múltiples dispositivos reales bajo un mismo punto de medición | 🟠 Alta |
| B7 | **Sin protocolo de comandos** — no hay modelo de comando, cola, confirmación | Los actuadores no pueden recibir instrucciones | 🔴 Crítica |
| B8 | **Sin soporte multi-medición** — 1 empalme = 1 lectura trifásica | ACU/ME necesitan 3 mediciones independientes | 🟠 Alta |
| B9 | **Gateway sin lógica** — solo hay un enum | No se pueden enrutar datos ni comandos a través del gateway | 🟠 Alta |
| B10 | **Sin schedulers/horarios** — control estadístico requiere programación temporal | No se puede programar encendido/apagado automático | 🟡 Media |
| B11 | **Alertas solo eléctricas** — umbrales de voltaje, corriente, frecuencia, FP | No se pueden configurar alertas para nivel de agua, temperatura, etc. | 🟡 Media |

### 4.2 Frontend — Brechas

| # | Brecha | Impacto |
|---|--------|---------|
| F1 | **Sin UI de control** — no hay botones de encendido/apagado, subir/bajar | No se pueden operar actuadores |
| F2 | **Sin vista de topología** — no se ve la jerarquía Gateway→CU→ME | No se entiende la red |
| F3 | **Dashboard solo eléctrico** — gráficas fijas de voltaje/corriente/potencia | No se pueden visualizar otras variables |
| F4 | **Tab Dispositivos vacío** — hardcoded `[]` | No se ven los dispositivos reales del empalme |
| F5 | **Sin configuración de schedules** — no hay UI para horarios | Control estadístico imposible |
| F6 | **Sin indicador de modo de control** — no se sabe si controla plataforma, botonera o sensor | Confusión operativa |

---

## 5. Soporte de Variables Genéricas (No Solo Fases Eléctricas)

### 5.1 Problema Actual

El modelo `Lectura` está fijo a:
```typescript
faseR: { voltaje, corriente, potencia, energia, frecuencia, factorPotencia }
faseS: { voltaje, corriente, potencia, energia, frecuencia, factorPotencia }
faseT: { voltaje, corriente, potencia, energia, frecuencia, factorPotencia }
```

### 5.2 Solución Propuesta: Canales Genéricos

En vez de `faseR`, `faseS`, `faseT` con campos eléctricos fijos, usar **3 canales genéricos** donde cada canal tiene **6 variables numericas** cuyo significado depende del `tipoLectura`:

```typescript
// Nuevo modelo: LecturaGenerica
interface ILecturaGenerica {
  timestamp: Date;
  empalmeId: string;
  dispositivoId: string;
  
  tipoLectura: TipoLectura; // Define qué significan los valores
  
  canal1: IDatosCanal;  // Antes: faseR
  canal2: IDatosCanal;  // Antes: faseS
  canal3: IDatosCanal;  // Antes: faseT
  
  señal_dbm?: number;
}

interface IDatosCanal {
  var1: number;  // Antes: voltaje       | Puede ser: nivel, temperatura, presión...
  var2: number;  // Antes: corriente     | Puede ser: caudal, humedad, RPM...
  var3: number;  // Antes: potencia      | Puede ser: potencia, flujo...
  var4: number;  // Antes: energia       | Puede ser: energía acumulada, volumen...
  var5: number;  // Antes: frecuencia    | Puede ser: frecuencia, velocidad...
  var6: number;  // Antes: factorPotencia | Puede ser: factor, eficiencia...
}
```

### 5.3 Mapeo de Variables por Tipo de Lectura

| TipoLectura | var1 | var2 | var3 | var4 | var5 | var6 | Unidades |
|---|---|---|---|---|---|---|---|
| `electrica_trifasica` | Voltaje (V) | Corriente (A) | Potencia (kW) | Energía (kWh) | Frecuencia (Hz) | Factor Potencia | V, A, kW, kWh, Hz, - |
| `electrica_monofasica` | Voltaje (V) | Corriente (A) | Potencia (kW) | Energía (kWh) | Frecuencia (Hz) | Factor Potencia | Igual, 1 solo canal |
| `agua` | Nivel (m) | Caudal (L/min) | Presión (bar) | Volumen acum. (m³) | Temperatura (°C) | pH | m, L/min, bar, m³, °C, - |
| `diesel_generador` | Voltaje (V) | Corriente (A) | Potencia (kW) | Energía (kWh) | Nivel combustible (%) | RPM | V, A, kW, kWh, %, RPM |
| `temperatura_hvac` | Temp. ambiente (°C) | Temp. objetivo (°C) | Humedad (%) | Consumo (kWh) | Velocidad fan (%) | Estado (0/1) | °C, °C, %, kWh, %, bool |
| `sensor_movimiento` | Detecciones/min | Estado (0/1) | Luminosidad (lux) | Horas acum. (h) | Temperatura (°C) | - | det/min, bool, lux, h, °C, - |
| `roller_screen` | Posición (%) | Corriente motor (A) | Potencia (kW) | Ciclos acum. | Estado (0=detenido,1=subiendo,2=bajando) | - | %, A, kW, ciclos, estado, - |

### 5.4 Compatibilidad Retroactiva

El mapeo para `electrica_trifasica` es **idéntico** al modelo actual:
- `canal1` = `faseR`, `canal2` = `faseS`, `canal3` = `faseT`
- `var1` = `voltaje`, `var2` = `corriente`, ..., `var6` = `factorPotencia`

**Esto permite migrar sin perder datos existentes.** Solo se renombran los campos y se agrega el campo `tipoLectura`.

---

## 6. Cambios Principales Necesarios

### CAMBIO 1: Nuevo Modelo de Dispositivo con Jerarquía 🔴

**Archivo afectado:** `backend/src/models/Dispositivo.ts`

```typescript
// NUEVOS TIPOS DE DISPOSITIVO
enum TipoDispositivo {
  // Unidades de control
  CU = 'CU',           // Control original (AP)
  CU2 = 'CU2',         // Control Roller Screen
  CU3 = 'CU3',         // Control Baños
  ACU = 'ACU',          // Unidad avanzada
  ME = 'ME',            // Módulo de expansión
  GATEWAY = 'GATEWAY',  // Gateway LoRa↔WiFi/GSM
  
  // Tipos legacy (mantener por compatibilidad)
  MEDIDOR_TRIFASICO = 'medidor_trifasico',
  SENSOR = 'sensor',
  ACTUADOR = 'actuador',
}

// NUEVO: Modos de control
enum ModoControl {
  PLATAFORMA = 'plataforma',     // Controlado desde la plataforma (Maestro remoto)
  BOTONERA = 'botonera',         // Controlado por botonera física (Maestro local)
  SENSOR = 'sensor',             // Controlado por sensor (movimiento, luz, etc.)
  ESTADISTICO = 'estadistico',   // Controlado por horario/programación
  MANUAL = 'manual',             // Control manual directo  
}

// NUEVO: Jerarquía
interface IDispositivoNuevo {
  dispositivoId: string;
  tipo: TipoDispositivo;
  
  // Jerarquía
  gatewayId?: string;        // ID del Gateway padre (para CU, CU2, CU3)
  acuId?: string;            // ID del ACU padre (para ME)
  
  // Capacidades
  canalesMedicion: number;    // 0 (gateway), 1 (sensor), 3 (CU/ACU/ME)
  cantidadActuadores: number; // 0 (gateway/sensor), 1 (CU/CU2/CU3), 3 (ACU/ME)
  tipoLectura: TipoLectura;  // Qué tipo de datos reporta
  
  // Control
  modosControlSoportados: ModoControl[];  // Qué modos acepta
  modoControlActual: ModoControl;          // Modo activo ahora
  
  // Aplicación
  aplicacion?: AplicacionDispositivo;  // II, IE, RS, UPA, etc.
  
  // Estado de actuadores
  actuadores: IActuador[];
  
  // Periféricos conectados (solo ACU)
  perifericosIds?: string[];
  modulosExpansionIds?: string[];  // Solo ACU
}

// NUEVO: Actuador
interface IActuador {
  actuadorId: string;          // "ACT-1", "ACT-2", "ACT-3"
  nombre: string;              // "Luminaria zona A", "Roller 1"
  tipo: TipoActuador;         // relay, motor, dimmer
  estado: boolean;             // true = encendido/activo
  posicion?: number;           // 0-100% (para rollers, dimmers)
  ultimoCambio: Date;
  cambiadoPor: ModoControl;   // Quién cambió el estado por última vez
}

enum TipoActuador {
  RELAY = 'relay',             // On/Off simple
  MOTOR = 'motor',             // Subir/bajar (roller)
  DIMMER = 'dimmer',           // Regulable 0-100%
  VALVULA = 'valvula',         // Apertura/cierre
}

enum AplicacionDispositivo {
  II = 'II',     // Iluminación Interior
  IE = 'IE',     // Iluminación Exterior
  RS = 'RS',     // Roller Screen
  UPA = 'UPA',   // Centro de comida Shell/ENEX
  DIESEL = 'DIESEL',  // Diesel Generador
  AGUA = 'AGUA',      // Medición de agua
  BANOS = 'BANOS',    // Baños/servicios
  HVAC = 'HVAC',      // Climatización
  GENERAL = 'GENERAL', // Uso general
}
```

### CAMBIO 2: Modelo de Lectura Genérica 🔴

**Archivo afectado:** `backend/src/models/Lectura.ts`

```typescript
enum TipoLectura {
  ELECTRICA_TRIFASICA = 'electrica_trifasica',
  ELECTRICA_MONOFASICA = 'electrica_monofasica',  
  AGUA = 'agua',
  DIESEL_GENERADOR = 'diesel_generador',
  TEMPERATURA_HVAC = 'temperatura_hvac',
  SENSOR_MOVIMIENTO = 'sensor_movimiento',
  ROLLER_SCREEN = 'roller_screen',
  CUSTOM = 'custom',   // Variables definidas por el usuario
}

interface ILecturaGenerica {
  timestamp: Date;
  empalmeId: string;
  dispositivoId: string;
  tipoLectura: TipoLectura;
  
  // 3 canales × 6 variables = 18 valores numéricos (misma cantidad que ahora)
  canal1: IDatosCanal;
  canal2: IDatosCanal;
  canal3: IDatosCanal;
  
  // Estado de actuadores al momento de la lectura
  estadoActuadores?: {
    act1: boolean;
    act2: boolean;
    act3: boolean;
  };
  
  señal_dbm?: number;
}

interface IDatosCanal {
  var1: number;
  var2: number;
  var3: number;
  var4: number;
  var5: number;
  var6: number;
}
```

**Dato clave:** Se mantiene la **misma cantidad de variables numéricas** (18 = 3×6). El campo `tipoLectura` determina qué significa cada variable. Esto hace que el protocolo UDP se mantenga casi igual en estructura.

### CAMBIO 3: Sistema de Comandos Bidireccional 🔴

**Archivos nuevos:**
- `backend/src/models/Comando.ts`
- `backend/src/services/command.service.ts`
- `backend/src/controllers/comando.controller.ts`
- `backend/src/routes/comando.routes.ts`

```typescript
// Nuevo modelo: Comando
enum TipoComando {
  ENCENDER = 'encender',
  APAGAR = 'apagar',
  TOGGLE = 'toggle',
  POSICION = 'posicion',      // Para rollers (0-100%)
  DIMMER = 'dimmer',          // Nivel de dimmer (0-100%)
  SCHEDULE = 'schedule',      // Programar horario
  CONFIG = 'config',          // Cambiar configuración
  RESET = 'reset',            // Reiniciar dispositivo
  STATUS = 'status',          // Solicitar estado
}

enum EstadoComando {
  PENDIENTE = 'pendiente',
  ENVIADO = 'enviado',
  CONFIRMADO = 'confirmado',
  ERROR = 'error',
  TIMEOUT = 'timeout',
}

interface IComando {
  comandoId: string;
  dispositivoId: string;       // A quién va dirigido
  gatewayId?: string;          // A través de qué gateway
  
  tipo: TipoComando;
  parametros: {
    actuadorId?: string;       // "ACT-1", "ACT-2", "ACT-3"
    valor?: number;            // 0-100 para posición/dimmer
    horario?: {                // Para schedules
      encendido: string;       // "HH:MM"
      apagado: string;         // "HH:MM"
      diasSemana: number[];    // [0-6]
    };
  };
  
  estado: EstadoComando;
  enviadoEn?: Date;
  confirmadoEn?: Date;
  errorMensaje?: string;
  
  // Auditoría
  creadoPor: ObjectId;        // Usuario que creó el comando
  timestamp: Date;
}
```

**Flujo de comando:**
```
Frontend → API REST (POST /comandos) → CommandService 
    → Cola en MongoDB (estado: pendiente)
    → WebSocket/UDP al Gateway  
    → Gateway retransmite por LoRa al CU/ACU
    → CU/ACU ejecuta y confirma
    → Gateway envía confirmación UDP
    → CommandService actualiza estado → WebSocket al Frontend
```

### CAMBIO 4: Modelo de Gateway Funcional 🟠

**Archivos afectados:**
- `backend/src/models/Dispositivo.ts` (refactorizar)
- `backend/src/services/udp-receiver.service.ts` (nuevo parser)

```typescript
interface IGateway {
  gatewayId: string;
  nombre: string;
  
  // Comunicación ascendente (al servidor)
  tipoConexion: 'wifi' | 'gsm' | 'ethernet';
  direccionIP: string;
  puerto: number;
  
  // Dispositivos conectados (LoRa descendente)
  dispositivosIds: string[];   // CUs conectadas a este gateway
  
  // Estado
  estado: 'online' | 'offline' | 'mantenimiento';
  ultimaConexion: Date;
  señalGSM?: number;           // dBm si usa GSM
  
  // Ubicación
  ubicacionGPS?: { lat: number; lng: number };
}
```

### CAMBIO 5: Sistema de Schedules (Control Estadístico) 🟡

**Archivos nuevos:**
- `backend/src/models/Schedule.ts`
- `backend/src/services/scheduler.service.ts`

```typescript
interface ISchedule {
  scheduleId: string;
  dispositivoId: string;
  actuadorId: string;
  
  nombre: string;
  habilitado: boolean;
  
  // Programación
  tipo: 'horario' | 'astronomico' | 'intervalo';
  
  // Horario fijo
  horaEncendido?: string;        // "18:30"
  horaApagado?: string;          // "06:00"
  
  // Astronómico (basado en ubicación GPS)
  eventoEncendido?: 'amanecer' | 'atardecer';
  eventoApagado?: 'amanecer' | 'atardecer';
  offsetMinutos?: number;        // +30 = 30 min después del evento
  
  // Días activos
  diasSemana: boolean[];         // [lun, mar, mié, jue, vie, sáb, dom]
  
  // Metadata
  creadoPor: ObjectId;
  ultimaEjecucion?: Date;
}
```

### CAMBIO 6: Protocolo UDP Extendido 🔴

**Archivo afectado:** `backend/src/services/udp-receiver.service.ts`

Nuevo formato que mantiene compatibilidad:

```
[VERSIÓN]:[GATEWAY_ID]:[DISPOSITIVO_ID]:[TIPO]:[V1] [V2] ... [V18]:[SEÑAL]:[ACT1,ACT2,ACT3]
```

| Campo | Descripción | Ejemplo |
|---|---|---|
| VERSIÓN | Versión del protocolo | `v2` |
| GATEWAY_ID | ID del gateway emisor | `GW-001` (o `direct` si no hay gateway) |
| DISPOSITIVO_ID | ID del dispositivo origen | `CU-005` |
| TIPO | Tipo de lectura | `electrica_trifasica`, `agua`, `diesel`, etc. |
| V1..V18 | 18 valores numéricos separados por espacio | `220.5 15.2 3.34 ...` |
| SEÑAL | Señal en dBm | `-67` |
| ACT1,ACT2,ACT3 | Estado de actuadores (0/1 o posición 0-100) | `1,0,75` |

**Ejemplo de paquete eléctrico trifásico (CU):**
```
v2:GW-001:CU-005:electrica_trifasica:220.5 15.2 3.34 1250.0 50.01 0.95 221.0 14.8 3.26 1248.5 50.01 0.96 219.8 15.5 3.41 1252.3 50.00 0.94:-67:1,0,0
```

**Ejemplo de paquete de agua:**
```
v2:direct:ACU-002:agua:3.5 12.4 2.1 1500.0 18.5 7.2 0 0 0 0 0 0 0 0 0 0 0 0:-45:0,1,0
```

**Compatibilidad v1:** Si el paquete no empieza con `v2:`, se parsea con el formato actual (solo 20 valores separados por espacio). Esto permite migración gradual.

### CAMBIO 7: Alertas Genéricas 🟡

**Archivo afectado:** `backend/src/models/Alerta.ts`, `backend/src/services/alert.service.ts`

```typescript
// Nuevos tipos de alerta
enum TipoAlerta {
  // Eléctricos (existentes)
  SOBRETENSION = 'sobretension',
  BAJA_TENSION = 'baja_tension',
  SOBRECORRIENTE = 'sobrecorriente',
  FACTOR_POTENCIA_BAJO = 'factor_potencia_bajo',
  FRECUENCIA_ANORMAL = 'frecuencia_anormal',
  DISPOSITIVO_OFFLINE = 'dispositivo_offline',
  
  // Nuevos - Agua
  NIVEL_AGUA_ALTO = 'nivel_agua_alto',
  NIVEL_AGUA_BAJO = 'nivel_agua_bajo',
  PRESION_AGUA_ANORMAL = 'presion_agua_anormal',
  
  // Nuevos - Diesel
  COMBUSTIBLE_BAJO = 'combustible_bajo',
  RPM_ANORMAL = 'rpm_anormal',
  
  // Nuevos - Ambiente
  TEMPERATURA_ALTA = 'temperatura_alta',
  TEMPERATURA_BAJA = 'temperatura_baja',
  HUMEDAD_ANORMAL = 'humedad_anormal',
  
  // Nuevos - Control
  ACTUADOR_ERROR = 'actuador_error',
  COMANDO_TIMEOUT = 'comando_timeout',
  GATEWAY_OFFLINE = 'gateway_offline',
  
  // Genérico
  UMBRAL_PERSONALIZADO = 'umbral_personalizado',
}
```

### CAMBIO 8: Configuración de Variables por Tipo de Lectura 🟡

**Archivo nuevo:** `backend/src/models/DefinicionVariable.ts`

Este modelo define cómo se llaman y qué unidades tienen las variables según el tipo de lectura:

```typescript
interface IDefinicionVariable {
  tipoLectura: TipoLectura;
  variables: {
    canal: 1 | 2 | 3;
    posicion: 1 | 2 | 3 | 4 | 5 | 6;  // var1..var6
    nombre: string;           // "voltaje", "nivel", "temperatura"
    nombreDisplay: string;    // "Voltaje", "Nivel de Agua", "Temperatura"
    unidad: string;           // "V", "m", "°C", "A", "kW"
    min?: number;             // Valor mínimo esperado
    max?: number;             // Valor máximo esperado
    decimales: number;        // Decimales para display
    icono?: string;           // Icono para el frontend
  }[];
}
```

**Registros predefinidos:**

```typescript
// Eléctrica trifásica (compatibilidad actual)
{
  tipoLectura: 'electrica_trifasica',
  variables: [
    { canal: 1, posicion: 1, nombre: 'voltaje', nombreDisplay: 'Voltaje Fase R', unidad: 'V', min: 200, max: 240, decimales: 1 },
    { canal: 1, posicion: 2, nombre: 'corriente', nombreDisplay: 'Corriente Fase R', unidad: 'A', min: 0, max: 100, decimales: 2 },
    { canal: 1, posicion: 3, nombre: 'potencia', nombreDisplay: 'Potencia Fase R', unidad: 'kW', decimales: 2 },
    { canal: 1, posicion: 4, nombre: 'energia', nombreDisplay: 'Energía Fase R', unidad: 'kWh', decimales: 1 },
    { canal: 1, posicion: 5, nombre: 'frecuencia', nombreDisplay: 'Frecuencia', unidad: 'Hz', min: 49.5, max: 50.5, decimales: 2 },
    { canal: 1, posicion: 6, nombre: 'factorPotencia', nombreDisplay: 'Factor de Potencia', unidad: '', min: 0, max: 1, decimales: 3 },
    // canal 2 = Fase S (mismas variables)
    // canal 3 = Fase T (mismas variables)
  ]
}

// Agua
{
  tipoLectura: 'agua',
  variables: [
    { canal: 1, posicion: 1, nombre: 'nivel', nombreDisplay: 'Nivel de Agua', unidad: 'm', min: 0, max: 10, decimales: 2 },
    { canal: 1, posicion: 2, nombre: 'caudal', nombreDisplay: 'Caudal', unidad: 'L/min', min: 0, decimales: 1 },
    { canal: 1, posicion: 3, nombre: 'presion', nombreDisplay: 'Presión', unidad: 'bar', min: 0, max: 10, decimales: 2 },
    { canal: 1, posicion: 4, nombre: 'volumen', nombreDisplay: 'Volumen Acumulado', unidad: 'm³', decimales: 1 },
    { canal: 1, posicion: 5, nombre: 'temperatura', nombreDisplay: 'Temperatura del Agua', unidad: '°C', min: 0, max: 100, decimales: 1 },
    { canal: 1, posicion: 6, nombre: 'ph', nombreDisplay: 'pH', unidad: '', min: 0, max: 14, decimales: 1 },
  ]
}

// Diesel Generador
{
  tipoLectura: 'diesel_generador',
  variables: [
    { canal: 1, posicion: 1, nombre: 'voltaje', nombreDisplay: 'Voltaje', unidad: 'V', decimales: 1 },
    { canal: 1, posicion: 2, nombre: 'corriente', nombreDisplay: 'Corriente', unidad: 'A', decimales: 2 },
    { canal: 1, posicion: 3, nombre: 'potencia', nombreDisplay: 'Potencia', unidad: 'kW', decimales: 2 },
    { canal: 1, posicion: 4, nombre: 'energia', nombreDisplay: 'Energía', unidad: 'kWh', decimales: 1 },
    { canal: 1, posicion: 5, nombre: 'combustible', nombreDisplay: 'Nivel Combustible', unidad: '%', min: 0, max: 100, decimales: 0 },
    { canal: 1, posicion: 6, nombre: 'rpm', nombreDisplay: 'RPM', unidad: 'RPM', decimales: 0 },
  ]
}
```

---

## 7. Nuevo Modelo de Datos Propuesto

### Diagrama de Entidades

```
┌──────────────┐
│     User     │
│ (admin/cli)  │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐       ┌───────────────┐
│   Empalme    │──────▶│   Gateway     │  (1 empalme puede tener N gateways)
│ (sitio/loc)  │       │ (WiFi/GSM)    │
└──────┬───────┘       └───────┬───────┘
       │                       │ 1:N (LoRa)
       │                       ▼
       │               ┌───────────────┐
       │               │  Dispositivo  │  CU, CU2, CU3, ACU
       │               │  (con actua-  │
       │               │   dores)      │
       │               └───────┬───────┘
       │                       │ 1:N (solo ACU→ME)
       │                       ▼
       │               ┌───────────────┐
       │               │      ME       │  Módulos de Expansión
       │               │ (3 med + 3    │
       │               │  actuadores)  │
       │               └───────────────┘
       │
       │ (lecturas de todos los disp.)
       ▼
┌──────────────┐       ┌───────────────┐       ┌───────────────┐
│   Lectura    │       │   Comando     │       │   Schedule    │
│ (time series)│       │ (cola FIFO)   │       │ (horarios)    │
│ 3 canales ×  │       │ encender/     │       │ control       │
│ 6 variables  │       │ apagar/pos    │       │ estadístico   │
└──────────────┘       └───────────────┘       └───────────────┘
                              ▲
                              │
                       ┌──────┴────────┐
                       │ DefinicionVar │
                       │ (metadatos de │
                       │  variables)   │
                       └───────────────┘
```

### Relaciones Clave

| Relación | Tipo | Descripción |
|---|---|---|
| User → Empalme | 1:N | Un cliente tiene N empalmes/sitios |
| Empalme → Gateway | 1:N | Un sitio puede tener N gateways |
| Gateway → Dispositivo (CU/ACU) | 1:N | Un gateway conecta N CUs vía LoRa |
| ACU → ME | 1:N | Un ACU tiene N módulos de expansión |
| Dispositivo → Lectura | 1:N | Cada dispositivo genera lecturas time series |
| Dispositivo → Comando | 1:N | Cada dispositivo puede recibir comandos |
| Dispositivo → Schedule | 1:N | Cada dispositivo puede tener programaciones |
| ACU = Gateway + CU | fusión | Una ACU es un Gateway con capacidades de CU |

---

## 8. Nuevo Protocolo UDP Propuesto

### 8.1 Formato v2 (datos → servidor)

```
v2:[GATEWAY_ID]:[DISPOSITIVO_ID]:[TIPO_LECTURA]:[18 valores separados por espacio]:[SEÑAL]:[ACT1,ACT2,ACT3]
```

### 8.2 Formato v2 (servidor → gateway, comandos)

```
CMD:[GATEWAY_ID]:[DISPOSITIVO_ID]:[TIPO_COMANDO]:[ACTUADOR_ID]:[VALOR]:[COMANDO_ID]
```

Ejemplos:
```
CMD:GW-001:CU-005:encender:ACT-1:1:CMD-20260216-001
CMD:GW-001:CU2-003:posicion:ACT-1:75:CMD-20260216-002
CMD:direct:ACU-001:apagar:ACT-2:0:CMD-20260216-003
```

### 8.3 Formato v2 (gateway → servidor, confirmación)

```
ACK:[COMANDO_ID]:[ESTADO]:[TIMESTAMP]
```

Ejemplo:
```
ACK:CMD-20260216-001:confirmado:1739721600
ACK:CMD-20260216-002:error:1739721600:motor_bloqueado
```

### 8.4 Compatibilidad v1

```javascript
// En udp-receiver.service.ts
parsePacket(msg: Buffer) {
  const raw = msg.toString().trim();
  
  // Detectar versión
  if (raw.startsWith('v2:')) {
    return this.parseV2(raw);
  } else if (raw.startsWith('ACK:')) {
    return this.parseACK(raw);
  } else {
    return this.parseV1Legacy(raw); // Formato actual sin cambios
  }
}
```

---

## 9. Orden de Implementación Recomendado

### Fase 1: Fundaciones (2-3 semanas) 🔴 Crítica

| # | Tarea | Archivos | Complejidad |
|---|--|---|---|
| 1.1 | Refactorizar modelo `Dispositivo` con jerarquía (tipos CU/CU2/CU3/ACU/ME/Gateway) | `Dispositivo.ts` | Alta |
| 1.2 | Crear modelo `LecturaGenerica` con canales + `tipoLectura` | `Lectura.ts` | Alta |
| 1.3 | Crear modelo `DefinicionVariable` | Nuevo archivo | Media |
| 1.4 | Migrar datos existentes: `faseR→canal1`, `faseS→canal2`, `faseT→canal3` | Script de migración | Media |
| 1.5 | Actualizar UDP receiver con parser v2 + compatibilidad v1 | `udp-receiver.service.ts` | Alta |
| 1.6 | Actualizar data generator para generar con formato genérico | `data-generator.service.ts` | Media |

### Fase 2: Control y Comandos (2-3 semanas) 🔴 Crítica

| # | Tarea | Archivos | Complejidad |
|---|--|---|---|
| 2.1 | Crear modelo `Comando` | Nuevo archivo | Media |
| 2.2 | Crear `CommandService` con cola, envío UDP, y confirmaciones | Nuevo archivo | Alta |
| 2.3 | Crear endpoints REST `POST/GET /comandos` | Nuevo controller + routes | Media |
| 2.4 | Agregar eventos WebSocket para comandos (`comando:enviado`, `comando:confirmado`) | `socket.service.ts` | Media |
| 2.5 | Implementar envío UDP bidireccional (servidor → gateway) | `udp-receiver.service.ts` | Alta |
| 2.6 | Crear modelo `Actuador` embebido en dispositivos | `Dispositivo.ts` | Media |

### Fase 3: Gateway y Topología (1-2 semanas) 🟠 Alta

| # | Tarea | Archivos | Complejidad |
|---|--|---|---|
| 3.1 | Implementar lógica de Gateway (registro, routing, heartbeat) | `gateway.service.ts` (nuevo) | Alta |
| 3.2 | Crear endpoint para registrar CUs bajo un Gateway | Controller existente | Media |
| 3.3 | Crear endpoint para registrar MEs bajo una ACU | Controller existente | Media |
| 3.4 | Vista de topología en frontend | Nuevo componente | Media |

### Fase 4: Schedules y Control Estadístico (1-2 semanas) 🟡 Media

| # | Tarea | Archivos | Complejidad |
|---|--|---|---|
| 4.1 | Crear modelo `Schedule` | Nuevo archivo | Media |
| 4.2 | Crear `SchedulerService` con cron jobs | Nuevo archivo | Alta |
| 4.3 | Crear CRUD de schedules | Controller + routes | Media |
| 4.4 | UI de configuración de horarios | Nuevo componente frontend | Media |

### Fase 5: Frontend Multi-Variable (2-3 semanas) 🟡 Media

| # | Tarea | Archivos | Complejidad |
|---|--|---|---|
| 5.1 | Dashboard genérico que muestre variables según `tipoLectura` | `DashboardPage.tsx` refactor | Alta |
| 5.2 | Gráficas dinámicas basadas en `DefinicionVariable` | Componentes de gráficas | Alta |
| 5.3 | Panel de control de actuadores (on/off, posición) | Nuevo componente | Media |
| 5.4 | Vista de estado por dispositivo con modo de control activo | Nuevo componente | Media |
| 5.5 | Alertas genéricas con umbrales configurables por tipo de variable | Refactor alertas | Media |

### Fase 6: Integración y Testing (1-2 semanas) 🟡 Media

| # | Tarea | Archivos | Complejidad |
|---|--|---|---|
| 6.1 | Tests de integración para protocolo v2 | Tests nuevos | Media |
| 6.2 | Tests de flujo de comandos completo | Tests nuevos | Media |
| 6.3 | Simulador de Gateway + CUs para desarrollo | `data-generator.service.ts` | Media |
| 6.4 | Documentación actualizada del protocolo | MD files | Baja |

---

## Resumen de Esfuerzo Estimado

| Fase | Duración | Criticidad |
|------|----------|------------|
| Fase 1: Fundaciones | 2-3 semanas | 🔴 Bloqueante |
| Fase 2: Control y Comandos | 2-3 semanas | 🔴 Bloqueante |
| Fase 3: Gateway y Topología | 1-2 semanas | 🟠 Alta |
| Fase 4: Schedules | 1-2 semanas | 🟡 Puede diferirse |
| Fase 5: Frontend Multi-Variable | 2-3 semanas | 🟡 Puede diferirse |
| Fase 6: Integration & Testing | 1-2 semanas | 🟡 Continua |
| **TOTAL** | **~9-15 semanas** | |

---

## Nota sobre Variables Genéricas

El concepto clave es que **la cantidad de datos numéricos no cambia** (18 valores = 3 canales × 6 variables). Lo que cambia es el **significado** de cada valor según el `tipoLectura`. Esto permite:

1. **Reutilizar toda la infraestructura** de time series, índices, batch processing, WebSocket
2. **No romper el protocolo UDP** — misma estructura de 18 números
3. **Dashboards configurables** — el frontend consulta `DefinicionVariable` para saber qué nombre, unidad e ícono mostrar para cada campo
4. **Alertas universales** — los umbrales se configuran contra `var1`, `var2`, etc., y el display muestra el nombre real según el tipo

Ejemplo: donde antes era `faseR.voltaje = 220.5V`, ahora para un sensor de agua es `canal1.var1 = 3.5m` (nivel). **Misma posición, diferente interpretación.**
