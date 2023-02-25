# IOT ESP server

## Servidor web usando ESP32/ESP8266

Usando la placa ESP32/ESP8266 en la carpeta del dispositivo. De forma predeterminada, cuando se graba el firmware de MicroPython,  se crea un archivo [boot.py](https://github.com/ericksc/esp_wifi_server/blob/main/boot.py).

Para este proyecto necesitarás un archivo [boot.py](https://github.com/ericksc/esp_wifi_server/blob/main/boot.py) y un archivo [main.py](https://github.com/ericksc/esp_wifi_server/blob/main/main.py). El  archivo boot.py tiene el código que solo necesita ejecutarse una vez en el arranque. Esto incluye la importación de bibliotecas, credenciales de red, creación de instancias de pines, conexión a la red y otras configuraciones.

El archivo [main.py](https://github.com/ericksc/esp_wifi_server/blob/main/main.py) contendrá el código que ejecuta el servidor web para servir archivos y realizar tareas basadas en las solicitudes recibidas por el cliente.


Nuestro servidor web utilizando sockets y la API de socket de Python. La documentación oficial importa la biblioteca de sockets de la siguiente manera

### Detalle de [boot.py](https://github.com/ericksc/esp_wifi_server/blob/main/boot.py)

```python
try:
  import usocket as socket
except:
  import socket
Necesitamos importar la  clase Pin desde el módulo de la máquina para poder interactuar con los GPIO. 
from machine import Pin
```
 
Después de importar la biblioteca de sockets, necesitamos importar la biblioteca de red. La  biblioteca de red nos permite conectar el ESP32 o ESP8266 a una red Wi-Fi.

```Python
import network
```

Las siguientes líneas desactivan los mensajes de depuración del sistema operativo del proveedor:
```python
import esp
esp.osdebug(None)
```

Luego, ejecutamos un recolector de basura:
```python
import gc
gc.collect()
```

Un recolector de elementos no utilizados es una forma de administración automática de memoria. Esta es una forma de recuperar la memoria ocupada por objetos que ya no son utilizados por el programa. Esto es útil para ahorrar espacio en la memoria flash.
Las siguientes variables contienen sus credenciales de red:

```python
ssid = 'REPLACE_WITH_YOUR_SSID'
password = 'replace_with_your_password'
```

Debe reemplazar las palabras resaltadas en rojo con su SSID y contraseña de red, para que el ESP pueda conectarse a su enrutador.
Luego, configure el ESP32 o ESP8266 como una estación Wi-Fi:

```python
station = network.WLAN(network.STA_IF)
```

Después de eso, active la estación:

```python
station.active(True)
```
Finalmente, el ESP32/ESP8266 se conecta a su router utilizando el SSID y la contraseña definidos anteriormente:

```python
station.connect(ssid, password)
```

La siguiente instrucción garantiza que el código no continúe mientras el ESP no esté conectado a la red.

```python
while station.isconnected() == False:
  pass
```

Después de una conexión exitosa, imprima parámetros de interfaz de red como la dirección IP ESP32/ESP8266: use el método ifconfig() en el  objeto de estación.

```python
print('Connection successful')
print(station.ifconfig())
```

Cree un objeto Pin llamado led que  es una salida, que hace referencia al ESP32/ESP8266 GPIO2:

```python
led = Pin(2, Pin.OUT)
```

### [main.py](https://github.com/ericksc/esp_wifi_server/blob/main/main.py)


El script comienza creando una función llamada web_page(). Esta función devuelve una variable llamada html que contiene el texto HTML para construir la página web.
 
```python
def web_page():
La página web muestra el estado actual de GPIO. Por lo tanto, antes de generar el texto HTML, debemos verificar el estado del LED. Guardamos su estado en la variable gpio_state:
if led.value() == 1:
  gpio_state="ON"
else:
  gpio_state="OFF"
```

Después de eso, la variable gpio_state se incorpora al texto HTML usando signos "+" para concatenar cadenas.

```python
html = """<html><head> <title>ESP Web Server</title> <meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="icon" href="data:,"> <style>html{font-family: Helvetica; display:inline-block; margin: 0px auto; text-align: center;}
h1{color: #0F3376; padding: 2vh;}p{font-size: 1.5rem;}.button{display: inline-block; background-color: #e7bd3b; border: none; 
border-radius: 4px; color: white; padding: 16px 40px; text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}
.button2{background-color: #4286f4;}</style></head><body> <h1>ESP Web Server</h1> 
<p>GPIO state: <strong>""" + gpio_state + """</strong></p><p><a href="/?led=on"><button class="button">ON</button></a></p>
<p><a href="/?led=off"><button class="button button2">OFF</button></a></p></body></html>"""
```

Creación de un servidor de socket
Después de crear el HTML para construir la página web, necesitamos crear un socket de escucha para escuchar las solicitudes entrantes y enviar el texto HTML en respuesta. Para una mejor comprensión, la siguiente figura muestra un diagrama sobre cómo crear sockets para la interacción servidor-cliente:
 
 
Cree un socket usando socket.socket(), y especifique el tipo de socket. Creamos un nuevo objeto socket llamado s con la familia de direcciones dada y el tipo de socket. Este es un socket TCP STREAM:

```python
s = zócalo. zócalo (zócalo. AF_INET, zócalo. SOCK_STREAM)
```

A continuación, enlace el socket a una dirección (interfaz de red y número de puerto) utilizando el método bind(). El  método bind() acepta una variable tupple con la dirección IP y el número de puerto:

```python
s.bind(('', 80))
```

En nuestro ejemplo, estamos pasando una cadena vacía ' ' como dirección IP y puerto 80. En este caso, la cadena vacía hace referencia a la dirección IP del host local (es decir, la dirección IP ESP32 o ESP8266).
La siguiente línea permite que el servidor acepte conexiones; Hace un zócalo de "escucha". El argumento especifica el número máximo de conexiones en cola. El máximo es 5.

```python
s.listen(5)
```

En el bucle while es donde escuchamos las solicitudes y enviamos las respuestas. Cuando un cliente se conecta, el servidor llama al método accept() para aceptar la conexión. Cuando un cliente se conecta, guarda un nuevo objeto socket para aceptar y enviar datos sobre  la variable conn, y guarda la dirección del cliente para conectarse al servidor en la variable addr.

```python
conn, addr = s. aceptar()
```

A continuación, imprima la dirección del cliente guardado en la variable addr.

```python
print('Tengo una conexión de %s' %  str(addr))
```

Los datos se intercambian entre el cliente y el servidor utilizando los  métodos send() y recv().
La siguiente línea obtiene la solicitud recibida en el socket recién creado y la guarda en la variable request.

```python
request = conn.recv(1024)
```

El método recv() recibe los datos del socket del cliente (recuerde que hemos creado un nuevo objeto socket en la  variable conn). El argumento del  método recv() especifica el máximo de datos que se pueden recibir a la vez.
La siguiente línea simplemente imprime el contenido de la solicitud:

```python
print('Content = %s' % str(request))
```

A continuación, cree una variable llamada response que contenga el texto HTML devuelto por la  función web_page():

```python
response = web_page()
```

Finalmente, envíe la respuesta al cliente socket utilizando los  métodos send() y sendall():

```python
conn.send('HTTP/1.1 200 OK\n')
conn.send('Content-Type: text/html\n')
conn.send('Connection: close\n\n')
conn.sendall(response)
```

Al final, cierre el socket creado.

```python
conn.close()
```

## Prueba del servidor web
Cargue los archivos main.py y boot.py en el ESP32/ESP8266. La  carpeta del dispositivo debe contener dos archivos: boot.py y main.py.
Después de cargar los archivos, pulse el botón integrado ESP EN/RST.
 
 
Después de unos segundos, debería establecer una conexión con su enrutador e imprimir la dirección IP en el Shell.
 
Abra su navegador y escriba su dirección IP ESP que acaba de encontrar. Debería ver la página del servidor web como se muestra a continuación.
 
Cuando presiona el botón ON, realiza una solicitud en la dirección IP ESP seguida de /?led=on. El LED integrado ESP32/ESP8266 se enciende y el estado de GPIO se actualiza en la página.
Nota: algunos LED integrados ESP8266 encienden el LED con un comando OFF y apagan el LED con el comando ON.
 
 
Cuando presiona el botón OFF, realiza una solicitud en la dirección IP ESP seguida de /?led=off. El LED se apaga y se actualiza el estado de GPIO.
 
 
Nota: Se esta controlando el LED integrado que corresponde a GPIO 2. Puede controlar cualquier otro GPIO con cualquier otra salida (un relé, por ejemplo) utilizando el mismo método. Además, puede modificar el código para controlar varios GPIO o cambiar el texto HTML para crear una página web diferente.
Terminando
Este tutorial le mostró cómo crear un servidor web simple con firmware MicroPython para controlar los GPIO ESP32/ESP8266 utilizando sockets y la biblioteca de sockets de Python


