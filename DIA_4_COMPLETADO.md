# ✅ DÍA 4 COMPLETADO - API REST Core (Gestión de Empalmes)
**Fecha:** 31 de diciembre de 2025  
**Duración:** ~6 horas  
**Estado:** ✅ COMPLETADO Y TESTEADO

---

## 📋 TAREAS COMPLETADAS

### 1. CRUD Completo de Empalmes
- ✅ GET /empalmes - Listar empalmes (filtrado por rol)
- ✅ GET /empalmes/:id - Obtener empalme específico
- ✅ POST /empalmes - Crear empalme (solo admin)
- ✅ PUT /empalmes/:id - Actualizar empalme (solo admin)
- ✅ DELETE /empalmes/:id - Eliminar empalme - soft delete (solo admin)

### 2. Gestión de Dispositivos
- ✅ GET /empalmes/:id/dispositivos - Listar dispositivos
- ✅ POST /empalmes/:id/dispositivos - Agregar dispositivo (solo admin)
- ✅ DELETE /empalmes/:id/dispositivos/:deviceId - Eliminar dispositivo (solo admin)

### 3. Validaciones con Joi
- ✅ Validación de datos de entrada
- ✅ Schemas para crear/actualizar empalmes
- ✅ Schemas para dispositivos
- ✅ Validación de parámetros de query

### 4. Middleware de Manejo de Errores
- ✅ Error handler centralizado
- ✅ Manejo de errores de Mongoose
- ✅ Manejo de errores de JWT
- ✅ Handler para rutas 404
- ✅ Diferentes respuestas para dev/prod

### 5. Control de Acceso por Roles
- ✅ Middleware `requireRoles()` para autorización
- ✅ Admin puede ver/modificar todos los empalmes
- ✅ Cliente solo puede ver sus empalmes asignados
- ✅ Validación de permisos en cada endpoint

---

## 📁 ARCHIVOS CREADOS

```
backend/src/
├── controllers/
│   └── empalme.controller.ts       # Lógica de negocio de empalmes (420 líneas)
├── routes/
│   └── empalme.routes.ts           # Rutas de empalmes con middleware
├── middleware/
│   ├── validation.middleware.ts    # Validación con Joi
│   └── error.middleware.ts         # Manejo centralizado de errores
├── validators/
│   └── empalme.validator.ts        # Schemas Joi para empalmes
└── tests/
    └── test-empalmes.js            # Tests automatizados (12 casos)
```

---

## 🔧 CORRECCIONES APLICADAS

### Problema 1: Middleware de validación con req.query
**Causa:** `req.query` es readonly en Express  
**Solución:**
```typescript
// Para query, no podemos reasignar directamente
if (source !== 'query') {
  req[source] = value;
}
```

### Problema 2: Tipo UserRole en requireRoles
**Causa:** Pasando string literal en lugar de enum  
**Solución:**
```typescript
// Antes
requireRoles('admin')

// Después
import { UserRole } from '../models/User';
requireRoles(UserRole.ADMIN)
```

### Problema 3: Campos del modelo Empalme diferentes
**Causa:** Validadores usaban campos diferentes al modelo  
**Solución:** Mapeo de campos al crear empalme:
```typescript
dispositivos: (dispositivos || []).map((d) => ({
  dispositivoId: d.deviceId,  // Campo correcto del modelo
  nombre: d.nombre,
  tipo: 'medidor_trifasico',
  activo: d.activo
}))
```

---

## 🧪 TESTS EJECUTADOS

**Script:** `tests/test-empalmes.js` (12 casos de prueba)

### Resultados:
```
✅ Test 1: Login como Admin - OK
✅ Test 2: Login como Cliente - OK
✅ Test 3: Crear empalme (Admin) - OK
✅ Test 4: Listar empalmes (Cliente) - OK - Ve solo 3 asignados
✅ Test 5: Listar empalmes (Admin) - OK - Ve todos (4)
✅ Test 6: Obtener empalme específico - OK
✅ Test 7: Agregar dispositivo - OK
✅ Test 8: Listar dispositivos - OK
✅ Test 9: Actualizar empalme - OK
✅ Test 10: Crear empalme como Cliente - OK - Acceso denegado (403)
✅ Test 11: Eliminar dispositivo - OK
✅ Test 12: Eliminar empalme (soft delete) - OK
```

---

## 🔑 ENDPOINTS DISPONIBLES

### GET /empalmes
**Descripción:** Listar empalmes (filtrado por rol)  
**Auth:** Bearer token requerido  
**Query params:**
```typescript
{
  activo?: boolean,
  page?: number,    // Default: 1
  limit?: number    // Default: 20, Max: 100
}
```
**Response 200:**
```json
{
  "success": true,
  "data": {
    "empalmes": [
      {
        "_id": "...",
        "empalmeId": "6098972",
        "nombre": "Edificio Principal",
        "clienteId": {
          "nombre": "Diego",
          "email": "diego@empresa.cl",
          "empresa": "Empresa Demo"
        },
        "dispositivos": [...],
        "activo": true,
        "createdAt": "..."
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 3,
      "pages": 1
    }
  }
}
```

