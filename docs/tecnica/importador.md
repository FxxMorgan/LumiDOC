# 📥 Importador de Datos Legacy

Sistema de importación de archivos `.txt` del sistema legacy de Luminova a MongoDB.

## 🎯 Características

- ✅ Parseo robusto de formato legacy UDP
- ✅ Validación de datos
- ✅ Bulk insert optimizado (1000 registros/lote)
- ✅ Progress bar en tiempo real
- ✅ Manejo de duplicados
- ✅ Extracción automática de timestamps
- ✅ Modo dry-run para pruebas
- ✅ CLI interactiva con comandos

## 📋 Formato de Datos Legacy

### Estructura de línea
```
userId valor1 valor2 ... valor18 señal_dbm
```

### Ejemplo real
```
6098974 214.50 3.36 0.56 250 49.80 0.77 214.80 0.00 0.00 90 49.80 1.00 214.80 0.12 0.00 10 49.80 0.03 -60
```

### Valores por fase (18 total)
**Fase R** (índices 0-5):
- Voltaje (V)
- Corriente (A)
- Potencia (kW)
- Energía (kWh)
- Frecuencia (Hz)
- Factor de Potencia

**Fase S** (índices 6-11):
- Voltaje (V)
- Corriente (A)
- Potencia (kW)
- Energía (kWh)
- Frecuencia (Hz)
- Factor de Potencia

**Fase T** (índices 12-17):
- Voltaje (V)
- Corriente (A)
- Potencia (kW)
- Energía (kWh)
- Frecuencia (Hz)
- Factor de Potencia

**Metadata**:
- Señal (dBm) - Opcional

## 🚀 Uso

### Prerrequisitos

```bash
# Navegar al directorio backend
cd luminova/backend

# Instalar dependencias
npm install

# Configurar MongoDB en .env
MONGODB_URI=mongodb+srv://...
```

### Comandos Disponibles

#### 1. Validar Archivo (sin importar)

```bash
npm run import validate -- -f /ruta/archivo.txt

# Opciones
npm run import validate -- \
  -f /ruta/archivo.txt \
  --lines 1000  # Número de líneas a validar
```

**Ejemplo de salida:**
```
🔍 Validando archivo...

📊 Resultado de validación:
   Total líneas: 100
   Válidas: 98 (98.0%)
   Inválidas: 2 (2.0%)

❌ Errores encontrados:
   Línea 45: Número incorrecto de valores: 15 (esperado: 20-21)
   Línea 78: Valor no numérico en posición 3: ABC

✅ Ejemplo de lectura parseada:
{
  "empalmeId": "6098972",
  "fases": {
    "R": {
      "voltaje": 214.5,
      "corriente": 3.36,
      ...
    }
  }
}
```

#### 2. Importar Archivo Individual

```bash
npm run import file -- \
  -f /ruta/LEC_AAA_001_001__25-12-11.txt \
  -e 6098972

# Con opciones completas
npm run import file -- \
  -f /home/feer/Documentos/Lumin/ejemplos_txt/LEC_AAA_001_001__25-12-11.txt \
  -e 6098972 \
  --date 2025-12-11 \
  --interval 2 \
  --batch 1000 \
  --skip-errors
```

**Parámetros:**
- `-f, --file <path>` - **Requerido** - Ruta al archivo .txt
- `-e, --empalme <id>` - **Requerido** - ID del empalme (ej: 6098972)
- `-d, --date <date>` - Fecha base (YYYY-MM-DD) - Se extrae del nombre si no se proporciona
- `-i, --interval <seconds>` - Intervalo entre lecturas (default: 2)
- `-b, --batch <size>` - Tamaño del lote para inserción (default: 1000)
- `--skip-errors` - Continuar si hay errores (default: true)
- `--dry-run` - Simular sin insertar en BD

