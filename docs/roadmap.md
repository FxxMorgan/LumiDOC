# 🗺️ ROADMAP SISTEMA LUMINOVA
## Dashboard de Monitoreo Energético Multi-tenant

**Duración:** 25 días hábiles (~5.5 semanas)  
**Fecha inicio:** 29 de diciembre 2025  
**Fecha término estimado:** 6 de febrero 2026  
**Horario:** 7:00 - 18:00/19:00 hrs (8-12 horas/día)  
**⚠️ Días NO disponibles:** 31 dic, 1 ene  

---

## 📋 RESUMEN EJECUTIVO

### Objetivo
Crear sistema web completo de monitoreo eléctrico trifásico con autenticación JWT, visualización en tiempo real, gestión de clientes/empalmes y persistencia en MongoDB Atlas.

### Alcance
- ✅ **Asumido:** Backend receptor de datos desde dispositivos (existente)
- 🔨 **Por construir:**
  - Sistema de autenticación y autorización (JWT)
  - Base de datos MongoDB Atlas con Time Series
  - API REST completa
  - Frontend web responsivo
  - Dashboard de visualización en tiempo real
  - Sistema de alertas básico
  - Panel administrativo

### Stack Tecnológico

**Backend:**
- Node.js 20+ con Express
- MongoDB Atlas (Time Series Collections)
- Mongoose ODM
- JWT (jsonwebtoken + bcrypt)
- Socket.io (tiempo real)
- Dotenv, Joi (validación)

**Frontend:**
- React 18 + TypeScript
- Vite (build tool)
- TailwindCSS + shadcn/ui
- Recharts (gráficas)
- React Query (cache)
- Socket.io-client
- React Router v6

**DevOps:**
- Git/GitHub
- Railway/Render (backend deploy)
- Vercel (frontend deploy)
- MongoDB Atlas (cloud database)

---

## 📅 CRONOGRAMA DETALLADO

### **SEMANA 1: Fundamentos (Días 1-5)**

#### **Día 1 - Lunes 29/12** - Setup Inicial
**Horas:** 8-10h  
**Tareas:**
- [ ] Crear repositorio Git con estructura monorepo
- [ ] Inicializar proyecto backend (Express + TypeScript)
- [ ] Inicializar proyecto frontend (Vite + React + TS)
- [ ] Configurar ESLint, Prettier, .gitignore
- [ ] Crear cuenta MongoDB Atlas y cluster
- [ ] Configurar variables de entorno (.env.example)
- [ ] Setup Docker Compose para desarrollo local

**Entregables:**
```
luminova/
├── backend/
│   ├── src/
│   ├── package.json
│   └── tsconfig.json
├── frontend/
│   ├── src/
│   ├── package.json
│   └── vite.config.ts
├── docker-compose.yml
└── README.md
```

---

#### **Día 2 - Martes 30/12** - Modelos de Datos
**Horas:** 8-10h  
**Tareas:**
- [ ] Diseñar esquemas Mongoose (Usuario, Empalme, Dispositivo)
- [ ] Implementar modelo Usuario (email, password, rol)
- [ ] Implementar modelo Empalme (clienteId, dispositivos)
- [ ] Implementar modelo Lectura (Time Series)
- [ ] Crear índices optimizados
- [ ] Escribir seeders para datos de prueba
- [ ] Tests unitarios de modelos

**Esquemas:**
```javascript
// Usuario: admin/cliente con empalmes asignados
// Empalme: datos del sitio + dispositivos
// Lectura: timestamp, empalmeId, fase, voltaje, corriente, etc.
```

---

#### **🚫 Miércoles 31/12 - DÍA NO DISPONIBLE**
_(Tareas redistribuidas a otros días)_

---

#### **🚫 Jueves 01/01 - DÍA NO DISPONIBLE**
_(Festivo - Año Nuevo)_

---

#### **Día 3 - Viernes 02/01** - Autenticación JWT
**Horas:** 10-12h  
**Tareas:**
- [ ] Implementar registro de usuarios (POST /auth/register)
- [ ] Implementar login con JWT (POST /auth/login)
- [ ] Middleware de autenticación (verifyToken)
- [ ] Middleware de autorización por roles
- [ ] Hash de contraseñas con bcrypt
- [ ] Refresh token opcional
- [ ] Endpoints de perfil de usuario
- [ ] Tests de autenticación

