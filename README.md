# Informe de Auditoría de Red Wi-Fi Insegura

**Autor:** Christian Lange  
**Fecha:** 29 de septiembre de 2026  
**Rol:** Auditor de seguridad junior (ejercicio práctico)

---

## Introducción

Cuando una persona se conecta a la Wi-Fi de una cafetería o de un aeropuerto, comparte el "aire" con decenas de desconocidos. Todo lo que su computadora envía viaja como paquetes de datos que pueden ser observados por otros usuarios de la misma red.

En este informe se documentan los riesgos reales de navegar por un sitio **HTTP** desde una red pública. Se analiza una solicitud real, se identifica qué información queda expuesta, se explica cómo una **VPN** protege el tráfico y se proponen **3 Reglas de Oro** para navegar con seguridad.

---

## Sitio analizado

| Dato | Valor |
|---|---|
| Sitio visitado | `http://neverssl.com` (sitio que funciona de forma intencional sin cifrado, es decir, solo por HTTP) |
| Host observado en la captura | `astoundingsilverserenemagic.neverssl.com` (subdominio al que neverssl.com redirige) |
| Servidor (IP) | `34.223.124.45`, puerto 80 |
| Protocolo | HTTP/1.1, sin cifrado |
| Herramienta de captura | Wireshark |

---

## Evidencia observada

### Datos de la solicitud

| Campo | Valor observado |
|---|---|
| URL solicitada | `http://astoundingsilverserenemagic.neverssl.com/online/` |
| Método HTTP | `GET` |
| Host | `astoundingsilverserenemagic.neverssl.com` |
| Protocolo utilizado | `HTTP/1.1` (texto plano, puerto 80) |
| Respuesta del servidor | `200 OK` (`text/html`) |

### Headers enviados por el navegador

```http
GET /online/ HTTP/1.1
Host: astoundingsilverserenemagic.neverssl.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://neverssl.com/
Accept-Encoding: gzip, deflate
Accept-Language: es-ES,es;q=0.9
```

### Respuesta del servidor (fragmento)

```http
HTTP/1.1 200 OK
Server: Apache/2.4.68
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
```

### Capturas de pantalla

**Solicitud `GET` capturada con Wireshark:** se leen el método, el host, la URI y todos los headers en texto plano.

<img width="960" height="530" alt="wireshark-get-online" src="https://github.com/user-attachments/assets/edc41d62-cd53-4a79-9fa9-808bb8af4d74" />

**Respuesta `200 OK` del servidor:** el contenido de la página también viaja legible.

<img width="960" height="478" alt="wireshark-200-ok" src="https://github.com/user-attachments/assets/9bac5571-8b59-40f8-9867-623a85e77a5c" />

### Información que puede observarse durante la solicitud

Solo con mirar el tráfico, sin ninguna herramienta especial, se ven al menos estos elementos:

- **Host:** el sitio al que se conecta el usuario (`astoundingsilverserenemagic.neverssl.com`).
- **URL:** la ruta exacta que se visita (`/online/`).
- **Método:** `GET`, es decir, qué acción se está pidiendo.
- **Headers:** información adicional que el navegador envía sin que el usuario lo note.
- **User-Agent:** revela el navegador (Chrome 144) y el sistema operativo (Windows).
- **Accept-Language:** revela el idioma y la región del usuario (`es-ES`).
- **Referer:** muestra desde qué página venía (`http://neverssl.com/`), es decir, el recorrido de navegación.

---

## Riesgos encontrados

### ¿Por qué las redes públicas son tan peligrosas?

Una Wi-Fi pública funciona como hablar en voz alta en una plaza: cualquiera que pase cerca puede escuchar. Los ataques más comunes son:

1. **Intercepción de datos (sniffing):** alguien en la misma red usa herramientas como Wireshark para "escuchar" los paquetes que la computadora envía al router.
2. **Ataques de intermediario (Man-in-the-Middle):** el atacante engaña al dispositivo para que crea que él es el router. Toda la información pasa primero por su equipo antes de salir a Internet.
3. **Redes gemelas malignas (Evil Twin):** el atacante crea un punto de acceso con un nombre casi idéntico al del local para que la víctima se conecte a él y robarle los datos.

### Qué podría interceptar un atacante al navegar por HTTP