**Ejemplo de salida:**
```
📂 Importando archivo legacy...

✅ Empalme encontrado: Edificio Principal

📅 Fecha extraída del nombre: 2025-12-11
📊 Parseando archivo...
   Procesadas: 22,040 líneas...
✅ Parseo completado en 1.23s

📈 Estadísticas de parseo:
   Total líneas: 22,040
   Exitosas: 22,040 (100.0%)
   Fallidas: 0 (0.0%)

⏰ Asignando timestamps...
✅ Timestamps asignados (intervalo: 2s)

💾 Insertando en MongoDB...
   Progreso: 22,040/22,040 (100.0%)
✅ Inserción completada en 8.45s

📊 Resultado de la importación:
   ✅ Insertadas: 22,040
   ⏭️  Duplicados: 0
   ❌ Errores: 0

⚡ Velocidad: 2,279 registros/segundo

✅ Proceso completado exitosamente
```

#### 3. Importar Directorio Completo

```bash
npm run import directory -- \
  -d /ruta/directorio \
  -e 6098972

# Con opciones
npm run import directory -- \
  -d /home/feer/Documentos/Lumin/ejemplos_txt \
  -e 6098972 \
  --pattern "*.txt" \
  --interval 2 \
  --batch 1000
```

**Parámetros:**
- `-d, --dir <path>` - **Requerido** - Ruta al directorio
- `-e, --empalme <id>` - **Requerido** - ID del empalme
- `-p, --pattern <glob>` - Patrón de archivos (default: "*.txt")
- `-i, --interval <seconds>` - Intervalo entre lecturas (default: 2)
- `-b, --batch <size>` - Tamaño del lote (default: 1000)
- `--skip-errors` - Continuar si hay errores
- `--dry-run` - Simular importación

**Ejemplo de salida:**
```
📁 Importando directorio...

✅ Empalme: Edificio Principal

📄 Archivos encontrados: 5

[1/5] LEC_AAA_001_001__25-12-11.txt
   Parseadas: 22,040
   ✅ Insertadas: 22,040

[2/5] LEC_AAA_001_001__25-12-12.txt
   Parseadas: 22,040
   ✅ Insertadas: 22,040

[3/5] LEC_AAA_001_001__25-12-13.txt
   Parseadas: 22,040
   ✅ Insertadas: 15,200
   ⏭️  Duplicados: 6,840

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 RESUMEN TOTAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Archivos procesados: 5
✅ Total insertadas: 110,200
⏭️  Total duplicados: 6,840
❌ Total errores: 0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 📁 Estructura de Archivos Legacy

### Nombres de archivo con fecha

El sistema extrae automáticamente la fecha del nombre del archivo:

```
LEC AAA 001 001  25-12-11.txt  →  2025-12-11
Ene 6098972 25-12-17.txt       →  2025-12-17
archivo-25-01-15.txt            →  2025-01-15
```

**Formato reconocido:** `YY-MM-DD`

### Estructura de directorio sugerida

```
datos-legacy/
├── empalme-6098972/
│   ├── LEC_AAA_001_001__25-12-11.txt
│   ├── LEC_AAA_001_001__25-12-12.txt
│   └── LEC_AAA_001_001__25-12-13.txt
├── empalme-6098974/
│   └── ...
└── empalme-6098980/
    └── ...
```

## ⚙️ Configuración Avanzada

### Tamaño de Lotes (Batch Size)

Controla cuántos registros se insertan en cada operación:

```bash
# Lote pequeño (más lento, menos memoria)
--batch 500

# Lote mediano (default, balanceado)
--batch 1000

# Lote grande (más rápido, más memoria)
--batch 5000
```

**Recomendaciones:**
- **500-1000**: Archivos pequeños o recursos limitados
- **1000-2000**: Uso general (default)
- **2000-5000**: Archivos grandes con buena conectividad

### Intervalo entre Lecturas

Define el tiempo entre cada registro:

```bash
# Cada 2 segundos (default - dispositivos normales)
--interval 2

# Cada 5 segundos (dispositivos menos frecuentes)
--interval 5

# Cada 10 segundos
--interval 10
```

### Modo Dry-Run

Simula la importación sin modificar la base de datos:

```bash
npm run import file -- \
  -f archivo.txt \
  -e 6098972 \
  --dry-run