**Endpoints:**
```
POST   /api/auth/register
POST   /api/auth/login
GET    /api/auth/me
PUT    /api/auth/profile
POST   /api/auth/refresh (opcional)
```

---

#### **Día 4 - Lunes 05/01** - API REST Core
**Horas:** 10-12h  
**Tareas:**
- [ ] CRUD Empalmes (solo admin puede crear)
- [ ] Endpoint listar empalmes del usuario
- [ ] CRUD Dispositivos (asociados a empalme)
- [ ] Validación de datos con Joi
- [ ] Manejo de errores centralizado
- [ ] Logging con Winston/Pino
- [ ] Documentación con Swagger (opcional)

**Endpoints:**
```
GET    /api/empalmes              (usuario: sus empalmes, admin: todos)
POST   /api/empalmes              (solo admin)
GET    /api/empalmes/:id
PUT    /api/empalmes/:id          (solo admin)
DELETE /api/empalmes/:id          (solo admin)

GET    /api/empalmes/:id/dispositivos
POST   /api/empalmes/:id/dispositivos
```

---

#### **Día 5 - Martes 06/01** - Importador Legacy
**Horas:** 8-10h  
**Tareas:**
- [ ] Script para parsear archivos .txt legacy
- [ ] Bulk insert a MongoDB (lecturas históricas)
- [ ] Validación de formato de datos
- [ ] Manejo de duplicados
- [ ] Progress bar para importación
- [ ] Documentación del proceso

**Script:**
```bash
node scripts/import-legacy.js --empalme 6098972 --file "Ene 6098972 25-12-17.txt"
```

---

### **SEMANA 2: API de Lecturas (Días 6-10)**

#### **Día 6 - Miércoles 07/01** - Endpoints de Lecturas
**Horas:** 10-12h  
**Tareas:**
- [ ] GET lecturas con filtros (fecha, fase, empalme)
- [ ] Paginación y límites de resultados
- [ ] Endpoint de última lectura (tiempo real)
- [ ] Endpoint de estadísticas (promedio, máx, mín)
- [ ] Optimización de queries con agregaciones
- [ ] Tests de endpoints

**Endpoints:**
```
GET /api/lecturas?empalmeId=X&desde=YYYY-MM-DD&hasta=YYYY-MM-DD&fase=R
GET /api/lecturas/ultima/:empalmeId
GET /api/lecturas/stats/:empalmeId?periodo=24h
```

---

#### **Día 7 - Jueves 08/01** - WebSockets Tiempo Real
**Horas:** 10-12h  
**Tareas:**
- [ ] Integrar Socket.io en backend
- [ ] Rooms por empalme (aislamiento de datos)
- [ ] Autenticación de sockets con JWT
- [ ] Emitir nuevas lecturas a clientes conectados
- [ ] Manejo de reconexiones
- [ ] Tests de eventos Socket.io

**Eventos:**
```javascript
// Cliente → Servidor
socket.emit('subscribe:empalme', empalmeId)

// Servidor → Cliente
socket.on('lectura:nueva', (data) => {...})
socket.on('dispositivo:offline', (data) => {...})
```

---

#### **Día 8 - Viernes 09/01** - Receptor de Datos Dispositivos
**Horas:** 10-12h  
**Tareas:**
- [ ] Endpoint POST /api/data/ingest (recepción de lecturas)
- [ ] Validación de estructura de datos entrantes
- [ ] Rate limiting (prevenir spam)
- [ ] Guardar en MongoDB (batch inserts)
- [ ] Emitir a WebSockets conectados
- [ ] Logs de dispositivos conectados
- [ ] API key por dispositivo (opcional)

**Payload esperado:**
```json
{
  "dispositivoId": "dev_001",
  "empalmeId": "6098972",
  "timestamp": "2025-01-04T10:30:00Z",
  "faseR": { "voltaje": 220, "corriente": 8.5, ... },
  "faseS": { ... },
  "faseT": { ... },
  "señal_dbm": -62
}
```

---

#### **Día 9 - Lunes 12/01** - Sistema de Alertas
**Horas:** 10-12h  
**Tareas:**
- [ ] Modelo de Alerta (tipo, umbral, empalme)
- [ ] CRUD de configuración de alertas
- [ ] Verificador de umbrales (voltaje, corriente)
- [ ] Generación de notificaciones internas
- [ ] Endpoint para listar alertas activas
- [ ] Log de historial de alertas

