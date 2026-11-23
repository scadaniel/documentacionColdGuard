# ColdGuard IoT (`CoolGuard` Firmware)
**Sistema IoT de Monitoreo, Respaldo y Auditoría de Cadena de Frío**

* **Institución:** Instituto de Formación Técnica Superior N° 14 (IFTS N° 14)
* **Cátedras:** Desarrollo de Aplicaciones para IoT (PP3) 
* **Autor / Project Manager:** José Daniel Ons
* **Plataforma Hardware:** Espressif ESP32-S3 + Display TFT 1,47" ST7789 (320×172) + Sensor DHT11
* **Framework:** ESP-IDF (Lenguaje C / FreeRTOS) + LVGL v9.6 (SquareLine Studio)

---

## 1. Descripción General
**ColdGuard IoT** es un nodo embebido inteligente diseñado para supervisar de forma continua la cadena de frío en heladeras críticas de vacunas, medicamentos y reactivos biológicos (+2 °C a +8 °C), alineado con la **Disposición ANMAT N° 2819/2004**.

El firmware ejecuta una arquitectura multitarea sobre **FreeRTOS** que:
1. Adquiere temperatura y humedad cada 2 segundos mediante sensor digital **DHT11**.
2. Despliega los valores en tiempo real sobre una pantalla gráfica **TFT a color de 1,47"** (`ST7789` + `LVGL`).
3. Sincroniza la hora oficial vía **SNTP** (`pool.ntp.org`, huso horario Argentina `UTC-3`).
4. Publica telemetría en formato **JSON** vía **MQTT** (QoS 1) hacia la plataforma Cloud Dockerizada (Mosquitto + MongoDB/MySQL).
5. Ante caídas de red o desconexión del broker, resguarda las mediciones en una cola circular **FIFO en memoria Flash NVS** y las retransmite automáticamente al reconectar.

---

## 2. Arquitectura y Pinout de Hardware (ESP32-S3)

| Periférico / Bus | Señal | GPIO ESP32-S3 | Descripción Técnica |
| :--- | :--- | :---: | :--- |
| **Display TFT 1,47" ST7789** | `SCLK` | `GPIO 40` | Reloj bus SPI2 (20 MHz) |
| **Display TFT 1,47" ST7789** | `MOSI` | `GPIO 45` | Datos SPI hacia controlador ST7789 |
| **Display TFT 1,47" ST7789** | `CS` | `GPIO 42` | Chip Select |
| **Display TFT 1,47" ST7789** | `DC` | `GPIO 41` | Data / Command |
| **Display TFT 1,47" ST7789** | `RST` | `GPIO 39` | Reset por hardware del LCD |
| **Display TFT 1,47" ST7789** | `BL` | `GPIO 46` | Control de retroiluminación (Backlight) |
| **Sensor Ambiental DHT11** | `DATA` | `GPIO 5` | Bus single-wire con pull-up interna |
| **Pulsador Factory Reset** | `BTN` | `GPIO 4` | Pulsación sostenida de 3 s borra NVS |

---

## 3. Estructura del Repositorio

```text
scadaniel-display_st7789/
├── CMakeLists.txt              # Configuración raíz del proyecto ESP-IDF
├── partitions.csv              # Tabla de particiones (NVS 24KB, Factory App 2MB)
├── Doxyfile                    # Configuración para generación automática de docs HTML
├── README.md                   # Documentación principal del proyecto
├── README.txt                  # Versión en texto plano de la documentación
├── docs/                       # Documentación técnica (SRS, Master Test Plan, Project Charter)
├── main/
│   ├── main.c                  # Arranque del sistema, eventos IP y tarea sensor_task
│   └── idf_component.yml       # Dependencias gestionadas (esp_lcd_st7789, lvgl 9.6)
└── components/
    ├── dht11/                  # Driver single-wire con validación de checksum CRC
    ├── display/                # Inicialización SPI2/ST7789, tarea LVGL y pantallas SquareLine
    ├── mqtt_client_gui/        # Cliente MQTT QoS 1 y cola offline FIFO en Flash NVS
    ├── reloj_manager/          # Configuración UTC-3 y sincronización horaria SNTP
    ├── reset_button/           # Supervisor de botón de restauración de fábrica
    └── wifi_manager/           # Gestión Wi-Fi STA y servidor HTTP SoftAP (192.168.4.1)