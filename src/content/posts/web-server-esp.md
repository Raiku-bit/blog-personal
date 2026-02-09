---
title: 'Web Server con ESP8266MOD'
description: 'Servir una página web con un ESP8266MOD'
date: 2026-02-08
image: 'images/posts/esp8266mod.jpeg'
tags: ['Electrónica', 'IoT', 'Programación']
---

# Inspiración 

Compré hace poco un ESP8266MOD y me puse a probarlo con NodeMCU. Como las páginas web me llaman la atención y sabiendo que el ESP8266MOD es bastante ajustado en recursos, dije: "Qué más da, vamos a programar una web sencilla a ver qué tanto resiste".

# Componentes 

- ESP8266MOD
- Un cable para conectarlo a la computadora
- NodeMCU

# Programación

Al ser un ESP8266MOD bastante ajustado en recursos, se usa el lenguaje C++ para programarlo. Así que diría que no es difícil, porque con las bases básicas puedes programarlo, pero sí necesitas un poco de conocimiento de C++. La página se crea en HTML, es bastante fácil de programar y para mi prueba usé algo sencillo como un mensaje que dice mi nombre y un pequeño párrafo de una canción que me gusta.

# Estructura del proyecto

```
web-server-esp/
├── data/
│   ├── index.html
├── src/
│   ├── main.cpp
├── platformio.ini
```

# Código fuente y explicación de cada parte

## Conectar al WiFi el ESP8266MOD 

Usaremos la librería `<ESP8266WiFi.h>`.

```cpp
#include <ESP8266WiFi.h>
// Creamos un servidor en el puerto 80
WiFiServer server(80);
// Conectamos al wifi
WiFi.begin("aqui_va_el_nombre_de_tu_wifi", "aqui_va_la_contraseña_de_tu_wifi");
  Serial.print("Conectando");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  // Información de la conexión
  Serial.println("\nConectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());   

  // Inicio del servidor 
  server.begin();
```

## Servimos el index.html

Usaremos la librería `<LittleFS.h>` para montar el sistema de archivos y poder servir el index.html.    

```cpp
if (!LittleFS.begin()) {
    Serial.println("¡ERROR CRÍTICO! No se pudo montar LittleFS");
  } else {
    Serial.println("LittleFS montado con éxito.");
    Dir dir = LittleFS.openDir("/");
    int contador = 0;
    while (dir.next()) {
      Serial.print("Archivo encontrado: ");
      Serial.print(dir.fileName());
      Serial.print(" - Tamaño: ");
      Serial.println(dir.fileSize());
      contador++;
    }
    if(contador == 0) Serial.println("ADVERTENCIA: La memoria Flash está VACÍA.");
  }
```

## Escuchando al cliente

Creamos un objeto cliente para escuchar las peticiones del cliente y poder responderlas.

```cpp
 // Escuchar al cliente
  WiFiClient client = server.accept(); 
  
  if (client) {
    // Espera activa de datos del cliente
    unsigned long timeout = millis();
    while (client.connected() && !client.available()) {
      if (millis() - timeout > 500) { 
        client.stop();
        return;
      }
      delay(1);
    }

    // Lectura y procesamiento de la petición HTTP
    String request = client.readStringUntil('\r');
    Serial.println("Petición: " + request);
    client.flush();

    // Lógica para determinar el archivo y el tipo de contenido (MIME type)
    String path = "/index.html";
    String contentType = "text/html";

    if (request.indexOf("GET /style.css") != -1) {
      path = "/style.css";
      contentType = "text/css";
    }

    // Apertura del archivo solicitado desde LittleFS
    File file = LittleFS.open(path, "r");
    
    if (file) {
      // Envío de cabeceras HTTP al navegador
      client.println("HTTP/1.1 200 OK");
      client.print("Content-Type: ");
      client.println(contentType);
      client.println("Connection: close");
      client.println();

      // Envío del contenido del archivo y cierre
      client.write(file);
      file.close();
    } 
    else {
      // Respuesta en caso de archivo no encontrado
      client.println("HTTP/1.1 404 Not Found");
      client.println();
      client.println("404: Archivo no encontrado");
    }

    // Finalización de la conexión con el cliente
    delay(1);
    client.stop();
  }
```

# Conclusiones
 
Fue divertido hacer el experimento. Me dirán: "Oye, dale color con un archivo CSS", pero como tiene tan poco procesamiento a la hora de intentar aplicarle estilo me dio el error de reiniciarse el ESP8266MOD o directamente no carga el CSS. Posiblemente sea un error de mi código y se pueda solucionar jugando con los recursos, pero al ser un proyecto que hice en una tarde y disfruté, me siento satisfecho con lo logrado.