```

**Útil para:**
- Validar datos antes de importar
- Estimar tiempo de importación
- Verificar formato sin afectar BD

## 🔧 Resolución de Problemas

### Error: "Empalme no encontrado"

```
❌ Error: Empalme 6098972 no encontrado en la base de datos
💡 Tip: Crea el empalme primero usando la API POST /empalmes
```

**Solución:**
1. Crear el empalme primero vía API o admin panel
2. Verificar que el ID sea correcto

### Error: "Número incorrecto de valores"

```
Línea 45: Número incorrecto de valores: 15 (esperado: 20-21)
```

**Causas comunes:**
- Línea corrupta en el archivo
- Formato diferente al esperado
- Valores faltantes

**Solución:**
- Usar `--skip-errors` para continuar
- Validar archivo con comando `validate`

### Error: "Valor no numérico"

```
Línea 78: Valor no numérico en posición 3: ABC
```

**Solución:**
- Revisar manualmente la línea en el archivo
- Limpiar datos antes de importar

### Rendimiento Lento

**Optimizaciones:**
- Aumentar `--batch` a 2000-5000
- Verificar conectividad a MongoDB
- Usar red local si es posible
- Cerrar otras aplicaciones

### Muchos Duplicados

```
⏭️  Duplicados: 15,000
```

**Causa:** Archivos ya importados previamente

**Solución:**
- Normal en re-importaciones
- Los duplicados se detectan por timestamp + empalmeId
- No afecta los datos existentes

## 📊 Rendimiento Esperado

**Hardware de referencia:**
- CPU: Intel i5 o superior
- RAM: 8GB
- Red: 10 Mbps

**Velocidad promedio:**
- Parseo: 15,000-20,000 líneas/segundo
- Inserción: 2,000-3,000 registros/segundo
- Total: ~1,500-2,500 registros/segundo

**Ejemplo real:**
```
Archivo con 22,040 líneas:
- Parseo: 1.2 segundos
- Inserción: 8.5 segundos
- Total: 9.7 segundos
- Velocidad: 2,279 registros/segundo
```

## 🎯 Casos de Uso

### Caso 1: Migración Inicial Completa

```bash
# 1. Validar primero
npm run import validate -- -f datos/sample.txt

# 2. Importar un archivo de prueba
npm run import file -- \
  -f datos/sample.txt \
  -e 6098972 \
  --dry-run

# 3. Importar todo el directorio
npm run import directory -- \
  -d datos/empalme-6098972 \
  -e 6098972
```

### Caso 2: Actualización Diaria

```bash
# Script automatizado (cron)
npm run import file -- \
  -f /datos/daily-$(date +%Y-%m-%d).txt \
  -e 6098972 \
  --skip-errors
```

### Caso 3: Recuperación de Datos

```bash
# Importar rango de fechas
for file in datos/empalme-6098972/25-12-*.txt; do
  npm run import file -- -f "$file" -e 6098972
done
```

## 📝 Notas Importantes

1. **Prerequisitos:**
   - El empalme debe existir en la BD antes de importar
   - MongoDB debe estar conectado y accesible
   - Suficiente espacio en disco (Time Series usa compresión)

2. **Duplicados:**
   - Se detectan por combinación de timestamp + empalmeId
   - Los duplicados se saltan automáticamente
   - No afecta los datos existentes

3. **Timestamps:**
   - Se asignan automáticamente basados en fecha del archivo
   - Incrementan en intervalos configurables (default: 2s)
   - Primera lectura = fecha base, siguientes incrementan

4. **Errores:**
   - Por default continúa con `--skip-errors`
   - Los errores se registran pero no detienen el proceso
   - Usar modo `validate` para revisar antes de importar

## 🔗 Ver También

- [Modelo Lectura](../src/models/Lectura.ts) - Schema de Time Series
- [Parser Legacy](../src/utils/legacy-parser.ts) - Funciones de parseo
- [API Lecturas](./API_LECTURAS.md) - Endpoints de consulta
