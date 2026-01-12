# 📅 DÍA 17 - SISTEMA DE ALERTAS FRONTEND

**Fecha:** 12 de enero 2026  
**Duración:** 10-12h  
**Estado:** ✅ COMPLETADO  

---

## 🎯 OBJETIVOS

Implementar el sistema completo de alertas en el frontend, permitiendo a los usuarios:
- Ver historial de alertas
- Recibir notificaciones en tiempo real
- Reconocer y resolver alertas
- Configurar umbrales personalizados por empalme

---

## ✅ TAREAS COMPLETADAS

### 1. Vista de Lista de Alertas (AlertasPage)
**Archivo:** `frontend/src/pages/AlertasPage.tsx`

**Características:**
- ✅ Grid responsivo con tarjetas de alerta
- ✅ Filtros por estado: Todas, Activa, Resuelta, Ignorada
- ✅ Indicadores de severidad con colores (crítica 🔴, alta 🟠, media 🟡, baja 🔵)
- ✅ Información resumida: timestamp, empalme, mensaje, estado
- ✅ Click en tarjeta abre modal de detalle
- ✅ Integración con endpoint `/alertas/:empalmeId/historial`
- ✅ Obtiene últimas 100 alertas por empalme
- ✅ Loading states y manejo de errores

**Endpoints utilizados:**
```typescript
GET /alertas/:empalmeId/historial?limit=100
```

**Filtrado:**
- Cliente-side sobre alertas obtenidas
- Estados: activa, reconocida, resuelta, ignorada

---

### 2. Badge de Notificaciones en Header
**Archivo:** `frontend/src/components/layout/Header.tsx`

**Características:**
- ✅ Badge rojo con contador de alertas activas
- ✅ React Query con `refetchInterval: 30000` (30s)
- ✅ Socket.io listener para evento `alerta:umbral`
- ✅ Invalidación automática de queries al recibir nueva alerta
- ✅ Click en badge navega a `/alertas`
- ✅ Ícono Bell de lucide-react

**Implementación:**
```typescript
// Query para contador
const { data: alertCount } = useQuery({
  queryKey: ['alertas-count'],
  queryFn: async () => {
    const promises = empalmesData.map(async (empalme) => {
      const response = await api.get(`/dev/alertas/${empalme.empalmeId}/activas`);
      return response.data.data.length;
    });
    const counts = await Promise.all(promises);
    return counts.reduce((sum, count) => sum + count, 0);
  },
  refetchInterval: 30000,
  enabled: !!empalmesData
});

// Socket.io listener
useEffect(() => {
  socket.on('alerta:umbral', () => {
    queryClient.invalidateQueries({ queryKey: ['alertas-count'] });
  });
}, []);
```

---

### 3. Modal de Detalle de Alerta
**Archivo:** `frontend/src/components/alerts/AlertDetailModal.tsx`

**Características:**
- ✅ Radix UI Dialog para modal
- ✅ Grid con severidad y estado
- ✅ Información completa: empalme, timestamp, mensaje, métrica, valores
- ✅ Timeline de eventos (creación, reconocimiento, resolución)
- ✅ Botones de acción según estado:
  - **Activa**: Reconocer y Resolver
  - **Reconocida**: Resolver
  - **Resuelta/Ignorada**: Solo lectura
- ✅ Mutaciones con React Query
- ✅ Toast notifications on success/error
- ✅ Invalidación de queries tras acciones

**Endpoints de acciones:**
```typescript
// Reconocer alerta
PATCH /alertas/:id/reconocer
Body: { notas?: string }

// Resolver alerta (solo admin)
PATCH /alertas/:id/resolver
```

**Validaciones:**
- Solo permite resolver a usuarios con rol `admin`
- Validación de `alerta !== null` antes de mutaciones
- Disable buttons durante pending state

---

### 4. Toast Notifications
**Archivo:** `frontend/src/components/ui/Toaster.tsx` + `App.tsx`

**Características:**
- ✅ Librería Sonner para toast notifications
- ✅ Toast al recibir nueva alerta via Socket.io
- ✅ Emojis según severidad:
  - Crítica: 🔴
  - Alta: 🟠
  - Media: 🟡
  - Baja: 🔵