**Tipos de alertas:**
- Sobretensión (>240V)
- Baja tensión (<200V)
- Sobrecorriente (>100A)
- Dispositivo offline (>5min sin datos)

---

#### **Día 10 - Martes 13/01** - Tests & Documentación Backend
**Horas:** 10-12h  
**Tareas:**
- [ ] Tests de integración de APIs
- [ ] Tests de flujo completo (login → datos)
- [ ] Documentación OpenAPI/Swagger
- [ ] README del backend
- [ ] Postman Collection
- [ ] Verificación de cobertura de tests (>70%)

---

### **SEMANA 3: Frontend Base (Días 11-15)**

#### **Día 11 - Miércoles 14/01** - Setup Frontend
**Horas:** 10-12h  
**Tareas:**
- [ ] Configurar React Router (rutas públicas/privadas)
- [ ] Configurar TailwindCSS + shadcn/ui
- [ ] Configurar React Query
- [ ] Configurar Axios interceptors (JWT)
- [ ] Context API para autenticación
- [ ] Layout principal (Header, Sidebar, Content)
- [ ] Componentes base (Button, Input, Card)

**Rutas:**
```
/login
/dashboard
/empalmes/:id
/empalmes/:id/dispositivos
/alertas
/admin/usuarios (solo admin)
```

---

#### **Día 12 - Jueves 15/01** - Autenticación Frontend
**Horas:** 10-12h  
**Tareas:**
- [ ] Página de Login
- [ ] Página de Registro (opcional)
- [ ] Almacenamiento seguro de JWT (httpOnly si es cookie)
- [ ] ProtectedRoute HOC
- [ ] Manejo de expiración de token
- [ ] Redirect después de login
- [ ] Logout y limpieza de sesión

---

#### **Día 13 - Viernes 16/01** - Dashboard Principal
**Horas:** 10-12h  
**Tareas:**
- [ ] Vista de lista de empalmes del usuario
- [ ] Cards con info resumida (estado, última lectura)
- [ ] Indicadores de estado (online/offline)
- [ ] Filtros y búsqueda
- [ ] Skeleton loaders
- [ ] Responsive design (mobile-first)

---

#### **Día 14 - Lunes 19/01** - Vista Detalle Empalme
**Horas:** 10-12h  
**Tareas:**
- [ ] Layout de detalle de empalme
- [ ] Tabs: Tiempo Real / Histórico / Dispositivos / Alertas
- [ ] Widgets de métricas actuales (Voltaje, Corriente, Potencia)
- [ ] Lista de dispositivos conectados
- [ ] Información del empalme (dirección, etc.)
- [ ] Navegación breadcrumb

---

#### **Día 15 - Martes 20/01** - Gráficas Básicas
**Horas:** 10-12h  
**Tareas:**
- [ ] Integrar Recharts
- [ ] Gráfica de Voltaje (3 fases) vs tiempo
- [ ] Gráfica de Corriente vs tiempo
- [ ] Gráfica de Potencia vs tiempo
- [ ] Selector de rango de fechas
- [ ] Selector de tipo de gráfica
- [ ] Tooltips personalizados
- [ ] Colores por fase (R=verde, S=naranja, T=azul)

---

### **SEMANA 4: Funcionalidades Avanzadas (Días 16-20)**

#### **Día 16 - Miércoles 21/01** - Tiempo Real con WebSockets
**Horas:** 10-12h  
**Tareas:**
- [ ] Integrar Socket.io-client
- [ ] Suscripción a empalme activo
- [ ] Actualización de widgets en tiempo real
- [ ] Actualización de gráficas (streaming data)
- [ ] Indicador de conexión (conectado/desconectado)
- [ ] Manejo de reconexiones automáticas

---

#### **Día 17 - Jueves 22/01** - Sistema de Alertas Frontend
**Horas:** 10-12h  
**Tareas:**
- [ ] Vista de lista de alertas
- [ ] Badge de notificaciones en header
- [ ] Modal de detalle de alerta
- [ ] Filtros por tipo/estado
- [ ] Marcar alerta como leída
- [ ] Sonido/toast al recibir nueva alerta
- [ ] Configuración de alertas por empalme

---

