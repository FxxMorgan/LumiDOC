# ✅ Día 9 Completado - Sistema de Alertas Inteligente

**Fecha:** 6 de Enero, 2026  
**Objetivo:** Implementar sistema completo de alertas automáticas con detección de umbrales, notificaciones y gestión de configuraciones

---

## 📋 Tareas Completadas

### 1. ✅ Modelos de Alerta y Configuración

**Archivo:** `src/models/Alerta.ts` (270 líneas)

**Características implementadas:**

#### Modelo de Alerta
**Schema principal:**
```typescript
{
  empalmeId: string,           // Empalme que generó la alerta
  tipo: TipoAlerta,            // Tipo de alerta (enum)
  severidad: SeveridadAlerta,  // Alta/Media/Baja
  estado: EstadoAlerta,        // Activa/Reconocida/Resuelta
  mensaje: string,             // Descripción del problema
  valor: number,               // Valor que disparó la alerta
  umbral: number,              // Umbral configurado
  fase: 'R' | 'S' | 'T',      // Fase afectada (opcional)
  timestamp: Date,             // Momento de la alerta
  reconocidaPor: ObjectId,     // Usuario que reconoció
  reconocidaEn: Date,          // Momento de reconocimiento
  resueltaEn: Date,            // Momento de resolución
  notas: string                // Notas adicionales
}
```

**Tipos de Alerta:**
- `SOBRETENSION` - Voltaje por encima del umbral
- `BAJA_TENSION` - Voltaje por debajo del umbral
- `SOBRECORRIENTE` - Corriente excesiva
- `FACTOR_POTENCIA_BAJO` - Factor de potencia bajo
- `FRECUENCIA_ANORMAL` - Frecuencia fuera de rango
- `DISPOSITIVO_OFFLINE` - Sin datos por tiempo prolongado

#### Modelo ConfiguracionAlerta
**Schema de configuración:**
```typescript
{
  empalmeId: string,
  tipo: TipoAlerta,
  habilitada: boolean,
  umbralMinimo: number,        // Umbral inferior
  umbralMaximo: number,        // Umbral superior
  notificarPorEmail: boolean,
  emailsNotificacion: string[],
  tiempoOffline: number        // Minutos para considerar offline
}
```

**Métodos estáticos:**
- `crearAlerta()` - Crea nueva alerta con cálculo automático de severidad
- `resolverAlerta()` - Marca como resuelta con timestamp
- `obtenerActivas()` - Alertas activas por empalme
- `obtenerHistorial()` - Historial con filtros de fecha y tipo
- `obtenerConfiguracion()` - Config activa de un empalme
- `obtenerConfiguracionPorTipo()` - Config específica de un tipo

---

### 2. ✅ Servicio de Alertas (AlertService)

**Archivo:** `src/services/alert.service.ts` (413 líneas)

**Características implementadas:**

#### Verificación Automática de Umbrales
```typescript
verificarLectura(empalmeId, faseR, faseS, faseT)
```

**Por cada fase verifica:**
- Sobretensión (default: >240V)
- Baja tensión (default: <200V)
- Sobrecorriente (default: >100A)
- Factor de potencia bajo (default: <0.85)
- Frecuencia anormal (default: 50±0.5Hz)

#### Umbrales por Defecto
```typescript
UMBRALES_DEFAULT = {
  SOBRETENSION: { max: 240 },
  BAJA_TENSION: { min: 200 },
  SOBRECORRIENTE: { max: 100 },
  FACTOR_POTENCIA_BAJO: { min: 0.85 },
  FRECUENCIA_ANORMAL: { min: 49.5, max: 50.5 },
  DISPOSITIVO_OFFLINE: { minutos: 5 }
}
```

#### Monitoreo de Dispositivos Offline
- Verificación cada 1 minuto
- Compara última lectura con timestamp actual
- Crea alerta si `lastSeen > tiempoOffline`
- Resuelve automáticamente cuando vuelve online

