# Conexión a WiFi y Traducción de Texto con Google Cloud Translation

## Objetivo

El objetivo de este proyecto es conectar un ESP32 a una red WiFi, obtener el código de idioma para "Spanish" utilizando la API de Google Cloud Translation y traducir una cadena de texto al español. El texto traducido se imprime en el monitor serie.

## Materiales

- ESP32
- Librerias:
```.ini
    lib_deps = 
        bblanchon/ArduinoJson@^7.0.4
        esphome/ESP32-audioI2S@^2.0.7
        earlephilhower/ESP8266Audio@^1.9.7
```

## Procedimiento

1. Configurar el ESP32 para conectarse a una red WiFi.
2. Obtener el código de idioma para "Spanish" usando la API de Google Cloud Translation.
3. Traducir una cadena de texto al español.
4. Imprimir el texto traducido en el monitor serie.

### Código
```cpp
    #include <Arduino.h>
    #include <WiFi.h>
    #include <WiFiClientSecure.h>
    #include <HTTPClient.h>
    #include <ArduinoJson.h>
    #include "Audio.h"
    #include <SD.h>
    #include <Audio.h>
    #include <SPI.h>
    #include <Base64.h>

    String getLanguageCode(const char* languageName);
    String translateText(String text, const char* apiKey, const char* targetLanguage);


    const char* ssid = "RedmiNuria";
    const char* password = "Patata123";
    const char* apiKey = "AIzaSyCz4Pb-7OIi3Gs6LGgJ-XHZ2Xy__hRAeZQ";

    // Función para conectar a WiFi
    void connectToWiFi() {
        // WiFi.config(INADDR_NONE, INADDR_NONE, INADDR_NONE, dns1, dns2);
        WiFi.begin(ssid, password);
        Serial.print("Conectando a WiFi");
        while (WiFi.status() != WL_CONNECTED) {
            delay(1000);
            Serial.print(".");
        }
        Serial.println("");
        Serial.println("Conectado a la red WiFi");
    }

    void setup() {
        // Inicializar la comunicación serie
        Serial.begin(115200);
        // Comienza la conexión WiFi
        Serial.println();
        Serial.println();
        Serial.print("Conectando a ");
        Serial.println(ssid);

        // Inicializa la conexión WiFi
        WiFi.begin(ssid, password);

        // Espera hasta que la conexión se realice
        while (WiFi.status() != WL_CONNECTED) {
            delay(500);
            Serial.print(".");
        }

        // Una vez conectado, imprime la dirección IP asignada
        Serial.println("");
        Serial.println("WiFi conectado.");
        Serial.print("Dirección IP: ");
        Serial.println(WiFi.localIP());

        // Traducir el texto
        String TL = getLanguageCode("Spanish"); // Código de idioma de destino (ejemplo: "es" para español)
        // Convertir TL a un array de caracteres (char[])
        char TLCharArray[TL.length() + 1];
        TL.toCharArray(TLCharArray, TL.length() + 1);
        String Variable = "Hello, my name is Nuria.";
        // Traducir el texto
        String translatedText = translateText(Variable, apiKey, TLCharArray);
        // Imprimir
        Serial.println(translatedText);
    }
        
    String getLanguageCode(const char* languageName) {
        HTTPClient http;
        String url = "https://translation.googleapis.com/language/translate/v2/languages?key=";
        url += apiKey;
        url += "&target=en";  // Para obtener los nombres de idiomas en inglés

        http.begin(url);
        int httpResponseCode = http.GET();


        String languageCode = "";
        if (httpResponseCode == HTTP_CODE_OK) {
            String response = http.getString();
            StaticJsonDocument<4096> doc;
            deserializeJson(doc, response);
            //DynamicJsonDocument doc(4096);
            //deserializeJson(doc, response);

            JsonArray languages = doc["data"]["languages"];
            for (JsonObject language : languages) {
                //if (String(language["name"]).equalsIgnoreCase(languageName))
                if (String(language["name"].as<const char*>()).equalsIgnoreCase(languageName)) {
                    languageCode = language["language"].as<String>();
                    break;
                }
            }
        } else {
            Serial.print("Error en la solicitud: ");
            Serial.println(httpResponseCode);
        }

        http.end();
        return languageCode;
    }

    // TRADUCTOR

    String translateText(String text, const char* apiKey, const char* targetLanguage) {
        HTTPClient http;

        String url = "https://translation.googleapis.com/language/translate/v2?key=";
        url += apiKey;
        Serial.println(url);
        http.begin(url);
        http.addHeader("Content-Type", "application/json");

        String jsonBody = "{\"q\":\"";
        jsonBody += text;
        jsonBody += "\",\"target\":\"";
        jsonBody += targetLanguage;
        jsonBody += "\",\"format\":\"text\"}";  // No especificar "source" para detección automática

        Serial.println(jsonBody);

        int httpResponseCode = http.POST(jsonBody);

        String translatedText = "";

        if (httpResponseCode == HTTP_CODE_OK) {
            String response = http.getString();
            Serial.println(response);
            // Parsear la respuesta JSON para obtener la traducción
            StaticJsonDocument<1024> doc;
            DeserializationError error = deserializeJson(doc, response);
            //DynamicJsonDocument doc(1024);
            //DeserializationError error = deserializeJson(doc, response);

            if (error) {
                Serial.print("deserializeJson() failed: ");
                Serial.println(error.c_str());
            } else {
                translatedText = doc["data"]["translations"][0]["translatedText"].as<String>();
            }
        } else {
            Serial.print("Error en la solicitud: ");
            Serial.println(httpResponseCode);
        }

        http.end();

        return translatedText;
    }

    void loop() {
        // Aquí puedes agregar el código que necesitas ejecutar repetidamente.
    }
```

### Descripción
Este código configura el ESP32 para conectarse a una red WiFi y utiliza la API de Google Cloud Translation para traducir texto. Primero, se definen las bibliotecas necesarias y las constantes para la configuración de WiFi y las API. La función ``connectToWiFi()`` establece la conexión WiFi. En ``setup()``, se inicializa la comunicación serie y se conecta a la red WiFi. Luego, se obtiene el código del idioma para "Spanish" utilizando ``getLanguageCode()``. La función ``translateText()`` se utiliza para traducir una cadena de texto al idioma deseado. Finalmente, el texto traducido se imprime en el monitor serie.
### Diagrama de flujos
```mermaid 
    graph TD
        A[Inicio] --> B[Conectar a WiFi]
        B --> C[Obtener código de idioma]
        C --> D[Traducir texto]
        D --> E[Imprimir texto traducido en monitor serie]
    E --> F[Fin]
```
## Conclusión: 
Este proyecto muestra cómo conectar un ESP32 a una red WiFi y utilizar la API de Google Cloud Translation para traducir texto. El texto traducido se imprime en el monitor serie.