#### **Día 18 - Viernes 23/01** - Gráficas Avanzadas
**Horas:** 10-12h  
**Tareas:**
- [ ] Gráfica de Energía acumulada
- [ ] Gráfica de Factor de Potencia
- [ ] Gráfica de Frecuencia
- [ ] Comparación entre fases
- [ ] Zoom y pan en gráficas
- [ ] Export a PNG/CSV
- [ ] Selector de métricas múltiples

---

#### **Día 19 - Lunes 26/01** - Panel Administrativo
**Horas:** 10-12h  
**Tareas:**
- [ ] Vista de gestión de usuarios (solo admin)
- [ ] Crear/editar/eliminar usuarios
- [ ] Asignar empalmes a usuarios
- [ ] Vista de gestión de empalmes
- [ ] Crear/editar empalmes
- [ ] Crear/editar dispositivos
- [ ] Logs de actividad del sistema

---

#### **Día 20 - Martes 27/01** - Estadísticas y Reportes
**Horas:** 10-12h  
**Tareas:**
- [ ] Vista de estadísticas diarias/mensuales
- [ ] Consumo energético total
- [ ] Comparativas de períodos
- [ ] Gráficas de tendencias
- [ ] Top eventos (picos de consumo)
- [ ] Export de reportes PDF (opcional)

---

### **SEMANA 5: Testing, Deploy & Refinamiento (Días 21-25)**

#### **Día 21 - Miércoles 28/01** - Tests Frontend
**Horas:** 10-12h  
**Tareas:**
- [ ] Tests unitarios de componentes clave
- [ ] Tests de integración de flujos
- [ ] Tests E2E con Playwright/Cypress (opcional)
- [ ] Corrección de bugs encontrados
- [ ] Verificación de accesibilidad (a11y)

---

#### **Día 22 - Jueves 29/01** - Optimización & Performance
**Horas:** 10-12h  
**Tareas:**
- [ ] Code splitting y lazy loading
- [ ] Memoización de componentes pesados
- [ ] Optimización de queries MongoDB (índices)
- [ ] Compresión de respuestas (gzip)
- [ ] Lazy loading de gráficas
- [ ] Análisis con Lighthouse
- [ ] Caché de datos frecuentes

---

#### **Día 23 - Viernes 30/01** - Seguridad & Hardening
**Horas:** 10-12h  
**Tareas:**
- [ ] Audit de dependencias (npm audit)
- [ ] Rate limiting en APIs críticas
- [ ] Validación exhaustiva de inputs
- [ ] CORS configurado correctamente
- [ ] Headers de seguridad (Helmet.js)
- [ ] Sanitización de datos
- [ ] Pruebas de penetración básicas

---

#### **Día 24 - Lunes 02/02** - Deploy a Producción
**Horas:** 10-12h  
**Tareas:**
- [ ] Configurar CI/CD con GitHub Actions
- [ ] Deploy backend a Railway/Render
- [ ] Deploy frontend a Vercel
- [ ] Configurar variables de entorno producción
- [ ] Configurar dominio personalizado
- [ ] Configurar SSL/HTTPS
- [ ] Monitoreo básico (UptimeRobot)
- [ ] Logs centralizados (opcional)

---

#### **Día 25 - Martes 03/02** - Testing UAT & Documentación
**Horas:** 10-12h  
**Tareas:**
- [ ] Pruebas de aceptación con usuarios reales
- [ ] Corrección de últimos bugs
- [ ] Documentación de usuario (manual)
- [ ] Documentación técnica completa
- [ ] Video demo del sistema
- [ ] Handoff y capacitación
- [ ] Retrospectiva y mejoras futuras

---

## 🎯 HITOS PRINCIPALES

| Semana | Hito | Criterio de Éxito |
|--------|------|-------------------|
| 1 | Backend Core | API REST funcional con JWT + DB configurada |
| 2 | Integración Datos | Receptor de dispositivos + WebSockets activos |
| 3 | Frontend Base | Login + Dashboard + Gráficas básicas |
| 4 | Funcionalidades | Tiempo real + Alertas + Admin panel |
| 5 | Producción | Sistema deployado y funcional |

---

## 📊 RECURSOS NECESARIOS

### Humanos
- 1 Full-stack Developer (tú)
- 1 Tester/QA (opcional, días 21-25)
- 1 Designer UI/UX (opcional, consulta días 11-15)