| Información expuesta | Ejemplo en esta captura | Riesgo |
|---|---|---|
| Sitios y páginas visitadas | Host y URL `/online/` | Pérdida de privacidad: el atacante conoce los hábitos de navegación. |
| Datos del dispositivo | User-Agent: Chrome 144 sobre Windows | Permite preparar ataques dirigidos a ese navegador o sistema. |
| Idioma y región | Accept-Language: `es-ES` | Perfilado del usuario. |
| Recorrido de navegación | Referer: `http://neverssl.com/` | El atacante sabe de dónde viene el usuario. |
| Contenido de la página | Respuesta `200 OK` legible | Puede leerse y también **modificarse** en tránsito (por ejemplo, inyectando enlaces falsos). |
| Credenciales, formularios y cookies | No viajaron en esta solicitud, pero en un sitio con sesión irían en texto plano | Robo de usuarios y contraseñas, y **secuestro de la sesión**. |

### Error común

> "Si la red tiene contraseña, es segura."

**Falso.** Si la contraseña se entrega a todos los clientes (por ejemplo, en un ticket), todos tienen la "llave" para intentar descifrar el tráfico de los demás o realizar ataques internos. Además, navegar por sitios HTTP expone la información aunque la red tenga clave.

---

## Cómo ayuda una VPN

Una VPN (red privada virtual) actúa como un **túnel blindado** dentro de la red pública. Aunque el atacante capture los datos, solo verá ruido cifrado ilegible.

- **Cifrado:** el tráfico se cifra en el dispositivo antes de salir a la Wi-Fi. Lo que el atacante captura son bytes sin sentido, no cabeceras ni contenido legible.
- **Túnel seguro (encapsulamiento):** los paquetes originales (por ejemplo, la solicitud `GET` de esta captura) se **encapsulan** dentro de otros paquetes cifrados que viajan hasta el servidor VPN. Desde allí salen hacia Internet.
- **Protección del tráfico:** el contenido, las URL, los headers y las consultas DNS dejan de ser visibles en la red local, por lo que se reduce el riesgo de sniffing y de ataques Man-in-the-Middle.
- **Privacidad:** quien observe la red solo ve que el dispositivo se comunica con el servidor VPN, no qué sitios se visitan. El sitio de destino, además, ve la dirección IP del servidor VPN y no la del usuario.

```text
Sin VPN:  Equipo --(HTTP en texto plano)--> Wi-Fi pública --> Internet --> Sitio

Con VPN:  Equipo ==[túnel cifrado]==> Wi-Fi pública ==> Servidor VPN --> Internet --> Sitio
```

**Limitación:** la VPN cifra el tramo entre el dispositivo y el servidor VPN. Si el sitio usa HTTP, el último tramo (del servidor VPN al sitio) sigue viajando sin cifrar. Por eso la VPN **no reemplaza** a HTTPS: lo ideal es usar ambos.

---

## 3 Reglas de Oro para navegar en redes Wi-Fi públicas

### 1. Verifica que el sitio use HTTPS
Antes de ingresar cualquier dato, confirma que la dirección empiece con `https://` y muestre el candado. Evita ingresar contraseñas, datos bancarios o información personal en sitios HTTP. Ten en cuenta que el candado solo indica que la conexión es privada, no que el sitio sea honesto.

### 2. Usa una VPN de confianza
Activa la VPN antes de conectarte a una red pública. El túnel cifrado protege todo tu tráfico, incluso el de sitios o aplicaciones que no usen HTTPS.

### 3. Reduce tu exposición
- Confirma con el personal del local el **nombre exacto** de la red para evitar una red gemela maligna.
- Desactiva la compartición de archivos e impresoras.
- **Olvida** las redes públicas que ya no uses, para que el dispositivo no se conecte solo.
- Mantén el sistema y el navegador **actualizados** y activa la autenticación multifactor (MFA) en tus cuentas importantes.

---

## Conclusión

El análisis muestra que una simple visita a un sitio HTTP deja al descubierto el host, la URL, el método y los headers, incluyendo datos del dispositivo y del idioma del usuario. En una Wi-Fi pública, cualquier persona conectada a la misma red puede capturar esa información. La combinación de **HTTPS**, **VPN** y buenos hábitos reduce de forma importante ese riesgo.
