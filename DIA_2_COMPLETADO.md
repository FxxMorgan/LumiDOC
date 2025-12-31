# ✅ DÍA 2 COMPLETADO - Modelos de Datos

**Fecha:** 30 de diciembre 2025  
**Tiempo invertido:** 8-10h  
**Estado:** ✅ COMPLETADO

---

## 🎯 Tareas Completadas

### 1. Configuración de Base de Datos ✅

**Archivo creado:** `src/config/database.ts`
- ✅ Función de conexión a MongoDB
- ✅ Logger con Winston
- ✅ Event listeners (error, disconnect)
- ✅ Cierre graceful con SIGINT

### 2. Modelo Usuario ✅

**Archivo:** `src/models/User.ts`

**Características:**
- ✅ Interface TypeScript completa
- ✅ Roles: ADMIN y CLIENT
- ✅ Hash automático de contraseñas (bcrypt)
- ✅ Método `comparePassword()` para autenticación
- ✅ Método estático `findByEmail()`
- ✅ Relación con empalmes (array de ObjectIds)
- ✅ Campos: email, password, nombre, apellido, rol, empalmes, activo
- ✅ Timestamps automáticos
- ✅ Password excluido de JSON por defecto

**Índices:**
- `email` (único)
- `rol + activo`
- `empalmes`

### 3. Modelo Empalme ✅

**Archivo:** `src/models/Empalme.ts`

**Características:**
- ✅ Interface TypeScript completa
- ✅ Estados: ACTIVO, INACTIVO, MANTENIMIENTO, ERROR
- ✅ Dispositivos embebidos (array de subdocumentos)
- ✅ Relación con Usuario (clienteId)
- ✅ Configuración de alertas (voltaje, corriente)
- ✅ Ubicación GPS opcional
- ✅ Método `findByEmpalmeId()`
- ✅ Método `findByClientId()`
- ✅ Método `actualizarUltimaLectura()`

**Campos principales:**
- `empalmeId` (único, ej: "6098972")
- `nombre`, `descripcion`, `direccion`
- `clienteId` (ref a User)
- `dispositivos[]` (dispositivoId, nombre, tipo, ubicacion, activo)
- `estado`, `configuracion`, `ubicacionGPS`

**Índices:**
- `empalmeId` (único)
- `clienteId`
- `clienteId + activo`
- `estado + activo`
- `dispositivos.dispositivoId`

### 4. Modelo Lectura (Time Series) ✅

**Archivo:** `src/models/Lectura.ts`

**Características:**
- ✅ **Time Series Collection** optimizada
- ✅ Datos de 3 fases (R, S, T)
- ✅ 6 valores por fase (voltaje, corriente, potencia, energía, frecuencia, factor de potencia)
- ✅ Metadata: señal dBm del dispositivo
- ✅ Método `obtenerUltima()`
- ✅ Método `obtenerRango()` con filtros
- ✅ Método `crearDesdeUDP()` para parsear datos UDP

**Configuración Time Series:**
```javascript
timeseries: {
  timeField: 'timestamp',
  metaField: 'empalmeId',
  granularity: 'seconds'
}
```

**Índices optimizados:**
- `empalmeId + timestamp` (descendente)
- `dispositivoId + timestamp`
- `timestamp`

### 5. Script de Seeding ✅

**Archivo:** `src/scripts/seed.ts`

**Datos creados:**

**Usuarios:**
1. Admin: `admin@luminova.cl` / `admin123`
2. Cliente Diego: `diego@empresa.cl` / `cliente123`
3. Cliente María: `maria@empresa.cl` / `cliente123`

**Empalmes:**
1. `6098972` - Edificio Principal (Diego)
2. `6098974` - Bodega Norte (Diego)  
3. `6098980` - Planta Producción (María)

**Lecturas:**
- 10 lecturas por empalme (últimos 20 segundos)
- Valores realistas basados en datos legacy
- Total: 20 lecturas de prueba

**Ejecutar:**
```bash
npm run seed
```

### 6. Exportación Centralizada ✅

**Archivo:** `src/models/index.ts`
- Exporta todos los modelos desde un punto
- Facilita imports: `import { User, Empalme, Lectura } from './models'`

---

## 📊 Estructura de Datos

### Usuario
```typescript
{
  email: string (único)
  password: string (hasheado)
  nombre: string
  apellido?: string
  rol: 'admin' | 'cliente'
  empalmes: ObjectId[]
  activo: boolean
  ultimoAcceso?: Date
  timestamps
}
```

### Empalme
```typescript
{
  empalmeId: string (único, ej: "6098972")
  nombre: string
  descripcion?: string
  direccion?: string
  clienteId: ObjectId -> User
  dispositivos: [{
    dispositivoId: string
    nombre: string
    tipo: string
    ubicacion?: string
    instaladoEn?: Date
    ultimaLectura?: Date
    activo: boolean
  }]
  estado: 'activo' | 'inactivo' | 'mantenimiento' | 'error'
  configuracion: {
    alertas: { voltajeMax, voltajeMin, corrienteMax }
    horarioMonitoreo: { inicio, fin }
  }
  ubicacionGPS?: { lat, lng }
  activo: boolean
  timestamps
}
```

