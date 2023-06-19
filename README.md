
https://www.mischianti.org/2022/08/25/stm32-wifinina-with-esp32-wifi-co-processor/

Install the ESPRESSIF IDE
-------------------------

git clone --branch v4.4.4 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf/
./install.sh # will install stuff in ~/.espressif/

Make NINA FW for the ESP32
--------------------------

. ~/esp-idf/export.sh
git clone https://github.com/arduino/nina-fw
cd nina-fw/
git checkout esp-idf-4.4
git pull
python /home/haba/Arduino/esp-idf/components/esptool_py/esptool/esptool.py --chip esp32 --port /dev/ttyUSB1  --baud 115200 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 40m --flash_size detect 0x1000 ./build/bootloader/bootloader.bin 0xf000 ./build/phy_init_data.bin 0x30000 ./build/nina-fw.bin 0x8000 ./build/partitions.bin

The pins to connect are in arduino/libaries/SPIS/src/SPIS.cpp last line

SPISClass SPIS(VSPI_HOST, 1,   12,   23,  18,  5,   33);
/*                           MOSI, MISO, SCK, CS, BUSY */

Table of the connections:

ESP32	Microcontroller	  STM32 Comment

GPIO05	CS	          PA4   Selectable
GPIO18	SCK		  PA5   Fix
GPIO23	MISO		  PA6   Fix
GPIO12	MOSI		  PA7   Fix
GPIO33	BUSY/READY (IRQ)  PB3   Selectable
EN      RST/EN	          PA10  Selectable

GND  connected
PA10 has pullup to 3.3V
ESP32 by own USB power or 5V to Vin from STM32


Make a test application for the WiFiNINA libary:

platformio.ini contains:
[env:Nucleo-F411RE]
platform = ststm32
board = nucleo_f411re
framework = arduino
lib_deps = 
        ${env.lib_deps}
        https://github.com/adafruit/WiFiNINA
        SPI
build_flags = -std=c++17  -Os -g2 -Wunused-variable
monitor_speed = 115200
monitor_echo = yes

This will download the WiFiNINA from adafruit

 void setPins(int8_t cs=10, int8_t ready=7, int8_t reset=5, int8_t gpio0=6, SPIClass *spi = &SPI);

  #define SPIWIFI       SPI  // The SPI port
  #define SPIWIFI_SS    PA4   // Chip select pin
  #define ESP32_RESETN  PA10   // Reset pin
  #define SPIWIFI_ACK   PB3   // a.k.a BUSY or READY pin
  #define ESP32_GPIO0   -1
  WiFi.setPins(SPIWIFI_SS, SPIWIFI_ACK, ESP32_RESETN, ESP32_GPIO0, &SPIWIFI);
