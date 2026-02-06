# 🔌 Prototipo Dispositivo IoT - Luminova Energy Monitor

## Resumen del Proyecto

Dispositivo IoT basado en **ESP32** para monitoreo de energía trifásica en tiempo real, 100% compatible con la plataforma Luminova. Lee voltaje, corriente, potencia, energía, frecuencia y factor de potencia de 3 fases y envía datos via UDP al servidor cada 2 segundos.

---

## 📋 Índice

1. [Especificaciones Técnicas](#especificaciones-técnicas)
2. [Lista de Componentes](#lista-de-componentes)
3. [Diagrama de Conexiones](#diagrama-de-conexiones)
4. [Código Fuente ESP32](#código-fuente-esp32)
5. [Configuración WiFi](#configuración-wifi)
6. [Protocolo de Comunicación](#protocolo-de-comunicación)
7. [Calibración](#calibración)
8. [Carcasa 3D](#carcasa-3d)
9. [Guía de Montaje](#guía-de-montaje)
10. [Troubleshooting](#troubleshooting)

---

## 🔧 Especificaciones Técnicas

### Parámetros Medidos por Fase (R, S, T)

| Parámetro | Rango | Precisión | Unidad |
|-----------|-------|-----------|--------|
| Voltaje | 0-300V AC | ±0.5% | V |
| Corriente | 0-100A | ±1% | A |
| Potencia Activa | 0-30kW | ±1% | kW |
| Energía Acumulada | 0-999999 | ±1% | kWh |
| Frecuencia | 45-65Hz | ±0.1Hz | Hz |
| Factor de Potencia | 0-1.00 | ±0.01 | - |

### Características del Dispositivo

| Característica | Valor |
|----------------|-------|
| Microcontrolador | ESP32-WROOM-32 |
| Chip de Medición | PZEM-004T v3.0 (x3) |
| Protocolo | UDP sobre WiFi |
| Frecuencia de Envío | 2 segundos |
| Alimentación | 5V DC / 2A |
| Consumo Promedio | ~500mA |
| Temperatura Operación | -10°C a +60°C |
| Conectividad | WiFi 802.11 b/g/n |
| Dimensiones | 120x80x35 mm |

---

## 🛒 Lista de Componentes

### Componentes Principales

| Componente | Cantidad | Precio Aprox. | Link Referencia |
|------------|----------|---------------|-----------------|
| ESP32-WROOM-32 DevKit | 1 | $8 USD | AliExpress/Amazon |
| PZEM-004T v3.0 100A | 3 | $12 USD c/u | AliExpress |
| Transformador CT 100A/50mA | 3 | $5 USD c/u | AliExpress |
| Fuente switching 5V 2A | 1 | $3 USD | AliExpress |
| Borneras 2 pines | 10 | $2 USD | Local |
| Caja plástica IP65 | 1 | $5 USD | Local |
| Cable AWG 18 (2m) | 1 | $2 USD | Local |
| PCB perforada 7x9cm | 1 | $1 USD | Local |
| Conectores JST 4 pines | 3 | $1 USD | AliExpress |

### Total Estimado: **~$75 USD**

### Herramientas Necesarias

- Soldador y estaño
- Multímetro
- Destornilladores
- Pelacables
- Pistola de silicona

---

## 📐 Diagrama de Conexiones

### Pinout ESP32 ↔ PZEM-004T

```
                    ┌─────────────────────────────────────┐
                    │           ESP32-WROOM-32            │
                    │                                     │
                    │  3V3 ─────┐                         │
                    │  GND ─────┼───────────────────────┐ │
                    │           │                       │ │
                    │  GPIO16 ──┼── RX PZEM Fase R     │ │
                    │  GPIO17 ──┼── TX PZEM Fase R     │ │
                    │           │                       │ │
                    │  GPIO25 ──┼── RX PZEM Fase S     │ │
                    │  GPIO26 ──┼── TX PZEM Fase S     │ │
                    │           │                       │ │
                    │  GPIO32 ──┼── RX PZEM Fase T     │ │
                    │  GPIO33 ──┼── TX PZEM Fase T     │ │
                    │           │                       │ │
                    │  VIN ─────┼── 5V Fuente          │ │
                    │  GND ─────┼── GND Fuente ────────┘ │
                    │                                     │
                    └─────────────────────────────────────┘

PZEM-004T v3.0 (x3 unidades):
┌──────────────────┐
│                  │
│  VCC ─── 5V      │  ← Alimentación
│  GND ─── GND     │
│  TX  ─── GPIO RX │  ← Datos al ESP32
│  RX  ─── GPIO TX │  ← Comandos del ESP32
│                  │
│  L ──── Línea AC │  ← Fase correspondiente (R/S/T)
│  N ──── Neutro   │
│  CT ─── Sensor   │  ← Transformador de corriente
│                  │
└──────────────────┘
```

### Conexión Eléctrica Trifásica

```
                    TABLERO ELÉCTRICO
        ┌─────────────────────────────────────────┐
        │                                         │
        │   L1 (R) ────────┬──────────────────────┼────► Carga
        │                  │                      │
        │              ┌───┴───┐                  │
        │              │  CT   │ ← Sensor Fase R  │
        │              │ 100A  │                  │
        │              └───┬───┘                  │
        │                  │                      │
        │   L2 (S) ────────┼──────────────────────┼────► Carga
        │                  │                      │
        │              ┌───┴───┐                  │
        │              │  CT   │ ← Sensor Fase S  │
        │              │ 100A  │                  │
        │              └───┬───┘                  │
        │                  │                      │
        │   L3 (T) ────────┼──────────────────────┼────► Carga
        │                  │                      │
        │              ┌───┴───┐                  │
        │              │  CT   │ ← Sensor Fase T  │
        │              │ 100A  │                  │
        │              └───┬───┘                  │
        │                  │                      │
        │   N (Neutro) ────┼──────────────────────┼────► Neutro
        │                  │                      │
        │              ┌───┴───┐                  │
        │              │FUENTE │                  │
        │              │ 5V DC │                  │
        │              └───┬───┘                  │
        │                  │                      │
        │              ┌───┴───┐                  │
        │              │ ESP32 │ ← Dispositivo    │
        │              │ + PCB │   Luminova       │
        │              └───────┘                  │
        │                                         │
        └─────────────────────────────────────────┘
```

---

## 💻 Código Fuente ESP32

### Archivo Principal: `luminova_iot.ino`

```cpp
/**
 * ╔══════════════════════════════════════════════════════════════════╗
 * ║            LUMINOVA IoT - Monitor Trifásico ESP32                ║
 * ╠══════════════════════════════════════════════════════════════════╣
 * ║  Versión: 1.0.0                                                  ║
 * ║  Autor: Luminova Team                                            ║
 * ║  Fecha: Enero 2026                                               ║
 * ║  Hardware: ESP32 + PZEM-004T v3.0 x3                             ║
 * ║  Protocolo: UDP → Puerto 5000                                    ║
 * ╚══════════════════════════════════════════════════════════════════╝
 * 
 * Formato UDP enviado (20 valores separados por espacio):
 * [empalmeId] [V_R] [I_R] [P_R] [E_R] [F_R] [FP_R] [V_S] [I_S] [P_S] [E_S] [F_S] [FP_S] [V_T] [I_T] [P_T] [E_T] [F_T] [FP_T] [señal]
 * 
 * Ejemplo real:
 * 6098974 214.50 3.36 0.56 250 49.80 0.77 214.80 0.00 0.00 90 49.80 1.00 214.80 0.12 0.00 10 49.80 0.03 -60
 */

#include <WiFi.h>
#include <WiFiUdp.h>
#include <HardwareSerial.h>
#include <PZEM004Tv30.h>
#include <EEPROM.h>
#include <ArduinoJson.h>

// ═══════════════════════════════════════════════════════════════════
// CONFIGURACIÓN - MODIFICAR SEGÚN TU INSTALACIÓN
// ═══════════════════════════════════════════════════════════════════

// WiFi
const char* WIFI_SSID = "TU_RED_WIFI";
const char* WIFI_PASSWORD = "TU_PASSWORD_WIFI";

// Servidor Luminova (cambiar IP según tu instalación)
const char* SERVER_IP = "192.168.1.100";  // IP del servidor backend
const uint16_t SERVER_PORT = 5000;         // Puerto UDP del backend

// Identificador único del empalme (debe coincidir con MongoDB)
const char* EMPALME_ID = "6098974";

// Intervalo de envío en milisegundos (2 segundos = 2000ms)
const uint32_t SEND_INTERVAL = 2000;

// ═══════════════════════════════════════════════════════════════════
// PINES - CONEXIONES HARDWARE
// ═══════════════════════════════════════════════════════════════════

// PZEM Fase R (Roja)
#define PZEM_R_RX 16
#define PZEM_R_TX 17

// PZEM Fase S (Amarilla)
#define PZEM_S_RX 25
#define PZEM_S_TX 26

// PZEM Fase T (Azul)
#define PZEM_T_RX 32
#define PZEM_T_TX 33

// LED de estado (integrado en ESP32)
#define LED_BUILTIN 2

// ═══════════════════════════════════════════════════════════════════
// OBJETOS GLOBALES
// ═══════════════════════════════════════════════════════════════════

// Instancias de Serial para cada PZEM
HardwareSerial SerialPZEM_R(1);  // Serial1
HardwareSerial SerialPZEM_S(2);  // Serial2

// Para el tercero usamos SoftwareSerial emulado
#include <SoftwareSerial.h>
SoftwareSerial SerialPZEM_T(PZEM_T_RX, PZEM_T_TX);

// Instancias de PZEM
PZEM004Tv30 pzemR(&SerialPZEM_R, PZEM_R_RX, PZEM_R_TX);
PZEM004Tv30 pzemS(&SerialPZEM_S, PZEM_S_RX, PZEM_S_TX);
PZEM004Tv30 pzemT(&SerialPZEM_T);

// Cliente UDP
WiFiUDP udp;

// Variables de estado
uint32_t lastSendTime = 0;
uint32_t messageCount = 0;
bool wifiConnected = false;

// ═══════════════════════════════════════════════════════════════════
// ESTRUCTURA DE DATOS POR FASE
// ═══════════════════════════════════════════════════════════════════

struct FaseData {
  float voltaje;
  float corriente;
  float potencia;
  float energia;
  float frecuencia;
  float factorPotencia;
  bool valid;
};

struct LecturaCompleta {
  FaseData faseR;
  FaseData faseS;
  FaseData faseT;
  int8_t rssi;  // Señal WiFi
};

// ═══════════════════════════════════════════════════════════════════
// SETUP INICIAL
// ═══════════════════════════════════════════════════════════════════

void setup() {
  // Inicializar Serial para debug
  Serial.begin(115200);
  delay(1000);
  
  Serial.println();
  Serial.println("╔══════════════════════════════════════════════════════════════╗");
  Serial.println("║            LUMINOVA IoT - Monitor Trifásico                  ║");
  Serial.println("╠══════════════════════════════════════════════════════════════╣");
  Serial.printf("║  Empalme ID: %-48s║\n", EMPALME_ID);
  Serial.printf("║  Servidor: %s:%-37d║\n", SERVER_IP, SERVER_PORT);
  Serial.println("╚══════════════════════════════════════════════════════════════╝");
  Serial.println();
  
  // Configurar LED
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, LOW);
  
  // Inicializar seriales PZEM
  SerialPZEM_R.begin(9600, SERIAL_8N1, PZEM_R_RX, PZEM_R_TX);
  SerialPZEM_S.begin(9600, SERIAL_8N1, PZEM_S_RX, PZEM_S_TX);
  SerialPZEM_T.begin(9600);
  
  delay(100);
  
  // Conectar WiFi
  connectWiFi();
  
  // Inicializar UDP
  udp.begin(0);  // Puerto local aleatorio
  
  Serial.println("✅ Sistema inicializado correctamente");
  Serial.println("📡 Iniciando envío de datos cada 2 segundos...");
  Serial.println();
}

// ═══════════════════════════════════════════════════════════════════
// LOOP PRINCIPAL
// ═══════════════════════════════════════════════════════════════════

void loop() {
  // Verificar conexión WiFi
  if (WiFi.status() != WL_CONNECTED) {
    wifiConnected = false;
    digitalWrite(LED_BUILTIN, LOW);
    connectWiFi();
  }
  
  // Enviar datos cada SEND_INTERVAL ms
  if (millis() - lastSendTime >= SEND_INTERVAL) {
    lastSendTime = millis();
    
    // Leer todas las fases
    LecturaCompleta lectura = leerTodasLasFases();
    
    // Enviar por UDP
    enviarUDP(lectura);
    
    // Parpadear LED
    blinkLED();
    
    // Incrementar contador
    messageCount++;
    
    // Log cada 30 segundos (15 mensajes)
    if (messageCount % 15 == 0) {
      imprimirEstadisticas(lectura);
    }
  }
  
  // Pequeño delay para evitar watchdog
  delay(10);
}

// ═══════════════════════════════════════════════════════════════════
// FUNCIONES DE CONEXIÓN WIFI
// ═══════════════════════════════════════════════════════════════════

void connectWiFi() {
  Serial.printf("📶 Conectando a WiFi: %s", WIFI_SSID);
  
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 30) {
    delay(500);
    Serial.print(".");
    attempts++;
    
    // Parpadeo rápido mientras conecta
    digitalWrite(LED_BUILTIN, !digitalRead(LED_BUILTIN));
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    wifiConnected = true;
    Serial.println();
    Serial.println("✅ WiFi conectado!");
    Serial.printf("   IP Local: %s\n", WiFi.localIP().toString().c_str());
    Serial.printf("   Señal: %d dBm\n", WiFi.RSSI());
    Serial.printf("   Gateway: %s\n", WiFi.gatewayIP().toString().c_str());
    Serial.println();
    
    // LED encendido = conectado
    digitalWrite(LED_BUILTIN, HIGH);
  } else {
    Serial.println();
    Serial.println("❌ Error al conectar WiFi. Reintentando en 5 segundos...");
    delay(5000);
  }
}

// ═══════════════════════════════════════════════════════════════════
// FUNCIONES DE LECTURA DE SENSORES
// ═══════════════════════════════════════════════════════════════════

/**
 * Lee datos de una fase específica
 */
FaseData leerFase(PZEM004Tv30& pzem, const char* nombreFase) {
  FaseData data;
  
  data.voltaje = pzem.voltage();
  data.corriente = pzem.current();
  data.potencia = pzem.power() / 1000.0;  // Convertir W a kW
  data.energia = pzem.energy();
  data.frecuencia = pzem.frequency();
  data.factorPotencia = pzem.pf();
  
  // Verificar si los datos son válidos
  data.valid = !isnan(data.voltaje) && 
               !isnan(data.corriente) && 
               !isnan(data.potencia);
  
  // Si no hay carga, establecer valores por defecto
  if (!data.valid || data.voltaje < 10) {
    // Sin conexión o sin energía
    data.voltaje = 0.0;
    data.corriente = 0.0;
    data.potencia = 0.0;
    data.energia = 0.0;
    data.frecuencia = 0.0;
    data.factorPotencia = 1.0;  // Factor perfecto cuando no hay carga
    data.valid = true;  // Marcamos como válido con valores cero
  }
  
  return data;
}

/**
 * Lee todas las fases
 */
LecturaCompleta leerTodasLasFases() {
  LecturaCompleta lectura;
  
  // Leer cada fase con un pequeño delay entre lecturas
  lectura.faseR = leerFase(pzemR, "R");
  delay(50);
  
  lectura.faseS = leerFase(pzemS, "S");
  delay(50);
  
  lectura.faseT = leerFase(pzemT, "T");
  
  // Leer señal WiFi
  lectura.rssi = WiFi.RSSI();
  
  return lectura;
}

// ═══════════════════════════════════════════════════════════════════
// FUNCIONES DE COMUNICACIÓN UDP
// ═══════════════════════════════════════════════════════════════════

/**
 * Envía los datos por UDP al servidor Luminova
 * 
 * Formato: [empalmeId] [V_R] [I_R] [P_R] [E_R] [F_R] [FP_R] [V_S] ... [señal]
 * Total: 20 valores separados por espacio
 */
void enviarUDP(LecturaCompleta& lectura) {
  // Construir mensaje UDP
  char buffer[256];
  
  snprintf(buffer, sizeof(buffer),
    "%s %.2f %.2f %.2f %.0f %.2f %.2f %.2f %.2f %.2f %.0f %.2f %.2f %.2f %.2f %.2f %.0f %.2f %.2f %d",
    EMPALME_ID,
    // Fase R
    lectura.faseR.voltaje,
    lectura.faseR.corriente,
    lectura.faseR.potencia,
    lectura.faseR.energia,
    lectura.faseR.frecuencia,
    lectura.faseR.factorPotencia,
    // Fase S
    lectura.faseS.voltaje,
    lectura.faseS.corriente,
    lectura.faseS.potencia,
    lectura.faseS.energia,
    lectura.faseS.frecuencia,
    lectura.faseS.factorPotencia,
    // Fase T
    lectura.faseT.voltaje,
    lectura.faseT.corriente,
    lectura.faseT.potencia,
    lectura.faseT.energia,
    lectura.faseT.frecuencia,
    lectura.faseT.factorPotencia,
    // Señal WiFi
    lectura.rssi
  );
  
  // Enviar por UDP
  udp.beginPacket(SERVER_IP, SERVER_PORT);
  udp.print(buffer);
  udp.endPacket();
  
  // Log del mensaje (solo en debug)
  #ifdef DEBUG
  Serial.printf("📤 UDP: %s\n", buffer);
  #endif
}

// ═══════════════════════════════════════════════════════════════════
// FUNCIONES AUXILIARES
// ═══════════════════════════════════════════════════════════════════

/**
 * Parpadea el LED para indicar actividad
 */
void blinkLED() {
  digitalWrite(LED_BUILTIN, LOW);
  delay(50);
  digitalWrite(LED_BUILTIN, HIGH);
}

/**
 * Imprime estadísticas en el monitor serial
 */
void imprimirEstadisticas(LecturaCompleta& lectura) {
  Serial.println("┌──────────────────────────────────────────────────────────────┐");
  Serial.println("│                    ESTADO DEL SISTEMA                        │");
  Serial.println("├──────────────────────────────────────────────────────────────┤");
  Serial.printf("│ Mensajes enviados: %-43lu│\n", messageCount);
  Serial.printf("│ Uptime: %lu segundos%41s│\n", millis() / 1000, "");
  Serial.printf("│ WiFi RSSI: %d dBm%44s│\n", lectura.rssi, "");
  Serial.println("├──────────────────────────────────────────────────────────────┤");
  Serial.println("│    FASE    │   VOLTAJE  │  CORRIENTE │  POTENCIA  │    FP    │");
  Serial.println("├────────────┼────────────┼────────────┼────────────┼──────────┤");
  Serial.printf("│  Fase R    │  %6.1f V  │  %6.2f A  │  %6.2f kW │   %.2f   │\n",
    lectura.faseR.voltaje, lectura.faseR.corriente, 
    lectura.faseR.potencia, lectura.faseR.factorPotencia);
  Serial.printf("│  Fase S    │  %6.1f V  │  %6.2f A  │  %6.2f kW │   %.2f   │\n",
    lectura.faseS.voltaje, lectura.faseS.corriente, 
    lectura.faseS.potencia, lectura.faseS.factorPotencia);
  Serial.printf("│  Fase T    │  %6.1f V  │  %6.2f A  │  %6.2f kW │   %.2f   │\n",
    lectura.faseT.voltaje, lectura.faseT.corriente, 
    lectura.faseT.potencia, lectura.faseT.factorPotencia);
  Serial.println("└──────────────────────────────────────────────────────────────┘");
  Serial.println();
}

// ═══════════════════════════════════════════════════════════════════
// FUNCIONES DE CONFIGURACIÓN (EEPROM)
// ═══════════════════════════════════════════════════════════════════

/**
 * Estructura de configuración guardada en EEPROM
 */
struct Config {
  char ssid[32];
  char password[64];
  char serverIP[16];
  uint16_t serverPort;
  char empalmeId[16];
  uint32_t sendInterval;
  uint8_t checksum;
};

/**
 * Guarda configuración en EEPROM
 */
void guardarConfig(Config& config) {
  config.checksum = calcularChecksum((uint8_t*)&config, sizeof(config) - 1);
  EEPROM.begin(sizeof(Config));
  EEPROM.put(0, config);
  EEPROM.commit();
  EEPROM.end();
}

/**
 * Carga configuración desde EEPROM
 */
bool cargarConfig(Config& config) {
  EEPROM.begin(sizeof(Config));
  EEPROM.get(0, config);
  EEPROM.end();
  
  uint8_t checksum = calcularChecksum((uint8_t*)&config, sizeof(config) - 1);
  return checksum == config.checksum;
}

/**
 * Calcula checksum simple
 */
uint8_t calcularChecksum(uint8_t* data, size_t len) {
  uint8_t sum = 0;
  for (size_t i = 0; i < len; i++) {
    sum ^= data[i];
  }
  return sum;
}

// ═══════════════════════════════════════════════════════════════════
// RESET DEL CONTADOR DE ENERGÍA (PZEM)
// ═══════════════════════════════════════════════════════════════════

/**
 * Resetea el contador de energía de todas las fases
 * Llamar solo cuando sea necesario (ej: inicio de mes)
 */
void resetearContadoresEnergia() {
  Serial.println("⚠️ Reseteando contadores de energía...");
  
  if (pzemR.resetEnergy()) {
    Serial.println("   ✅ Fase R: OK");
  } else {
    Serial.println("   ❌ Fase R: Error");
  }
  
  if (pzemS.resetEnergy()) {
    Serial.println("   ✅ Fase S: OK");
  } else {
    Serial.println("   ❌ Fase S: Error");
  }
  
  if (pzemT.resetEnergy()) {
    Serial.println("   ✅ Fase T: OK");
  } else {
    Serial.println("   ❌ Fase T: Error");
  }
  
  Serial.println("✅ Reset completado");
}
```

---

## 🖥️ Configuración desde la Plataforma Luminova

### Paso 1: Registrar el Dispositivo en el Sistema

Antes de cargar el firmware, debes registrar el dispositivo en la plataforma web:

1. **Acceder al Panel de Administración**
   ```
   URL: http://localhost:5173 (o tu dominio en producción)
   Login: admin@luminova.cl / admin123
   ```

2. **Ir a Gestión de Dispositivos**
   ```
   Menú lateral → Admin → Gestión de Dispositivos
   ```

3. **Crear Nuevo Dispositivo**
   
   Hacer click en el botón `+ Nuevo Dispositivo` y llenar el formulario:

   | Campo | Valor | Descripción |
   |-------|-------|-------------|
   | **ID Dispositivo** | `IOT-ESP32-001` | Identificador único del hardware |
   | **Nombre** | `Monitor Trifásico Principal` | Nombre descriptivo |
   | **Tipo** | `medidor_trifasico` | Seleccionar tipo de dispositivo |
   | **Empalme** | `6098974` (o el tuyo) | Empalme al que pertenece |
   | **Estado** | `activo` | Estado inicial |
   | **Dirección IP** | `192.168.1.150` (opcional) | IP que tendrá el ESP32 |
   | **Puerto** | `5000` | Puerto UDP |
   | **MAC Address** | `AA:BB:CC:DD:EE:FF` | MAC del ESP32 (opcional) |
   | **Ubicación** | `Tablero Principal Planta 1` | Ubicación física |
   | **Fabricante** | `Luminova DIY` | Fabricante |
   | **Modelo** | `ESP32-PZEM-3F` | Modelo del dispositivo |
   | **Número Serie** | `ESP32-2026-001` | Número de serie |

4. **Guardar Dispositivo**
   
   Click en `Crear Dispositivo`. El sistema validará y guardará los datos.

   ✅ **Verificación:** El dispositivo aparecerá en la lista con estado "Inactivo" hasta que se conecte.

### Paso 2: Obtener Configuración del Dispositivo

Una vez registrado, el dispositivo tendrá:

```json
{
  "_id": "67a3f2e1b8c9d0001234abcd",
  "dispositivoId": "IOT-ESP32-001",
  "nombre": "Monitor Trifásico Principal",
  "tipo": "medidor_trifasico",
  "empalmeId": "6098974",
  "estado": "activo",
  "direccionIP": "192.168.1.150",
  "puerto": 5000
}
```

### Paso 3: Verificar Asignación al Empalme

1. **Ir a Gestión de Empalmes**
   ```
   Menú lateral → Admin → Gestión de Empalmes
   ```

2. **Seleccionar tu Empalme**
   
   Click en el empalme correspondiente (ej: `6098974 - Empalme Industrial Norte`)

3. **Verificar Dispositivos Asignados**
   
   En la vista de detalle, verás la lista de dispositivos asignados:
   
   ```
   📟 Dispositivos Asignados (1)
   ├─ IOT-ESP32-001 - Monitor Trifásico Principal
   │  └─ Estado: Esperando conexión...
   ```

---

## 🌐 Configuración WiFi del Firmware

### Configuración Estática (Recomendada para Producción)

Modifica las siguientes líneas en el código según la configuración de tu instalación:

```cpp
// ═══════════════════════════════════════════════════════════════════
// 🔧 CONFIGURACIÓN - MODIFICAR SEGÚN TU INSTALACIÓN
// ═══════════════════════════════════════════════════════════════════

// ▸ Configuración WiFi
const char* WIFI_SSID     = "MiRedWiFi";           // Nombre de tu red WiFi
const char* WIFI_PASSWORD = "MiPasswordSegura123"; // Contraseña WiFi

// ▸ Servidor Luminova Backend
const char* SERVER_IP     = "192.168.1.100";       // IP del servidor backend
const uint16_t SERVER_PORT = 5000;                 // Puerto UDP (no cambiar)

// ▸ Identificador del Empalme (DEBE existir en MongoDB)
const char* EMPALME_ID    = "6098974";             // Tu ID de empalme
```

**⚠️ IMPORTANTE:** El `EMPALME_ID` debe coincidir exactamente con el `empalmeId` registrado en la base de datos MongoDB.

### Configuración Dinámica via Serial (Opcional)

Si deseas configurar el dispositivo sin recompilar, agrega este código al setup:

```cpp
void configSerial() {
  Serial.println("Modo configuración. Comandos disponibles:");
  Serial.println("  SSID:nombre     - Configurar red WiFi");
  Serial.println("  PASS:password   - Configurar contraseña");
  Serial.println("  SERVER:ip:port  - Configurar servidor");
  Serial.println("  ID:empalmeId    - Configurar ID empalme");
  Serial.println("  SAVE            - Guardar en EEPROM");
  Serial.println("  EXIT            - Salir de configuración");
  
  while (Serial.available() > 0) Serial.read();  // Limpiar buffer
  
  unsigned long timeout = millis() + 30000;  // 30 segundos
  
  while (millis() < timeout) {
    if (Serial.available() > 0) {
      String cmd = Serial.readStringUntil('\n');
      cmd.trim();
      
      if (cmd.startsWith("SSID:")) {
        String ssid = cmd.substring(5);
        Serial.printf("SSID configurado: %s\n", ssid.c_str());
        // Guardar en config...
      }
      else if (cmd == "EXIT") {
        break;
      }
      
      timeout = millis() + 30000;  // Reset timeout
    }
    delay(100);
  }
}
```

---

## 📡 Protocolo de Comunicación

### Formato de Mensaje UDP

El dispositivo envía un mensaje de texto plano con **20 valores separados por espacio**:

```
[empalmeId] [V_R] [I_R] [P_R] [E_R] [F_R] [FP_R] [V_S] [I_S] [P_S] [E_S] [F_S] [FP_S] [V_T] [I_T] [P_T] [E_T] [F_T] [FP_T] [señal]
```

### Descripción de Campos

| Posición | Campo | Descripción | Unidad | Ejemplo |
|----------|-------|-------------|--------|---------|
| 0 | empalmeId | ID único del empalme | - | 6098974 |
| 1 | V_R | Voltaje Fase R | V | 220.50 |
| 2 | I_R | Corriente Fase R | A | 15.30 |
| 3 | P_R | Potencia Fase R | kW | 3.25 |
| 4 | E_R | Energía Fase R | kWh | 1250 |
| 5 | F_R | Frecuencia Fase R | Hz | 50.00 |
| 6 | FP_R | Factor Potencia R | - | 0.95 |
| 7 | V_S | Voltaje Fase S | V | 221.20 |
| 8 | I_S | Corriente Fase S | A | 12.80 |
| 9 | P_S | Potencia Fase S | kW | 2.70 |
| 10 | E_S | Energía Fase S | kWh | 1180 |
| 11 | F_S | Frecuencia Fase S | Hz | 50.00 |
| 12 | FP_S | Factor Potencia S | - | 0.97 |
| 13 | V_T | Voltaje Fase T | V | 219.80 |
| 14 | I_T | Corriente Fase T | A | 10.50 |
| 15 | P_T | Potencia Fase T | kW | 2.20 |
| 16 | E_T | Energía Fase T | kWh | 1050 |
| 17 | F_T | Frecuencia Fase T | Hz | 50.00 |
| 18 | FP_T | Factor Potencia T | - | 0.98 |
| 19 | señal | RSSI WiFi | dBm | -65 |

### Ejemplo Real

```
6098974 220.50 15.30 3.25 1250 50.00 0.95 221.20 12.80 2.70 1180 50.00 0.97 219.80 10.50 2.20 1050 50.00 0.98 -65
```

### Validación en Backend

El servidor Luminova valida:

1. ✅ Exactamente 20 valores separados por espacio
2. ✅ empalmeId es numérico y existe en la base de datos
3. ✅ Todos los valores numéricos son parseables
4. ✅ Rate limit: máximo 10 mensajes/segundo por dispositivo

---

## 🔧 Calibración

### Calibración de Voltaje

Si el voltaje medido difiere del real, ajusta el factor de calibración:

```cpp
// Factor de calibración de voltaje (por defecto 1.0)
const float CAL_VOLTAGE_R = 1.0;  // Multiplicador fase R
const float CAL_VOLTAGE_S = 1.0;  // Multiplicador fase S
const float CAL_VOLTAGE_T = 1.0;  // Multiplicador fase T

// En la función leerFase():
data.voltaje = pzem.voltage() * CAL_VOLTAGE_R;
```

### Calibración de Corriente

Para calibrar corriente, usa un amperímetro de referencia:

```cpp
// Factor de calibración de corriente
const float CAL_CURRENT_R = 1.0;
const float CAL_CURRENT_S = 1.0;
const float CAL_CURRENT_T = 1.0;

// En la función leerFase():
data.corriente = pzem.current() * CAL_CURRENT_R;
```

### Procedimiento de Calibración

1. **Voltaje:**
   - Medir voltaje real con multímetro certificado
   - Dividir voltaje real / voltaje medido = factor
   - Ejemplo: 220V real / 218V medido = 1.009

2. **Corriente:**
   - Conectar carga conocida (ej: calefactor 1000W = ~4.5A en 220V)
   - Medir corriente real con amperímetro de pinza
   - Calcular factor de ajuste

3. **Guardar factores:**
   - Modificar constantes CAL_*
   - Recompilar y cargar firmware

---

## 📦 Carcasa 3D

### Diseño de Carcasa (OpenSCAD)

```openscad
// Carcasa para dispositivo Luminova IoT
// Dimensiones: 120x80x35mm

$fn = 50;

// Dimensiones
largo = 120;
ancho = 80;
alto = 35;
grosor = 2;

// Carcasa base
module carcasa_base() {
    difference() {
        // Caja exterior
        cube([largo, ancho, alto]);
        
        // Cavidad interior
        translate([grosor, grosor, grosor])
            cube([largo - 2*grosor, ancho - 2*grosor, alto]);
        
        // Orificios para cables (lado derecho)
        // Fase R
        translate([largo - grosor/2, 15, 15])
            rotate([0, 90, 0])
                cylinder(h = grosor + 1, d = 8);
        
        // Fase S
        translate([largo - grosor/2, 35, 15])
            rotate([0, 90, 0])
                cylinder(h = grosor + 1, d = 8);
        
        // Fase T
        translate([largo - grosor/2, 55, 15])
            rotate([0, 90, 0])
                cylinder(h = grosor + 1, d = 8);
        
        // Orificio USB (frontal)
        translate([grosor/2, ancho/2 - 5, 5])
            rotate([0, -90, 0])
                cube([12, 10, grosor + 1]);
        
        // Ventilación (superior)
        for (i = [0:4]) {
            translate([20 + i*20, grosor/2, alto - grosor/2])
                cube([10, ancho - grosor, grosor + 1]);
        }
    }
}

// Tapa con clip
module tapa() {
    difference() {
        union() {
            cube([largo, ancho, grosor]);
            
            // Clips de sujeción
            translate([10, grosor + 0.5, -5])
                cube([3, 2, 5]);
            translate([largo - 13, grosor + 0.5, -5])
                cube([3, 2, 5]);
            translate([10, ancho - grosor - 2.5, -5])
                cube([3, 2, 5]);
            translate([largo - 13, ancho - grosor - 2.5, -5])
                cube([3, 2, 5]);
        }
        
        // Logo/texto
        translate([largo/2, ancho/2, grosor - 0.5])
            linear_extrude(1)
                text("LUMINOVA", size = 8, halign = "center", valign = "center");
    }
}

// Soportes internos para PCB
module soportes_pcb() {
    // ESP32 (70x25mm aprox)
    // Posición: centrado
    esp_x = (largo - 70) / 2;
    esp_y = 5;
    
    soporte_h = 5;
    soporte_d = 6;
    agujero_d = 2.5;
    
    // 4 soportes esquinas
    translate([esp_x + 5, esp_y + 5, grosor])
        soporte(soporte_h, soporte_d, agujero_d);
    translate([esp_x + 65, esp_y + 5, grosor])
        soporte(soporte_h, soporte_d, agujero_d);
    translate([esp_x + 5, esp_y + 20, grosor])
        soporte(soporte_h, soporte_d, agujero_d);
    translate([esp_x + 65, esp_y + 20, grosor])
        soporte(soporte_h, soporte_d, agujero_d);
}

module soporte(h, d, hole) {
    difference() {
        cylinder(h = h, d = d);
        translate([0, 0, -0.5])
            cylinder(h = h + 1, d = hole);
    }
}

// Render
carcasa_base();
translate([0, 0, alto + 10])
    tapa();
soportes_pcb();
```

### Archivos STL

Los archivos STL listos para imprimir estarán disponibles en:
- `idea/3d/carcasa_base.stl`
- `idea/3d/carcasa_tapa.stl`

### Parámetros de Impresión

| Parámetro | Valor Recomendado |
|-----------|-------------------|
| Material | PETG o ABS |
| Altura de capa | 0.2mm |
| Relleno | 20% |
| Paredes | 3 |
| Soportes | No necesarios |
| Temperatura boquilla | 240°C (PETG) |
| Temperatura cama | 70°C (PETG) |

---

## 🔨 Guía de Montaje

### Paso 1: Preparar Componentes

1. Verificar que todos los componentes estén en buen estado
2. Identificar cada PZEM y CT correspondiente
3. Preparar cables de conexión (AWG 18-22)

### Paso 2: Soldar Conexiones ESP32

```
1. Soldar header pins al ESP32 si no vienen soldados
2. Soldar cables a los pines:
   - GPIO16/17 → PZEM R
   - GPIO25/26 → PZEM S
   - GPIO32/33 → PZEM T
3. Verificar continuidad con multímetro
```

### Paso 3: Conectar PZEMs

```
Para cada PZEM-004T:
1. Conectar VCC a 5V
2. Conectar GND a GND
3. Conectar TX del PZEM a RX del ESP32
4. Conectar RX del PZEM a TX del ESP32
5. NO CONECTAR AÚN las líneas de alto voltaje
```

### Paso 4: Cargar Firmware

```bash
# Instalar Arduino IDE 2.x
# Agregar soporte ESP32:
# File → Preferences → Additional boards manager URLs:
# https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

# Instalar librerías:
# - PZEM004Tv30 by Jakub Mandula
# - ArduinoJson by Benoit Blanchon

# Seleccionar placa: ESP32 Dev Module
# Puerto: /dev/ttyUSB0 (Linux) o COMx (Windows)
# Velocidad upload: 921600

# Compilar y cargar
```

### Paso 5: Probar Comunicación

```bash
# En el servidor Luminova, verificar que llegan los datos UDP:
# Backend corriendo con DATA_INGESTION_MODE=udp

# Ver logs del backend:
tail -f logs/backend.log | grep UDP

# Debería mostrar:
# 📡 UDP Receiver escuchando en 0.0.0.0:5000
# 📊 Dispositivo 6098974: 100 mensajes recibidos
```

### Paso 6: Conexión Eléctrica (⚠️ PELIGRO)

```
⚠️ ADVERTENCIA: ALTO VOLTAJE - PELIGRO DE MUERTE
⚠️ Esta etapa debe ser realizada por un electricista certificado
⚠️ Desconectar el breaker principal antes de trabajar

1. Identificar las 3 fases (L1/R, L2/S, L3/T) y Neutro
2. Conectar CT de 100A en cada fase:
   - Abrir el CT
   - Pasar el cable de la fase por dentro
   - Cerrar firmemente
   - Conectar cables del CT al PZEM correspondiente
3. Conectar línea de voltaje:
   - PZEM R → Fase L1 y Neutro
   - PZEM S → Fase L2 y Neutro
   - PZEM T → Fase L3 y Neutro
4. Conectar fuente de alimentación 5V a Fase y Neutro
5. Verificar aislamiento con megóhmetro
6. Cerrar carcasa
7. Reconectar breaker
```

### Paso 7: Verificación Final

```bash
# En el frontend de Luminova:
1. Ir a Dashboard → Empalmes → [Tu Empalme]
2. Verificar que se muestran lecturas en tiempo real
3. Comparar lecturas con medidor de referencia
4. Verificar todas las fases muestran datos coherentes

# Checklist:
☐ Voltaje ~220V en cada fase
☐ Corriente coherente con cargas conectadas
☐ Frecuencia ~50Hz (o 60Hz según país)
☐ Factor de potencia entre 0.8 y 1.0
☐ Energía incrementando gradualmente
☐ Señal WiFi estable (> -70dBm)
```

### Paso 8: Verificar Estado del Dispositivo en el Frontend

Una vez que el dispositivo comience a enviar datos:

1. **Ir a Gestión de Dispositivos**
   ```
   Admin → Gestión de Dispositivos
   ```

2. **Buscar tu Dispositivo**
   
   El dispositivo `IOT-ESP32-001` ahora mostrará:
   
   | Campo | Estado |
   |-------|--------|
   | **Estado** | 🟢 Activo |
   | **Última Conexión** | Hace 2 segundos |
   | **IP Asignada** | 192.168.1.150 |
   | **Mensajes Recibidos** | 150 (incrementando) |
   | **Señal WiFi** | -60 dBm (Excelente) |

3. **Ver Detalles en Tiempo Real**
   
   Click en el dispositivo para ver métricas detalladas:
   
   ```
   📊 Últimas Lecturas
   ├─ Fase R: 220.5V, 15.3A, 3.25kW, FP: 0.95
   ├─ Fase S: 221.2V, 12.8A, 2.70kW, FP: 0.97
   └─ Fase T: 219.8V, 10.5A, 2.20kW, FP: 0.98
   
   📈 Estadísticas (Últimas 24h)
   ├─ Uptime: 99.8%
   ├─ Mensajes enviados: 43,200
   ├─ Errores: 0
   └─ Energía total: 8.15 kW
   ```

4. **Verificar en Dashboard del Empalme**
   
   ```
   Dashboard → Empalmes → 6098974 - Empalme Industrial Norte
   
   Tab "Dispositivos":
   ┌─────────────────────────────────────────────────────┐
   │ 📟 IOT-ESP32-001 - Monitor Trifásico Principal      │
   ├─────────────────────────────────────────────────────┤
   │ Estado: 🟢 Activo | Última lectura: Hace 2s         │
   │ Voltaje promedio: 220.5V                            │
   │ Corriente total: 38.6A                              │
   │ Potencia total: 8.15kW                              │
   │ Factor de potencia: 0.97                            │
   └─────────────────────────────────────────────────────┘
   ```

5. **Monitorear Gráficas en Tiempo Real**
   
   En el tab "Gráficas" del empalme, verás:
   - Gráfica de voltaje trifásico (líneas R, S, T)
   - Gráfica de corriente por fase
   - Gráfica de potencia acumulada
   - Frecuencia de red
   - Factor de potencia

   Todas actualizándose cada 2 segundos con los datos del dispositivo.

### Paso 9: Configurar Alertas para el Dispositivo

1. **Ir a Configuración de Alertas**
   ```
   Admin → Configuración de Alertas
   ```

2. **Crear Reglas Específicas**
   
   Ejemplo de alertas para el dispositivo ESP32:
   
   | Tipo Alerta | Parámetro | Umbral | Acción |
   |-------------|-----------|--------|--------|
   | Sobrevoltaje | Voltaje R/S/T | > 240V | Email + Notificación |
   | Subvoltaje | Voltaje R/S/T | < 200V | Email + Notificación |
   | Sobrecorriente | Corriente R/S/T | > 80A | Email + SMS |
   | Señal Débil | RSSI WiFi | < -75 dBm | Notificación |
   | Dispositivo Offline | Última lectura | > 30s | Email |

3. **Activar Alertas**
   
   Las alertas se activarán automáticamente cuando el dispositivo envíe datos que superen los umbrales.

---

## 📊 Monitoreo Avanzado desde el Frontend

### Dashboard de Estadísticas

Ir a `Estadísticas` en el menú principal para ver:

1. **Top Eventos**
   - Sobrevoltajes detectados
   - Picos de corriente
   - Caídas de frecuencia
   - Desconexiones del dispositivo

2. **Comparativa de Períodos**
   ```
   Comparar consumo del empalme 6098974:
   - Esta semana vs semana pasada
   - Este mes vs mes pasado
   - Horario punta vs valle
   ```

3. **Análisis de Eficiencia**
   - Factor de potencia promedio por fase
   - Desbalance de fases
   - Armónicos detectados
   - Pérdidas estimadas

### Reportes Exportables

1. **Ir a Reportes**
   ```
   Menú → Reportes
   ```

2. **Seleccionar Tipo de Reporte**
   - Reporte de Consumo Energético
   - Reporte de Costos
   - Reporte de Eficiencia
   - Reporte Comparativo

3. **Configurar Filtros**
   ```
   Empalme: 6098974
   Dispositivo: IOT-ESP32-001 (opcional)
   Período: Última semana
   Agrupación: Por día
   ```

4. **Exportar en Múltiples Formatos**
   - 📄 PDF (con gráficas)
   - 📊 Excel/CSV (datos tabulares)
   - 🖼️ PNG (gráficas individuales)
   - 📦 ZIP (completo)

---

## 🔧 Administración del Dispositivo

### Cambiar Estado del Dispositivo

1. **Ir a Gestión de Dispositivos**
2. **Seleccionar el dispositivo** `IOT-ESP32-001`
3. **Click en botón "Editar"**
4. **Cambiar estado:**
   - `Activo` - Funcionamiento normal
   - `Inactivo` - Temporalmente deshabilitado
   - `Mantenimiento` - En proceso de mantenimiento
   - `Error` - Requiere atención

5. **Guardar cambios**

### Modificar Configuración del Dispositivo

Desde el modal de edición puedes actualizar:

- **Nombre y descripción** (para mejor identificación)
- **Empalme asignado** (reasignar a otro empalme)
- **Ubicación física** (cambió de tablero)
- **Datos técnicos** (modelo, serie, fabricante)

**Nota:** Cambiar el `dispositivoId` no se recomienda una vez en producción.

### Desactivar/Eliminar Dispositivo

**Desactivar (Recomendado):**
1. Editar dispositivo
2. Cambiar estado a `Inactivo`
3. El sistema dejará de mostrar alertas pero conserva histórico

**Eliminar (Permanente):**
1. Click en botón "Eliminar"
2. Confirmar eliminación
3. ⚠️ Se eliminarán TODOS los datos asociados

---

## 🔐 Configuración de Seguridad

### Credenciales de Acceso

**Usuarios por Defecto:**

| Email | Password | Rol | Permisos |
|-------|----------|-----|----------|
| admin@luminova.cl | admin123 | admin | Acceso total a gestión de dispositivos |
| diego@empresa.cl | cliente123 | cliente | Solo lectura de su empalme |

**⚠️ Cambiar passwords en producción:**

1. Ir a `Perfil de Usuario` (esquina superior derecha)
2. Click en "Cambiar Contraseña"
3. Usar contraseña fuerte (min. 8 caracteres)

### Control de Acceso por Usuario

**Asignar Empalmes a un Usuario Cliente:**

1. **Ir a Gestión de Usuarios**
   ```
   Admin → Gestión de Usuarios
   ```

2. **Seleccionar usuario cliente**
   
3. **Click en "Asignar Empalmes"**
   
4. **Seleccionar empalmes:**
   ```
   ☑ 6098974 - Empalme Industrial Norte
   ☑ 6098972 - Empalme Comercial
   ☐ 6098970 - Empalme Residencial (no asignado)
   ```

5. **Guardar**

Ahora el usuario `diego@empresa.cl` podrá ver:
- Dashboard del empalme 6098974
- Datos en tiempo real del dispositivo IOT-ESP32-001
- Gráficas históricas
- Reportes de consumo

Pero **NO** podrá:
- Crear/editar/eliminar dispositivos
- Ver otros empalmes
- Configurar alertas (solo recibirlas)

---

## 📱 Notificaciones y Alertas

### Configurar Notificaciones por Email

1. **Ir a Configuración de Alertas**
   ```
   Admin → Configuración de Alertas
   ```

2. **Configurar destinatarios:**
   ```json
   {
     "notificaciones": {
       "email": [
         "admin@empresa.cl",
         "operaciones@empresa.cl"
       ],
       "sms": ["+56912345678"],
       "webhook": "https://hooks.slack.com/services/..."
     }
   }
   ```

3. **Seleccionar tipos de alerta:**
   - ☑ Sobrevoltaje crítico (> 240V)
   - ☑ Dispositivo offline (> 5 min)
   - ☑ Error de comunicación
   - ☐ Avisos informativos

### Historial de Alertas

**Ver Registro de Actividad:**

```
Admin → Registro de Actividad

Filtros:
- Tipo: Alerta
- Dispositivo: IOT-ESP32-001
- Período: Última semana

Resultados:
┌────────────────────────────────────────────────────────────┐
│ 🚨 15/01/2026 10:30:45 - Sobrevoltaje en Fase R (245V)   │
│ 📧 Notificación enviada a: admin@empresa.cl               │
│ ✅ Retorno a valores normales: 10:32:10                   │
├────────────────────────────────────────────────────────────┤
│ ⚠️ 15/01/2026 08:15:20 - Señal WiFi débil (-78 dBm)      │
│ 📧 Notificación enviada a: operaciones@empresa.cl        │
│ ✅ Señal mejoró: 08:18:00 (-62 dBm)                       │
└────────────────────────────────────────────────────────────┘
```

---

## 🎯 Casos de Uso Prácticos

### Caso 1: Monitoreo de Instalación Industrial

**Escenario:** Fábrica con sistema trifásico 380V

**Configuración:**
1. Registrar dispositivo `IOT-FAB-001`
2. Asignar a empalme industrial
3. Configurar alertas:
   - Sobrecorriente > 90A (80% de capacidad)
   - Desbalance de fases > 15%
   - Factor de potencia < 0.90

**Beneficios:**
- Prevención de sobrecarga
- Optimización de factor de potencia
- Histórico para auditorías energéticas

### Caso 2: Edificio Comercial Multi-Tenant

**Escenario:** Edificio con 3 empalmes independientes

**Configuración:**
1. Registrar 3 dispositivos:
   - `IOT-EDI-P1` (Piso 1-3)
   - `IOT-EDI-P4` (Piso 4-6)
   - `IOT-EDI-COMUN` (Áreas comunes)

2. Crear usuarios por tenant:
   - `tenant1@edificio.cl` → Solo ve IOT-EDI-P1
   - `tenant2@edificio.cl` → Solo ve IOT-EDI-P4
   - `admin@edificio.cl` → Ve todos

**Beneficios:**
- Facturación individual por consumo real
- Transparencia para inquilinos
- Control de consumos comunes

### Caso 3: Residencial con Paneles Solares

**Escenario:** Casa con generación solar + red pública

**Configuración:**
1. Dispositivo principal: `IOT-CASA-RED`
2. Dispositivo solar: `IOT-CASA-SOLAR`
3. Comparativa de generación vs consumo

**Beneficios:**
- Optimización de autoconsumo
- ROI de inversión solar
- Detección de fallas en paneles

---

## 🔄 Actualización del Firmware OTA (Futuro)

### Preparación para OTA

Actualmente el firmware se carga via USB. Para habilitar OTA (Over-The-Air):

```cpp
// Agregar al código ESP32:
#include <ArduinoOTA.h>

void setup() {
  // ... código existente ...
  
  // Configurar OTA
  ArduinoOTA.setHostname("luminova-iot-001");
  ArduinoOTA.setPassword("admin"); // Cambiar en producción
  
  ArduinoOTA.onStart([]() {
    Serial.println("Iniciando actualización OTA...");
  });
  
  ArduinoOTA.onEnd([]() {
    Serial.println("\nActualización completada!");
  });
  
  ArduinoOTA.begin();
}

void loop() {
  ArduinoOTA.handle();
  // ... código existente ...
}
```

**Actualizar desde el frontend:**
1. Ir a Gestión de Dispositivos
2. Seleccionar dispositivo
3. Click en "Actualizar Firmware"
4. Subir archivo `.bin`
5. El dispositivo se actualiza automáticamente

*Nota: Esta funcionalidad se implementará en versiones futuras.*

---

## 🔍 Troubleshooting

### Problema: No se conecta a WiFi

```cpp
// Verificar:
1. SSID y password correctos (case sensitive)
2. Rango de WiFi alcanza el dispositivo
3. Router permite conexiones nuevas
4. Probar con hotspot del celular

// Solución: Agregar debug
Serial.printf("Estado WiFi: %d\n", WiFi.status());
// Códigos:
// 0 = WL_IDLE_STATUS
// 1 = WL_NO_SSID_AVAIL
// 4 = WL_CONNECT_FAILED
// 6 = WL_DISCONNECTED
```

### Problema: PZEM no responde

```cpp
// Verificar:
1. Conexiones TX/RX correctas (cruzadas)
2. Alimentación 5V estable
3. Cable de voltaje conectado al PZEM

// Test individual:
Serial.printf("Voltaje R: %.2f\n", pzemR.voltage());
// Si muestra NaN → problema de comunicación
// Si muestra 0 → no hay voltaje en la línea
```

### Problema: Datos no llegan al servidor

```bash
# Verificar conectividad:
ping 192.168.1.100  # IP del servidor

# Verificar puerto UDP abierto:
nc -u -l 5000  # En el servidor, escuchar UDP

# Verificar firewall:
sudo ufw allow 5000/udp  # Linux
```

### Problema: Lecturas incorrectas

```cpp
// Verificar:
1. CT instalado en dirección correcta (flecha hacia carga)
2. CT completamente cerrado
3. Solo UN cable pasa por el CT (no neutro+fase)
4. Calibración realizada

// Señales de CT mal instalado:
- Corriente negativa
- Potencia muy diferente a V*I*FP
- Energía no incrementa
```

### Problema: Reinicio constante del ESP32

```cpp
// Causas comunes:
1. Fuente de alimentación insuficiente (necesita 2A)
2. Watchdog timeout (loop muy largo)
3. Stack overflow (variables muy grandes)

// Solución: Agregar al setup()
esp_task_wdt_init(30, true);  // 30 segundos timeout
```

### LED de Estado

| Comportamiento LED | Significado |
|--------------------|-------------|
| Apagado | Sin alimentación o error crítico |
| Parpadeo rápido | Conectando a WiFi |
| Encendido fijo | WiFi conectado, sistema OK |
| Parpadeo cada 2s | Enviando datos normalmente |
| Parpadeo irregular | Error de comunicación PZEM |

---

## 📊 Monitoreo del Dispositivo

### Endpoint de Estadísticas UDP

El backend expone estadísticas del receptor UDP:

```bash
GET http://localhost:3000/udp-stats
```

Respuesta:
```json
{
  "isRunning": true,
  "port": 5000,
  "devices": {
    "6098974": {
      "lastSeen": "2026-01-15T10:30:45.123Z",
      "messageCount": 15234,
      "errorCount": 0
    }
  },
  "totalMessages": 15234,
  "totalErrors": 0
}
```

### Logs del Backend

```bash
# Ver actividad en tiempo real
tail -f logs/luminova.log | grep -E "UDP|6098974"

# Salida esperada:
[2026-01-15 10:30:45] 📡 UDP: 6098974 220.50 15.30 3.25...
[2026-01-15 10:30:47] 📡 UDP: 6098974 220.60 15.28 3.24...
[2026-01-15 10:30:49] 📡 UDP: 6098974 220.55 15.31 3.26...
```

---

## 🔒 Seguridad

### Recomendaciones

1. **WiFi:**
   - Usar WPA2/WPA3
   - Red separada para IoT (VLAN)
   - No exponer dispositivo a Internet

2. **Física:**
   - Carcasa con protección IP65
   - Instalación en tablero cerrado
   - Etiquetado de advertencia

3. **Eléctrica:**
   - Protección diferencial en el circuito
   - Fusibles en líneas de voltaje PZEM
   - Aislamiento galvánico CT

### Futuras Mejoras

- [ ] Cifrado de comunicación UDP
- [ ] Autenticación por API key
- [ ] OTA (actualización over-the-air)
- [ ] MQTT como alternativa a UDP

---

## 📄 Licencia

Este diseño es parte del proyecto Luminova y está bajo licencia propietaria.
Solo para uso interno y clientes autorizados.

---

## 📞 Soporte

Para consultas técnicas sobre el hardware:
- **Email:** soporte@luminova.cl
- **Issues:** GitHub Issues (repositorio privado)
- **Documentación:** `backend/docs/`

---

**Versión del documento:** 1.0.0  
**Última actualización:** 15 de enero 2026  
**Autor:** Luminova Team
