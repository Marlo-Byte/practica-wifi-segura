# Informe de Auditoría de Red Wi-Fi Insegura

## Introducción

Muchas veces nos conectamos a redes Wi-Fi públicas en un café, un aeropuerto o un shopping sin pensar en qué pasa con los datos que enviamos desde la computadora o el celular. El problema es que, al ser redes compartidas, cualquier persona conectada a la misma red podría intentar mirar el tráfico que circula si no está protegido.

El objetivo de esta práctica es ver en concreto qué información queda expuesta cuando navegamos por un sitio sin cifrado y entender por qué conviene usar HTTPS y una VPN.

Básicamente, la diferencia entre **HTTP** y **HTTPS** está en la seguridad:
- **HTTP** envía los datos en texto plano (sin cifrar), por el puerto 80. Si alguien intercepta la conexión, puede leer todo tal cual se envía.
- **HTTPS** agrega una capa de cifrado (TLS/SSL) y usa el puerto 443. Eso hace que la información viaje cifrada y además verifica que el servidor al que nos conectamos sea realmente quien dice ser.

## Sitio analizado

Para hacer la prueba entré al sitio `http://neverssl.com`. Esta página está hecha a propósito para funcionar únicamente con HTTP y nunca redirigir a HTTPS, lo cual sirve justo para ver cómo se comporta una conexión sin cifrado.

Los pasos que seguí fueron:
1. Abrir `http://neverssl.com` en el navegador.
2. Apretar `F12` para abrir las herramientas de desarrollador (DevTools).
3. Ir a la pestaña **Network** (Red) y recargar la página.
4. Hacer clic en la primera solicitud que apareció para revisar los detalles y los headers.

## Evidencia observada

### ¿Qué protocolo utiliza el sitio?

El sitio utiliza **HTTP** (`HTTP/1.1`), no HTTPS. Esto se ve tanto en la dirección (`http://`) como en la columna de protocolo en la pestaña Network y en el puerto que usa para conectarse (puerto 80). Como no tiene SSL/TLS, toda la comunicación entre mi navegador y el servidor va sin cifrar.

### Datos de la primera solicitud

Al revisar la primera petición (`neverssl.com`) en DevTools se pueden ver estos datos:

- **URL solicitada:** `http://neverssl.com/`
- **Método HTTP:** `GET`
- **Host:** `neverssl.com`
- **Protocolo utilizado:** `http/1.1`
- **Dirección remota:** `34.223.124.45:80`
- **Estado:** `200 OK`

Y en los **Request Headers** (cabeceras que envió mi navegador):

```http
GET / HTTP/1.1
Host: neverssl.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: es-ES,es;q=0.9,en;q=0.8
```

### ¿Qué información queda visible?

Con solo mirar esa solicitud ya se pueden ver varios datos útiles sin necesidad de descifrar nada:
- **Host y URL:** dejan claro exactamente a qué página estoy entrando y qué recurso estoy pidiendo.
- **Método GET:** muestra el tipo de petición que está haciendo el navegador (en otros sitios HTTP, si se enviara un formulario con `POST`, también se verían los datos enviados).
- **Headers:** muestran configuraciones del navegador como el idioma (`Accept-Language`) y los formatos que acepta (`Accept`), y si el sitio usara cookies de sesión también aparecerían acá.
- **User-Agent:** revela qué sistema operativo estoy usando (en este caso Windows 10/11 de 64 bits) y qué navegador y versión tengo instalada.

## Riesgos encontrados

Navegar por HTTP estando conectado a un Wi-Fi público es bastante peligroso porque el tráfico viaja sin protección por una red que no controlamos. Un atacante conectado a la misma red podría:

- **Espiar el tráfico (sniffing):** usando herramientas de captura de paquetes (como Wireshark) puede leer en texto plano qué sitios visito, las URLs completas, los headers y el contenido de las páginas.
- **Robar credenciales y datos personales:** si ingreso un usuario, una contraseña, un correo o cualquier formulario en un sitio HTTP, esos datos se pueden capturar directamente.
- **Robar cookies de sesión:** si un sitio guarda la sesión en una cookie y viaja por HTTP, el atacante puede copiarla y entrar a mi cuenta sin siquiera necesitar la contraseña.
- **Hacer un ataque Man-in-the-Middle (MITM):** el atacante puede ponerse en el medio entre mi dispositivo y el router (o crear un Wi-Fi falso con un nombre parecido al del lugar) no solo para leer lo que envío, sino también para modificar la página que recibo e inyectar código malicioso o formularios falsos.
- **Conocer datos de mi equipo:** gracias al `User-Agent`, puede saber mi sistema operativo y versión del navegador para buscar fallas conocidas si tengo algo desactualizado.

## Cómo ayuda una VPN

Usar una **VPN (Red Privada Virtual)** cambia bastante la situación cuando estamos en un Wi-Fi público:

- **Cifrado:** antes de que los datos salgan de mi dispositivo, la VPN los cifra. Si alguien en el Wi-Fi intenta capturar mis paquetes, solo va a ver información ilegible en lugar de URLs, headers o el `User-Agent`.
- **Túnel seguro:** crea un túnel cifrado directo entre mi computadora o celular y el servidor de la VPN. Así el tramo más vulnerable, que es la red Wi-Fi pública, queda protegido.
- **Protección del tráfico:** no protege únicamente lo que abro en el navegador, sino todo el tráfico de red del equipo (aplicaciones, consultas DNS y conexiones que no usen HTTPS), evitando que alguien en la red local pueda espiar o alterar los paquetes.
- **Privacidad:** oculta mi dirección IP real usando la del servidor VPN y evita que el dueño o administrador de la red Wi-Fi pública vea qué páginas estoy visitando.

De todas formas, la VPN protege el camino hasta el servidor VPN, así que lo ideal sigue siendo usar VPN junto con sitios que tengan HTTPS.

## 3 Reglas de Oro

1. **Usar siempre una VPN en redes públicas:** activarla apenas me conecto al Wi-Fi abierto para que todo el tráfico salga cifrado desde el principio.
2. **No hacer trámites sensibles ni ingresar contraseñas en Wi-Fi abierto:** evitar entrar al home banking, correo principal o cuentas importantes desde redes públicas, y verificar siempre que los sitios usen `https://`. Si necesito hacer algo importante, prefiero usar los datos móviles del celular.
3. **Desactivar la conexión automática y el uso compartido:** no dejar el Wi-Fi en conexión automática para evitar caer en redes falsas, marcar la red como pública en el sistema operativo y mantener apagado el compartir archivos o dispositivos.