#### Gestión Inteligente de Alertas
- **No duplica alertas activas** del mismo tipo/fase/empalme
- **Resolución automática** cuando el valor vuelve a la normalidad
- **Emisión a WebSockets** en tiempo real
- **Integración con configuración** o uso de defaults

#### Estadísticas
```typescript
getEstadisticas(): {
  totalActivas: number,
  totalResueltas: number,
  porTipo: Array<{tipo, count}>,
  porSeveridad: Array<{severidad, count}>
}
```

---

### 3. ✅ Controladores de Alertas

**Archivo:** `src/controllers/alerta.controller.ts` (378 líneas)

**Endpoints implementados:**

#### Gestión de Alertas
1. **GET `/alertas/:empalmeId/activas`**
   - Retorna alertas activas de un empalme
   - Permisos: Admin o propietario del empalme

2. **GET `/alertas/:empalmeId/historial`**
   - Historial con filtros: fecha, tipo
   - Paginación incluida
   - Permisos: Admin o propietario

3. **PATCH `/alertas/:id/reconocer`**
   - Marca alerta como reconocida
   - Agrega notas opcionales
   - Permisos: Admin o propietario

4. **PATCH `/alertas/:id/resolver`**
   - Resolución manual de alerta
   - Solo Admin (resolución automática es diferente)

#### Gestión de Configuración
5. **GET `/alertas/:empalmeId/configuracion`**
   - Obtiene configuraciones del empalme
   - Permisos: Admin o propietario

6. **POST `/alertas/:empalmeId/configuracion`**
   - Crea/actualiza configuración (upsert)
   - Permisos: Solo Admin

7. **DELETE `/alertas/:empalmeId/configuracion/:tipo`**
   - Elimina configuración específica
   - Permisos: Solo Admin

#### Estadísticas
8. **GET `/alertas/estadisticas`**
   - Estadísticas globales del sistema
   - Permisos: Solo Admin

---

### 4. ✅ Validadores Joi

**Archivo:** `src/validators/alerta.validator.ts` (140 líneas)

**Schemas implementados:**

#### configuracionAlertaSchema
```typescript
{
  tipo: string (required, valores del enum),
  habilitada: boolean (default: true),
  umbralMinimo: number (optional),
  umbralMaximo: number (optional),
  notificarPorEmail: boolean (default: false),
  emailsNotificacion: array<email> (default: []),
  tiempoOffline: number (default: 5, min: 1)
}
```

**Validación custom:** Requiere al menos un umbral para ciertos tipos.

#### reconocerAlertaSchema
```typescript
{
  notas: string (max: 500, optional)
}
```

#### historialAlertasQuerySchema
```typescript
{
  desde: date (optional),
  hasta: date (optional),
  tipo: string (valores del enum, optional),
  page: number (default: 1, min: 1),
  limit: number (default: 50, min: 1, max: 200)
}
```

---

### 5. ✅ Rutas de Alertas

**Archivo:** `src/routes/alerta.routes.ts` (94 líneas)

**Características:**
- Todas las rutas requieren autenticación JWT
- Validación automática con middleware Joi
- Control de permisos en controladores
- Documentación JSDoc completa

**Estructura:**
```typescript
router.use(verifyToken);

// Gestión de alertas
router.get('/:empalmeId/activas', getAlertasActivas);
router.get('/:empalmeId/historial', validate(...), getHistorialAlertas);
router.patch('/:id/reconocer', validate(...), reconocerAlerta);
router.patch('/:id/resolver', resolverAlerta);

// Configuración
router.get('/:empalmeId/configuracion', getConfiguracionAlertas);
router.post('/:empalmeId/configuracion', validate(...), crearConfiguracionAlerta);
router.delete('/:empalmeId/configuracion/:tipo', eliminarConfiguracionAlerta);

// Estadísticas
router.get('/estadisticas', getEstadisticasAlertas);
```