### Herramientas
- Cuenta MongoDB Atlas (Free tier → M2 si crece)
- Hosting backend (Railway Free/Pro ~$5/mes)
- Hosting frontend (Vercel Free)
- GitHub (plan gratuito)
- Figma/Excalidraw (diseño/diagramas)

---

## ⚠️ RIESGOS Y MITIGACIONES

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Receptor de dispositivos incompatible | Media | Alto | Día 8: verificar formato datos, adaptar parser |
| Volumen de datos alto (performance) | Alta | Medio | Usar Time Series, índices, paginación |
| Autenticación compleja | Baja | Medio | Usar librerías probadas (Passport/JWT) |
| Bugs en tiempo real | Media | Medio | Tests exhaustivos días 7 y 21 |
| Deploy fallido | Baja | Alto | Ensayar deploy en día 20, tener rollback |
| Scope creep | Alta | Alto | **Decir NO a features nuevas hasta post-MVP** |

---

## 📦 ENTREGABLES FINALES

### Backend
- [x] API REST documentada (Swagger)
- [x] Sistema de autenticación JWT
- [x] Modelos MongoDB optimizados
- [x] WebSockets para tiempo real
- [x] Sistema de alertas
- [x] Tests (>70% cobertura)

### Frontend
- [x] Aplicación React responsiva
- [x] Dashboard multi-empalme
- [x] Gráficas interactivas
- [x] Tiempo real con WebSockets
- [x] Panel administrativo
- [x] Sistema de alertas visual

### Documentación
- [x] README técnico
- [x] Manual de usuario
- [x] Guía de deployment
- [x] Postman Collection
- [x] Diagramas de arquitectura

### Infraestructura
- [x] Backend deployado (Railway)
- [x] Frontend deployado (Vercel)
- [x] MongoDB Atlas configurado
- [x] CI/CD configurado
- [x] Monitoreo básico

---

## 🚀 POST-MVP (Backlog Futuro)

### Prioridad Alta
- App móvil (React Native)
- Notificaciones push/email
- Exportación avanzada (Excel, PDF)
- Multi-idioma (i18n)

### Prioridad Media
- Dashboard público embebible
- API GraphQL
- Machine Learning (predicción consumo)
- Integración con facturación

### Prioridad Baja
- Modo oscuro/claro
- Personalización de dashboard
- Comparación entre clientes
- Reportes automáticos programados

---

## 📞 CONTACTOS Y RECURSOS

**Documentación Oficial:**
- MongoDB Time Series: https://www.mongodb.com/docs/manual/core/timeseries-collections/
- Socket.io: https://socket.io/docs/v4/
- JWT Best Practices: https://datatracker.ietf.org/doc/html/rfc8725

**Inspiración/Referencias:**
- Grafana (gráficas)
- Supabase (autenticación)
- Vercel Dashboard (UI/UX)

---

## ✅ CHECKLIST DE INICIO

**Antes de comenzar (Domingo 28/12):**
- [ ] Confirmar acceso al sistema receptor de datos existente
- [ ] Obtener ejemplos de payloads de dispositivos
- [ ] Crear cuenta MongoDB Atlas
- [ ] Crear cuenta Railway/Render
- [ ] Crear cuenta Vercel
- [ ] Definir nombre de dominio
- [ ] Crear repositorio Git
- [ ] Instalar Node.js 20+, Docker, VSCode
- [ ] Configurar ambiente de desarrollo
- [ ] ☕ Preparar café para el lunes 7am

**Lunes 29/12 a las 7:00am:** ¡Comenzamos! 🚀

---

**NOTA IMPORTANTE:** Este roadmap asume trabajo intensivo (8-12h/día) según disponibilidad. Los días 31/12 y 01/01 están marcados como no disponibles y las tareas se redistribuyeron. Priorizar MVP sobre perfección; mejor algo funcional en 25 días que algo perfecto en 50.

**Consejo de productividad:** Con sesiones de 10-12h, tomar breaks cada 2-3 horas (técnica Pomodoro extendida). Máxima productividad: 7-12h (mañana), break 12-14h, retomar 14-19h.

---

*Última actualización: 26 de diciembre 2025*  
*Proyecto: Luminova Dashboard v1.0*  
*Inicio real: Lunes 29 de diciembre 2025*