### Lectura (Time Series)
```typescript
{
  timestamp: Date
  empalmeId: string
  dispositivoId: string
  faseR: {
    voltaje: number
    corriente: number
    potencia: number
    energia: number
    frecuencia: number
    factorPotencia: number
  }
  faseS: { ... }
  faseT: { ... }
  señal_dbm?: number
  createdAt: Date
}
```

---

## 🧪 Verificación

### 1. Conectar a MongoDB Atlas

Ver guía: [CONFIGURAR_MONGODB.md](CONFIGURAR_MONGODB.md)

1. Crear cluster en MongoDB Atlas
2. Configurar usuario y contraseña
3. Whitelist de IPs
4. Copiar connection string
5. Actualizar `backend/.env`:
```env
MONGODB_URI=mongodb+srv://usuario:password@cluster.mongodb.net/luminova?retryWrites=true&w=majority
```

### 2. Iniciar servidor

```bash
cd backend
npm run dev
```

Salida esperada:
```
✅ MongoDB conectado: cluster-shard-00-00.xxxxx.mongodb.net
📊 Base de datos: luminova
🚀 Servidor corriendo en puerto 3000
```

### 3. Ejecutar seeding

```bash
npm run seed
```

Salida esperada:
```
🌱 Iniciando seeding de base de datos...
✅ Conectado a MongoDB
🧹 Limpiando colecciones...
👤 Creando usuarios...
⚡ Creando empalmes...
📊 Creando lecturas de prueba...

✅ SEEDING COMPLETADO
Usuarios: 3
Empalmes: 3
Lecturas: 20
```

### 4. Verificar en MongoDB Atlas

1. Ir a Database → Browse Collections
2. Verificar colecciones:
   - `users` (3 documentos)
   - `empalmes` (3 documentos)
   - `lecturas` (20 documentos) - Time Series

---

## 📁 Archivos Creados

```
backend/src/
├── config/
│   └── database.ts          ✅ Conexión MongoDB
├── models/
│   ├── User.ts              ✅ Modelo Usuario
│   ├── Empalme.ts           ✅ Modelo Empalme
│   ├── Lectura.ts           ✅ Modelo Lectura (Time Series)
│   └── index.ts             ✅ Exportación centralizada
├── scripts/
│   └── seed.ts              ✅ Seeder de datos
└── index.ts                 ✅ Actualizado con DB
```

---

## 🎓 Conceptos Implementados

### Time Series Collections
- Optimizado para datos temporales
- Compresión automática
- Queries más rápidas para rangos de tiempo
- Ideal para lecturas cada 2 segundos

### Índices Optimizados
- Búsquedas rápidas por empalmeId
- Queries eficientes por cliente
- Soporte para queries de rango temporal

### Relaciones
- User → Empalme (1:N)
- Empalme → Dispositivos (embebidos)
- Lectura → Empalme (referencia por string, no ObjectId)

### Seguridad
- Passwords hasheados con bcrypt (salt 10)
- Password no incluido en JSON responses
- Validaciones de email y longitudes

---

## 📝 Métodos Útiles Implementados

### User
```typescript
await User.findByEmail('diego@empresa.cl')
await user.comparePassword('cliente123')
```

### Empalme
```typescript
await Empalme.findByEmpalmeId('6098972')
await Empalme.findByClientId(userId)
empalme.actualizarUltimaLectura('6098972')
```

### Lectura
```typescript
await Lectura.obtenerUltima('6098972')
await Lectura.obtenerRango('6098972', desde, hasta, 'R')
await Lectura.crearDesdeUDP('6098972', '6098972', datos, -60)
```

---

## ⚠️ Notas Importantes

### MongoDB Atlas Free Tier
- ✅ 512 MB de almacenamiento
- ✅ Suficiente para desarrollo y pruebas
- ✅ Respaldos automáticos
- ⚠️ Para producción considerar tier M2+ ($9/mes)

### Time Series Collection
- Primera vez que MongoDB lo usa en Mongoose
- Excelente para este caso de uso (datos cada 2s)
- Compresión automática ahorra espacio
- Queries optimizadas para agregaciones temporales

### Formato UDP → MongoDB
El método `crearDesdeUDP()` parsea el formato legacy:
```
6098974 214.50 3.36 0.56 250 49.80 0.77 214.80 0.00 0.00 90 49.80 1.00 214.80 0.12 0.00 10 49.80 0.03 -60
```
Y lo convierte a estructura normalizada con 3 fases.

---

## 🚀 Próximo: Día 3 - Autenticación JWT

**Viernes 02/01 - 10-12h**

Tareas:
- Implementar registro de usuarios
- Implementar login con JWT
- Middlewares de autenticación y autorización
- Endpoints de perfil
- Refresh tokens (opcional)
- Tests de autenticación

---

## 📊 Progreso del Roadmap

**Semana 1: Fundamentos**
- ✅ Día 1: Setup Inicial (COMPLETADO)
- ✅ Día 2: Modelos de Datos (COMPLETADO)
- 🚫 Día 3: No disponible (31 dic)
- 🚫 Día 4: No disponible (1 ene)
- ⏳ Día 5: Autenticación JWT (viernes 2 ene)

**Tiempo estimado:** 40% del proyecto completado

---

**Estado final:** ✅ DÍA 2 COMPLETADO  
**Próximo día laborable:** Viernes 02/01 a las 7:00am  
**Pendiente:** Configurar MongoDB Atlas antes del viernes