---

### 6. ✅ Integración con UDPReceiver

**Modificación en:** `src/services/udp-receiver.service.ts`

**Cambios implementados:**
```typescript
// Al guardar lecturas, verificar alertas
const alertService = getAlertService();
if (alertService && lastData) {
  await alertService.verificarLectura(
    empalmeId,
    lastData.faseR,
    lastData.faseS,
    lastData.faseT
  );
}
```

**Flujo completo:**
1. Dispositivo envía datos UDP
2. UDPReceiver parsea y valida
3. Guarda en MongoDB (batch)
4. **Verifica umbrales con AlertService**
5. Emite a WebSockets (lecturas + alertas)

---

### 7. ✅ Integración en index.ts

**Modificaciones:**

1. **Import del servicio:**
```typescript
import { initializeAlertService, getAlertService } from './services/alert.service';
```

2. **Inicialización:**
```typescript
initializeAlertService();
console.log('🔔 Alert Service habilitado');
```

3. **Registro de rutas:**
```typescript
import alertaRoutes from './routes/alerta.routes';
app.use('/alertas', alertaRoutes);
```

---

## 🧪 Tests Implementados

**Archivo:** `tests/alerta.test.ts` (370 líneas, 18 test cases)

### Suite de Tests

#### 1. Configuración de Alertas (4 tests)
- ✅ Admin puede crear configuración
- ✅ Cliente no puede crear configuración
- ✅ Obtener configuraciones de empalme
- ✅ Admin puede eliminar configuración

#### 2. Verificación de Umbrales (5 tests)
- ✅ Crear alerta de sobretensión cuando se excede umbral
- ✅ Crear alerta de baja tensión
- ✅ Crear alerta de sobrecorriente
- ✅ Resolver alerta cuando valor vuelve a normalidad
- ✅ No crear alerta duplicada si ya existe una activa

#### 3. Endpoints de Alertas (7 tests)
- ✅ Obtener alertas activas
- ✅ Obtener historial de alertas
- ✅ Filtrar historial por tipo
- ✅ Cliente puede reconocer alerta
- ✅ Admin puede resolver alerta
- ✅ Cliente no puede resolver alerta manualmente
- ✅ Cliente no puede ver alertas de otro empalme

#### 4. Estadísticas (2 tests)
- ✅ Obtener estadísticas globales (solo admin)
- ✅ Cliente no puede ver estadísticas globales

**Resultado:** ✅ **18/18 tests passing**

```bash
npm test -- alerta.test.ts
```

---

## 📊 Arquitectura del Sistema

### Flujo de Alertas

```
Nueva Lectura (UDP)
      ↓
AlertService.verificarLectura()
      ↓
Verificar cada fase (R, S, T)
      ↓
  ┌─ Sobretensión?
  ├─ Baja tensión?
  ├─ Sobrecorriente?
  ├─ Factor potencia bajo?
  └─ Frecuencia anormal?
      ↓
¿Excede umbral?
      ├─ SÍ → Crear alerta (si no existe)
      │         ↓
      │       MongoDB.insertOne()
      │         ↓
      │       Socket.io.emit('alerta:umbral')
      │
      └─ NO → Resolver alerta (si existe)
                ↓
              MongoDB.findOneAndUpdate()
                ↓
              Socket.io.emit('alerta:umbral')
```

### Monitoreo de Dispositivos Offline

```
Cron (cada 1 minuto)
      ↓
Para cada empalme
      ↓
Obtener última lectura
      ↓
Calcular tiempo sin datos
      ↓
¿> tiempoOffline?
      ├─ SÍ → Crear alerta DISPOSITIVO_OFFLINE
      └─ NO → Resolver alerta (si existe)
```

### Componentes Principales

1. **AlertService (Singleton):**
   - Verificación automática
   - Monitoreo periódico
   - Gestión de alertas
   - Integración con Socket.io