- ✅ Duración: 8000ms (8s)
- ✅ Click en toast navega a `/alertas`
- ✅ Toast success/error en acciones de reconocer/resolver

**Implementación en App.tsx:**
```typescript
useEffect(() => {
  socket.on('alerta:umbral', (alerta: Alerta) => {
    const emoji = {
      critica: '🔴',
      alta: '🟠',
      media: '🟡',
      baja: '🔵',
    }[alerta.severidad] || '🔵';

    toast(`${emoji} Nueva Alerta: ${alerta.estado}: ${alerta.mensaje}`, {
      description: `Empalme: ${alerta.empalmeId}`,
      duration: 8000,
      action: {
        label: 'Ver',
        onClick: () => navigate('/alertas'),
      },
    });
  });
}, [navigate]);
```

---

### 5. Configuración de Alertas por Empalme
**Archivo:** `frontend/src/pages/ConfiguracionAlertasPage.tsx`

**Características:**
- ✅ Vista de configuraciones existentes
- ✅ Crear nueva configuración:
  - Seleccionar métrica (voltaje, corriente, potencia, etc.)
  - Seleccionar comparador (>, <, >=, <=, ==, !=)
  - Ingresar valor umbral
  - Seleccionar severidad
- ✅ Eliminar configuración existente
- ✅ CRUD completo con React Query
- ✅ Navegación desde EmpalmeDetallePage (botón Settings)
- ✅ Route: `/empalmes/:id/configuracion-alertas`

**Endpoints:**
```typescript
// Obtener configuraciones
GET /alertas/:empalmeId/configuracion

// Crear configuración
POST /alertas/:empalmeId/configuracion
Body: {
  tipo: TipoAlerta,
  metrica: string,
  comparador: string,
  umbral: number,
  severidad: SeveridadAlerta
}

// Eliminar configuración
DELETE /alertas/:empalmeId/configuracion/:tipo
```

**Tipos soportados:**
- Sobretensión
- Baja tensión
- Sobrecorriente
- Factor de potencia bajo
- Frecuencia anormal
- Dispositivo offline

---

## 🔧 COMPONENTES NUEVOS

### 1. `Toaster.tsx`
- Wrapper de Sonner
- Configuración global de toasts
- Instalado: `npm install sonner`

### 2. `dialog.tsx`
- Wrapper de Radix UI Dialog
- Componentes: Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogFooter
- Instalado: `npm install @radix-ui/react-dialog`

### 3. `AlertDetailModal.tsx`
- Modal completo de detalle de alerta
- Gestión de acciones (reconocer/resolver)
- 239 líneas

### 4. `ConfiguracionAlertasPage.tsx`
- Página completa de configuración CRUD
- 309 líneas

---

## 📝 CAMBIOS EN ARCHIVOS EXISTENTES

### `App.tsx`
- Importado `<Toaster />`
- Socket.io listener para alertas
- Toast notification al recibir alerta
- Route para ConfiguracionAlertasPage

### `Header.tsx`
- Badge de alertas con contador
- useQuery con refetchInterval
- Socket.io listener

### `EmpalmeDetallePage.tsx`
- Botón "Configurar Alertas" (Settings icon)
- Navigate a `/empalmes/:id/configuracion-alertas`

### `types/api.ts`
- Actualizado `Alerta` interface:
  ```typescript
  interface Alerta {
    _id: string;
    empalmeId: string;
    tipo: TipoAlerta;
    severidad: SeveridadAlerta;
    estado: EstadoAlerta;
    titulo: string;
    mensaje: string;
    metrica?: string;
    valorActual?: number;
    valorUmbral?: number;
    valorDetectado?: Record<string, any>; // NUEVO
    timestamp: string;
    timestampResolucion?: string;
    reconocidaEn?: string; // NUEVO
    resueltaEn?: string; // NUEVO
    createdAt: string;
    updatedAt: string;
  }
  ```
- Agregado `EstadoAlerta.RECONOCIDA = 'reconocida'`

---

## 🐛 BUGS CORREGIDOS

### 1. Error al reconocer/resolver alerta
**Problema:** `alerta._id` podía ser undefined si `alerta === null`

**Solución:**
```typescript
// Antes
onClick={() => reconocerMutation.mutate(alerta._id)}

// Después
onClick={() => alerta && reconocerMutation.mutate(alerta._id)}
```

