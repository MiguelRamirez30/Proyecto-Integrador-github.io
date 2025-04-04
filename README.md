##PROYECTO INTEGRADO

```markdown
# Proyecto de Control de Acceso RFID 🔑✨

Este proyecto utiliza un ESP32 y un módulo RFID RC522 para leer tarjetas RFID y enviar los datos a una base de datos. Además, se incluye una página web para mostrar los accesos y gestionar los registros.

## Contenido del Proyecto 📋🛠️

1. *Código para ESP32*: Implementación del código que permite leer tarjetas RFID.
2. *Página Web*: Interfaz para mostrar los registros de acceso.
3. *Base de Datos*: Almacenamiento de los datos leídos de las tarjetas.

## Estructura del Proyecto 🗂️📂

```
/proyecto-rfid
│
├── esp32_code.ino          # Código para el ESP32
├── index.html              # Página web principal
├── styles.css              # Estilos CSS para la página web
└── script.js               # Script JavaScript para manejar la lógica de la página
```

### Código del ESP32 💻

```cpp
#include "SPI.h"
#include "MFRC522.h"

#define SS_PIN 5      // Pin de selección de esclavo (SDA)
#define RST_PIN 2     // Cambiado a D4 (GPIO 2)
#define BUZZER_PIN 26 // Pin para el buzzer
#define GREEN_LED_PIN 27 // Pin para el LED verde (acceso permitido)
#define RED_LED_PIN 28   // Pin para el LED rojo (acceso denegado)

MFRC522 rfid(SS_PIN, RST_PIN); // Instancia de la clase
MFRC522::MIFARE_Key key;

#define ACCESS_DELAY 2000
#define DENIED_DELAY 1000

void setup() 
{
  Serial.begin(9600);   // Iniciar comunicación serie
  SPI.begin();          // Iniciar bus SPI
  rfid.PCD_Init();      // Iniciar MFRC522 

  pinMode(BUZZER_PIN, OUTPUT); // Configurar el pin del buzzer como salida
  pinMode(GREEN_LED_PIN, OUTPUT); // Configurar el pin del LED verde como salida
  pinMode(RED_LED_PIN, OUTPUT);   // Configurar el pin del LED rojo como salida
  pinMode(33, INPUT_PULLUP);   // Pin para sensor analógico (si lo necesitas)
  
  Serial.println("Put your card to the reader...");
}

void loop() 
{
  Serial.println("Put your card to the reader...");
  
  // Buscar nuevas tarjetas
  if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial())
    return;

  // Obtener tipo de PICC
  MFRC522::PICC_Type piccType = rfid.PICC_GetType(rfid.uid.sak);
  
  // Comprobar si el PICC es de tipo MIFARE Classic
  if (piccType != MFRC522::PICC_TYPE_MIFARE_MINI &&
      piccType != MFRC522::PICC_TYPE_MIFARE_1K &&
      piccType != MFRC522::PICC_TYPE_MIFARE_4K) 
  {
    Serial.println(F("Your tag is not of type MIFARE Classic."));
    return;
  }

  // Crear ID de tarjeta
  String strID = "";
  for (byte i = 0; i < 4; i++) {
    strID += (rfid.uid.uidByte[i] < 0x10 ? "0" : "") + String(rfid.uid.uidByte[i], HEX) + (i != 3 ? ":" : "");
  }
  strID.toUpperCase();
  
  Serial.print("Tap card key: ");
  Serial.println(strID);

  // Comprobar si la tarjeta es autorizada
  if (strID.indexOf("11:65:FB:0B") >= 0) {
    Serial.println("Acceso Autorizado ✅");
    
    // Encender LED verde
    digitalWrite(GREEN_LED_PIN, HIGH);
    digitalWrite(RED_LED_PIN, LOW); // Asegurarse de que el LED rojo esté apagado

    // Emitir un sonido en el buzzer
    tone(BUZZER_PIN, 8000); // Emitir un tono de 8000 Hz
    delay(800); // Esperar medio segundo
    noTone(BUZZER_PIN); // Detener el sonido

    delay(ACCESS_DELAY); // Esperar antes de volver a la lectura
  } else {
    Serial.println("Acceso Denegado ❌");
    
    // Encender LED rojo
    digitalWrite(RED_LED_PIN, HIGH);
    digitalWrite(GREEN_LED_PIN, LOW); // Asegurarse de que el LED verde esté apagado

    // Emitir un sonido





## Descripción de la Página Web 🌐

La página web se encarga de mostrar los registros de acceso de las tarjetas RFID. A continuación se presenta el código HTML básico:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control de Acceso RFID</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Registro de Tarjetas RFID</h1>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>UID</th>
                <th>Nombre</th>
                <th>Acceso</th>
                <th>Fecha de Registro</th>
            </tr>
        </thead>
        <tbody id="tabla-body">
            <!-- Aquí se insertarán los datos dinámicamente -->
        </tbody>
    </table>
    <script src="script.js"></script>
</body>
</html>

Funcionalidades ⚙️
Obtiene los datos de la API en PHP: La página web se conecta a una API que devuelve los registros de acceso.
Llena la tabla de la página web con los registros de las tarjetas: Los datos se muestran dinámicamente en la tabla.




fetch('url_de_tu_api')
    .then(response => response.json())
    .then(data => {
        let tbody = document.getElementById("tabla-body");
        tbody.innerHTML = "";
        data.forEach(registro => {
            let fila = `<tr>
                <td>${registro.id}</td>
                <td>${registro.uid}</td>
                <td>${registro.nombre}</td>
                <td>${registro.acceso ? '✅' : '❌'}</td>
                <td>${registro.fecha_registro}</td>
            </tr>`;
            tbody.innerHTML += fila;
        });
    })
    .catch(error => console.error("Error al obtener datos:", error));



Próximos Pasos 🚀
Agregar CSS para mejorar el diseño: Se puede personalizar el estilo de la página web para que sea más atractiva.
Desplegar todo en un servidor local o en la nube: Configurar un entorno de servidor para que la aplicación sea accesible desde cualquier lugar.
Contribuciones 🤝
Las contribuciones son bienvenidas. Si deseas colaborar, por favor abre un issue o envía un pull request.

Licencia 📄
Este proyecto está bajo la Licencia MIT. Para más detalles, consulta el archivo LICENSE.
