# Proyecto de Control de Acceso RFID 🔑✨

Este proyecto utiliza un **ESP32** y un módulo **RFID RC522** para leer tarjetas RFID y enviar los datos a una base de datos. Además, se incluye una página web para mostrar los accesos y gestionar los registros.

## Contenido del Proyecto 📋🛠️

1. **Código para ESP32**: Implementación del código que permite leer tarjetas RFID y controlar el acceso.
2. **Página Web**: Interfaz para mostrar los registros de acceso y gestionar la información.
3. **Base de Datos**: Almacenamiento de los datos leídos de las tarjetas RFID.

## Estructura del Proyecto 🗂️📂


/proyecto-rfid │ ├── esp32_code.ino # Código para el ESP32 ├── index.html 
# Página web principal ├── styles.css # Estilos CSS para la página web └── script.js 
# Script JavaScript para manejar la lógica de la página

### Descripción de los Archivos del Proyecto

- **esp32_code.ino**: Este archivo contiene el código que se ejecuta en el microcontrolador ESP32. Se encarga de leer las tarjetas RFID, verificar si son válidas y controlar los LEDs y el buzzer para indicar el estado del acceso.

- **index.html**: Este archivo es la página web principal que se muestra al usuario. Contiene la estructura HTML para mostrar los registros de acceso y permite la interacción con el sistema.

- **styles.css**: Este archivo contiene los estilos CSS que se aplican a la página web para mejorar su apariencia y hacerla más atractiva visualmente.

- **script.js**: Este archivo contiene el código JavaScript que se encarga de manejar la lógica de la página web, como la obtención de datos de la API y la actualización de la tabla de registros.

### Código del ESP32 💻

A continuación se presenta el código del ESP32, que incluye comentarios explicativos sobre su funcionamiento:

```cpp
#include "SPI.h"               // Librería para la comunicación SPI
#include "MFRC522.h"          // Librería para el módulo RFID RC522

#define SS_PIN 5              // Pin de selección de esclavo (SDA)
#define RST_PIN 2             // Pin de reinicio del módulo RFID
#define BUZZER_PIN 26         // Pin para el buzzer
#define GREEN_LED_PIN 27      // Pin para el LED verde (acceso permitido)
#define RED_LED_PIN 28        // Pin para el LED rojo (acceso denegado)

MFRC522 rfid(SS_PIN, RST_PIN); // Instancia de la clase MFRC522
MFRC522::MIFARE_Key key;        // Clave para el acceso a las tarjetas MIFARE

#define ACCESS_DELAY 2000     // Tiempo de espera para acceso permitido
#define DENIED_DELAY 1000     // Tiempo de espera para acceso denegado

void setup() 
{
  Serial.begin(9600);          // Iniciar comunicación serie
  SPI.begin();                 // Iniciar bus SPI
  rfid.PCD_Init();             // Iniciar el módulo MFRC522 

  pinMode(BUZZER_PIN, OUTPUT); // Configurar el pin del buzzer como salida
  pinMode(GREEN_LED_PIN, OUTPUT); // Configurar el pin del LED verde como salida
  pinMode(RED_LED_PIN, OUTPUT);   // Configurar el pin del LED rojo como salida
  pinMode(33, INPUT_PULLUP);   // Pin para sensor analógico (si lo necesitas)
  
  Serial.println("Put your card to the reader..."); // Mensaje inicial
}

void loop() 
{
  Serial.println("Put your card to the reader..."); // Mensaje en el bucle
  
  // Buscar nuevas tarjetas
  if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial())
    return; // Salir si no hay tarjeta presente

  // Obtener tipo de PICC
  MFRC522::PICC_Type piccType = rfid.PICC_GetType(rfid.uid.sak);
  
  // Comprobar si el PICC es de tipo MIFARE Classic
  if (piccType != MFRC522::PICC_TYPE_MIFARE_MINI &&
      piccType != MFRC522::PICC_TYPE_MIFARE_1K &&
      piccType != MFRC522::PICC_TYPE_MIFARE_4K) 
  {
    Serial.println(F("Your tag is not of type MIFARE Classic.")); // Mensaje de error
    return; // Salir si la tarjeta no es válida
  }

  // Crear ID de tarjeta
  String strID