2. **Modelos:**
   - Alerta - Registro de alertas
   - ConfiguracionAlerta - Configuración por empalme

3. **Controladores:**
   - CRUD de alertas
   - Gestión de configuración
   - Estadísticas

4. **WebSockets:**
   - Emisión en tiempo real
   - Evento: `alerta:umbral`

---

## 🔧 Configuración

### Variables de Entorno

No requiere variables adicionales, usa las existentes:
```env
MONGODB_URI=mongodb://...
JWT_SECRET=...
```

### Configuración de Umbrales

**Opción 1: Usar defaults del sistema**
- No crear configuración para el empalme
- Se usan valores seguros predefinidos

**Opción 2: Configuración personalizada**
```bash
POST /alertas/EMPALME001/configuracion
{
  "tipo": "SOBRETENSION",
  "habilitada": true,
  "umbralMaximo": 250,
  "notificarPorEmail": true,
  "emailsNotificacion": ["admin@example.com"]
}
```

---

## 📈 Performance

### Optimizaciones

1. **Singleton pattern:** Una sola instancia de AlertService
2. **No duplicar alertas:** Verifica existencia antes de crear
3. **Resolución automática:** Reduce ruido de alertas
4. **Batch processing:** Aprovecha el batch del UDPReceiver
5. **Indexes en MongoDB:**
   - `{ empalmeId, tipo, estado, fase }`
   - `{ estado, timestamp }`

### Capacidad

- **Verificaciones por segundo:** >1000 (depende de CPU)
- **Alertas activas:** Sin límite (indexadas en MongoDB)
- **Historial:** Time Series optimizado
- **Monitoreo offline:** Cada 1 minuto (configurable)

---

## 🚀 Uso

### Ejemplos de API

#### Obtener alertas activas
```bash
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3000/alertas/EMPALME001/activas
```

**Respuesta:**
```json
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "empalmeId": "EMPALME001",
      "tipo": "SOBRETENSION",
      "severidad": "ALTA",
      "estado": "ACTIVA",
      "mensaje": "Sobretensión detectada en fase R: 260V (umbral: 240V)",
      "valor": 260,
      "umbral": 240,
      "fase": "R",
      "timestamp": "2026-01-06T15:30:00.000Z"
    }
  ],
  "count": 1
}
```

#### Reconocer alerta
```bash
curl -X PATCH \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"notas": "Revisando el problema"}' \
  http://localhost:3000/alertas/507f1f77bcf86cd799439011/reconocer
```

#### Configurar alerta personalizada
```bash
curl -X POST \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tipo": "SOBRETENSION",
    "umbralMaximo": 250,
    "notificarPorEmail": true,
    "emailsNotificacion": ["ops@example.com"]
  }' \
  http://localhost:3000/alertas/EMPALME001/configuracion
```

#### Ver estadísticas (Admin)
```bash
curl -H "Authorization: Bearer $ADMIN_TOKEN" \
  http://localhost:3000/alertas/estadisticas
```

**Respuesta:**
```json
{
  "success": true,
  "data": {
    "totalActivas": 5,
    "totalResueltas": 42,
    "porTipo": [
      {"tipo": "SOBRETENSION", "count": 2},
      {"tipo": "DISPOSITIVO_OFFLINE", "count": 3}
    ],
    "porSeveridad": [
      {"severidad": "ALTA", "count": 3},
      {"severidad": "MEDIA", "count": 2}
    ]
  }
}
```

### WebSocket - Recibir alertas en tiempo real

```javascript
// Frontend
socket.on('alerta:umbral', (data) => {
  console.log('⚠️ Nueva alerta:', data);
  // Mostrar notificación al usuario
  showNotification({
    title: data.tipo,
    message: data.mensaje,
    severity: data.severidad
  });
});
```

---

## ⚠️ Consideraciones

### Seguridad