**Lógica de filtrado por rol:**
- **Admin:** Ve todos los empalmes del sistema
- **Cliente:** Solo ve empalmes asignados a su cuenta

---

### GET /empalmes/:id
**Descripción:** Obtener empalme específico  
**Auth:** Bearer token requerido  
**Params:** `id` (ObjectId del empalme)  
**Response 200:**
```json
{
  "success": true,
  "data": {
    "_id": "...",
    "empalmeId": "6098972",
    "nombre": "Edificio Principal",
    "direccion": "Av. Principal 123",
    "clienteId": {
      "nombre": "Diego",
      "email": "diego@empresa.cl"
    },
    "dispositivos": [
      {
        "dispositivoId": "6098972",
        "nombre": "Dispositivo Principal",
        "tipo": "medidor_trifasico",
        "activo": true
      }
    ],
    "ubicacionGPS": {
      "lat": -33.4372,
      "lng": -70.6506
    },
    "activo": true
  }
}
```

**Validación de permisos:**
- Cliente solo puede ver sus propios empalmes (403 si intenta acceder a otros)
- Admin puede ver cualquier empalme

---

### POST /empalmes
**Descripción:** Crear nuevo empalme  
**Auth:** Bearer token requerido + **Solo Admin**  
**Body:**
```json
{
  "empalmeId": "9876543",
  "nombre": "Bodega Norte",
  "clienteId": "507f1f77bcf86cd799439011",
  "ubicacion": {
    "direccion": "Calle Test 456",
    "ciudad": "Santiago",
    "region": "Metropolitana",
    "coordenadas": {
      "lat": -33.4489,
      "lng": -70.6693
    }
  },
  "dispositivos": [
    {
      "deviceId": "9876543",
      "nombre": "Medidor Principal",
      "fase": "R",
      "activo": true
    }
  ],
  "configuracion": {
    "alertasActivas": true,
    "umbrales": {
      "voltajeMin": 200,
      "voltajeMax": 240,
      "corrienteMax": 30,
      "potenciaMax": 10
    }
  }
}
```

**Response 201:**
```json
{
  "success": true,
  "message": "Empalme creado exitosamente",
  "data": {
    "_id": "...",
    "empalmeId": "9876543",
    "nombre": "Bodega Norte",
    ...
  }
}
```

**Validaciones:**
- empalmeId debe ser único (6-10 dígitos)
- clienteId debe ser un ObjectId válido
- Cliente debe existir en la base de datos
- Al crear, se agrega automáticamente al array `empalmes` del usuario

---

### PUT /empalmes/:id
**Descripción:** Actualizar empalme existente  
**Auth:** Bearer token requerido + **Solo Admin**  
**Body (campos opcionales):**
```json
{
  "nombre": "Nuevo Nombre",
  "ubicacion": {
    "direccion": "Nueva Dirección"
  },
  "dispositivos": [...],
  "activo": false
}
```

**Response 200:**
```json
{
  "success": true,
  "message": "Empalme actualizado exitosamente",
  "data": { ... }
}
```

**Lógica especial:**
- Si se cambia `clienteId`, se actualiza automáticamente:
  - Se remueve del array `empalmes` del cliente anterior
  - Se agrega al array `empalmes` del nuevo cliente

---

### DELETE /empalmes/:id
**Descripción:** Eliminar empalme (soft delete)  
**Auth:** Bearer token requerido + **Solo Admin**  
**Response 200:**
```json
{
  "success": true,
  "message": "Empalme eliminado exitosamente"
}
```

**Comportamiento:**
- **No elimina físicamente** el registro
- Marca `activo: false`
- Remueve la referencia del array `empalmes` del usuario
- Preserva datos históricos de lecturas

---