### 2. Alertas históricas no se mostraban
**Problema:** Query usaba endpoint `/dev/alertas/:id/activas` que solo devuelve alertas activas

**Solución:** Cambiar a endpoint `/alertas/:id/historial?limit=100`

### 3. Límite excedido en query
**Problema:** Intentaba obtener 1000 alertas pero backend tiene límite máximo de 100

**Solución:** Ajustar a `limit: 100` según validador del backend

---

## 🔄 OPTIMIZACIONES

### 1. Frecuencia de alertas en Data Generator
**Antes:** 10% de probabilidad (1 de cada 10 lecturas)
**Después:** 1% de probabilidad (aprox 1 de cada 100 lecturas)

**Archivo:** `backend/src/services/data-generator.service.ts`
```typescript
// Antes
const generarAlerta = Math.random() < 0.1;

// Después
const generarAlerta = Math.random() < 0.01;
```

---

## 🎨 DISEÑO Y UX

### Colores de Severidad
- **Crítica:** `bg-red-100 text-red-800 border-red-300`
- **Alta:** `bg-orange-100 text-orange-800 border-orange-300`
- **Media:** `bg-yellow-100 text-yellow-800 border-yellow-300`
- **Baja:** `bg-blue-100 text-blue-800 border-blue-300`

### Iconos de Estado
- **Activa:** `AlertTriangle` (lucide-react)
- **Reconocida:** `Clock`
- **Resuelta:** `CheckCircle`
- **Ignorada:** `XCircle`

### Animaciones
- Toasts con animación slide-in de Sonner
- Modal con fade-in de Radix UI
- Transitions smooth en hover states

---

## 📊 MÉTRICAS

- **Archivos creados:** 4
- **Archivos modificados:** 5
- **Líneas de código:** ~900
- **Componentes nuevos:** 4
- **Endpoints integrados:** 6
- **Queries React Query:** 4
- **Mutaciones React Query:** 2
- **Socket.io listeners:** 2

---

## 🔐 PERMISOS Y SEGURIDAD

### Reconocer Alerta
- ✅ Admin: Puede reconocer cualquier alerta
- ✅ Cliente: Solo puede reconocer alertas de sus propios empalmes
- ✅ Backend valida permisos en `reconocerAlerta` controller

### Resolver Alerta
- ✅ Solo Admin puede resolver manualmente
- ✅ Frontend muestra botón solo si tiene permisos
- ✅ Backend retorna 403 si usuario no es admin

### Ver Configuración
- ✅ Admin: Puede ver/editar cualquier configuración
- ✅ Cliente: Solo puede ver configuraciones de sus empalmes

---

## 🚀 PRÓXIMOS PASOS (DÍA 18)

### Mejoras pendientes para sistema de alertas:
- [ ] Paginación para más de 100 alertas
- [ ] Filtro por tipo de alerta (umbral/anomalía/dispositivo)
- [ ] Búsqueda de texto en alertas
- [ ] Filtro por rango de fechas personalizado
- [ ] Export de alertas a CSV/PDF
- [ ] Configuración de notificaciones por email
- [ ] Sonido al recibir alerta crítica

### Día 18 - Gráficas Avanzadas:
- [ ] Gráfica de Energía acumulada
- [ ] Gráfica de Factor de Potencia
- [ ] Gráfica de Frecuencia
- [ ] Comparación entre fases
- [ ] Zoom y pan en gráficas
- [ ] Export a PNG/CSV
- [ ] Selector de métricas múltiples

---

## 📚 DEPENDENCIAS NUEVAS

```json
{
  "sonner": "^1.x",
  "@radix-ui/react-dialog": "^1.x"
}
```

---

## 🔗 REFERENCIAS

- [Sonner Documentation](https://sonner.emilkowal.ski/)
- [Radix UI Dialog](https://www.radix-ui.com/primitives/docs/components/dialog)
- [React Query Mutations](https://tanstack.com/query/latest/docs/framework/react/guides/mutations)
- [Socket.io Client](https://socket.io/docs/v4/client-api/)

---

**Día completado exitosamente** ✅  
**Próximo:** Día 18 - Gráficas Avanzadas