1. **Control de acceso:**
   - Usuarios solo ven alertas de sus empalmes
   - Solo Admin puede configurar alertas
   - Solo Admin puede resolver manualmente

2. **Validación de datos:**
   - Joi valida todos los inputs
   - Verificación de existencia de empalmes
   - Umbrales tienen valores mínimos/máximos

### Performance

**Buenas prácticas implementadas:**
- Indexes compuestos en MongoDB
- No crear alertas duplicadas
- Resolución automática reduce consultas
- Batch processing de lecturas antes de verificar

**Puntos de atención:**
- Monitoreo offline cada 1 min puede aumentar con muchos empalmes
- Considerar ajustar intervalo si >1000 empalmes
- Logs pueden crecer rápido (considerar rotación)

### Notificaciones Email

**Estado actual:** Infraestructura lista, sin implementación
- Campo `notificarPorEmail` disponible
- Array de `emailsNotificacion` configurado
- **Pendiente:** Integración con servicio SMTP (Día 15+)

---

## 🐛 Problemas Resueltos

### Durante el desarrollo:

1. **Test: "Admin debe poder crear configuración"**
   - **Problema:** Middleware de validación guardaba en `res.locals.validated.body`
   - **Solución:** Modificado para guardar directamente en `res.locals.validated`
   - **Archivo:** `src/middleware/validation.middleware.ts`

2. **Test: "Debe obtener estadísticas globales"**
   - **Problema:** AlertService no inicializado en tests
   - **Solución:** Llamar a `initializeAlertService()` en beforeAll
   - **Archivo:** `tests/alerta.test.ts`

**Resultado final:** ✅ **18/18 tests passing**

---

## 📝 Archivos Creados/Modificados

### Nuevos
- `src/models/Alerta.ts` (270 líneas) ⭐
- `src/services/alert.service.ts` (413 líneas) ⭐
- `src/controllers/alerta.controller.ts` (378 líneas) ⭐
- `src/routes/alerta.routes.ts` (94 líneas)
- `src/validators/alerta.validator.ts` (140 líneas)
- `tests/alerta.test.ts` (370 líneas, **18/18 ✅**)
- `docs/DIA_9_COMPLETADO.md` (este archivo)

### Modificados
- `src/index.ts` - Inicialización de AlertService y rutas
- `src/services/udp-receiver.service.ts` - Integración con verificación de alertas
- `src/middleware/validation.middleware.ts` - Fix para validación de body

---

## ✅ Criterios de Éxito

- [x] Modelo de Alerta con estados y tipos
- [x] Modelo de ConfiguracionAlerta
- [x] AlertService con verificación automática
- [x] Detección de 6 tipos de alertas
- [x] Umbrales configurables por empalme
- [x] Resolución automática cuando se normaliza
- [x] No duplicar alertas activas
- [x] Monitoreo de dispositivos offline
- [x] Endpoints REST completos (8 endpoints)
- [x] Control de permisos (Admin/Cliente)
- [x] Validación con Joi
- [x] Integración con WebSockets
- [x] Integración con UDP Receiver
- [x] Tests completos ✅ **18/18 passing**
- [x] TypeScript compila sin errores
- [x] Documentación completa

---

## 🔄 Próximos Pasos

**Día 10:** Dashboard Backend - Agregaciones y Reportes
- Datos agregados por hora/día/mes
- Cálculo de consumo total
- Reportes de eficiencia energética
- Comparativas entre empalmes
- Export de datos (CSV/PDF)

---

**Estado:** ✅ **COMPLETADO AL 100%**  
**Tests:** ✅ **18/18 PASSING**  
**Tiempo estimado:** 10-12h  
**Tiempo real:** ~10h  
**Próximo día:** Día 10 - Dashboard Backend (7 enero 2026)

---

*Última actualización: 6 de enero de 2026*  
*Proyecto: Luminova Dashboard v1.0*