### GET /empalmes/:id/dispositivos
**Descripción:** Listar dispositivos de un empalme  
**Auth:** Bearer token requerido  
**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "dispositivoId": "6098972",
      "nombre": "Dispositivo Principal",
      "tipo": "medidor_trifasico",
      "activo": true,
      "instaladoEn": "2025-01-15T10:00:00.000Z"
    }
  ]
}
```

---

### POST /empalmes/:id/dispositivos
**Descripción:** Agregar dispositivo a empalme  
**Auth:** Bearer token requerido + **Solo Admin**  
**Body:**
```json
{
  "deviceId": "9876544",
  "nombre": "Dispositivo Secundario",
  "fase": "S",
  "activo": true
}
```

**Response 201:**
```json
{
  "success": true,
  "message": "Dispositivo agregado exitosamente",
  "data": [ /* array actualizado */ ]
}
```

**Validaciones:**
- deviceId único dentro del empalme
- Nombre opcional (auto-generado si no se proporciona)

---

### DELETE /empalmes/:id/dispositivos/:deviceId
**Descripción:** Eliminar dispositivo de empalme  
**Auth:** Bearer token requerido + **Solo Admin**  
**Response 200:**
```json
{
  "success": true,
  "message": "Dispositivo eliminado exitosamente",
  "data": [ /* array actualizado */ ]
}
```

---

## 🔐 CONTROL DE ACCESO

### Middleware requireRoles()
```typescript
// Definición
export const requireRoles = (...roles: UserRole[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction): void => {
    if (!req.userRole || !roles.includes(req.userRole)) {
      res.status(403).json({
        success: false,
        message: `Acceso denegado. Se requiere uno de los siguientes roles: ${roles.join(', ')}`,
      });
      return;
    }
    next();
  };
};

// Uso en rutas
router.post('/', verifyToken, requireRoles(UserRole.ADMIN), crearEmpalme);
```

### Matriz de Permisos

| Endpoint | Admin | Cliente |
|----------|-------|---------|
| GET /empalmes | ✅ Todos | ✅ Solo asignados |
| GET /empalmes/:id | ✅ Cualquiera | ✅ Solo si asignado |
| POST /empalmes | ✅ | ❌ 403 |
| PUT /empalmes/:id | ✅ | ❌ 403 |
| DELETE /empalmes/:id | ✅ | ❌ 403 |
| GET /empalmes/:id/dispositivos | ✅ Cualquiera | ✅ Solo si asignado |
| POST /empalmes/:id/dispositivos | ✅ | ❌ 403 |
| DELETE /empalmes/:id/dispositivos/:deviceId | ✅ | ❌ 403 |

---

## ⚠️ MANEJO DE ERRORES

### Error Handler Centralizado
```typescript
export const errorHandler = (
  err: Error | AppError,
  _req: Request,
  res: Response,
  _next: NextFunction
): void => {
  let statusCode = 500;
  let message = 'Error interno del servidor';

  // Error personalizado
  if (err instanceof AppError) {
    statusCode = err.statusCode;
    message = err.message;
  }
  // Mongoose ValidationError
  else if (err instanceof mongoose.Error.ValidationError) {
    statusCode = 400;
    message = 'Error de validación';
  }
  // Mongoose CastError (ID inválido)
  else if (err instanceof mongoose.Error.CastError) {
    statusCode = 400;
    message = `ID inválido: ${err.value}`;
  }
  // Duplicado (unique constraint)
  else if ((err as any).code === 11000) {
    statusCode = 400;
    message = 'Ya existe un registro con ese valor';
  }
  
  res.status(statusCode).json({
    success: false,
    message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};
```

### Tipos de Errores Manejados
1. ✅ AppError - Errores personalizados de la aplicación
2. ✅ ValidationError - Errores de validación de Mongoose
3. ✅ CastError - IDs inválidos
4. ✅ 11000 - Violación de unique constraint
5. ✅ JsonWebTokenError - Token inválido
6. ✅ TokenExpiredError - Token expirado
7. ✅ 404 - Rutas no encontradas

---

## 📊 INTEGRACIÓN CON INDEX.TS

```typescript
import empalmeRoutes from './routes/empalme.routes';
import { errorHandler, notFoundHandler } from './middleware/error.middleware';

// Rutas
app.use('/auth', authRoutes);
app.use('/empalmes', empalmeRoutes);  // ← Nueva

// Manejo de errores (siempre al final)
app.use(notFoundHandler);   // ← 404
app.use(errorHandler);      // ← Errores centralizados
```

---

## 🚀 PRÓXIMOS PASOS (DÍA 5)

### Importador de Datos Legacy
- [ ] Script para parsear archivos .txt del sistema legacy
- [ ] Bulk insert optimizado a MongoDB
- [ ] Validación de formato de datos UDP
- [ ] Manejo de duplicados
- [ ] Progress bar para importaciones grandes
- [ ] Documentación del proceso

**Fecha:** Viernes 2 de enero 2026  
**Horas estimadas:** 8-10h

---

## 📝 NOTAS

- ✅ API REST completamente funcional con CRUD completo
- ✅ Control de acceso por roles implementado
- ✅ Validaciones robustas con Joi
- ✅ Manejo de errores centralizado y consistente
- ✅ Tests automatizados (12 casos de prueba)
- ✅ Soft delete para preservar integridad de datos
- ✅ Paginación implementada para escalabilidad
- 💡 Considerar agregar filtros avanzados (por ciudad, región, etc.)
- 💡 Implementar rate limiting para producción

---

**Estado del Proyecto:** 16% completado (4/25 días)  
**Días invertidos:** 4  
**Días disponibles:** 21  
**Líneas de código agregadas:** ~850 líneas
