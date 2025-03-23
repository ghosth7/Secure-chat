import os
import sys
import subprocess
import socket
import socks
import threading
import json
import base64
import hashlib
import time
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from stem.control import Controller

# Dependencias necesarias
DEPENDENCIAS = ["stem", "cryptography", "requests", "pycryptodome", "pysocks"]

def instalar_dependencias():
    """Detecta el sistema operativo e instala dependencias si faltan"""
    print("\n🔍 Verificando dependencias...")

    for paquete in DEPENDENCIAS:
        try:
            __import__(paquete)
        except ImportError:
            print(f"⚠️ Falta {paquete}, instalando...")
            subprocess.run(["pip", "install", paquete])

    # Instalar Tor si no está instalado (solo en Linux/Termux)
    if sys.platform.startswith("linux"):
        if not os.path.exists("/usr/bin/tor") and not os.path.exists("/data/data/com.termux/files/usr/bin/tor"):
            print("⚠️ Tor no está instalado. Instalándolo...")
            if "ANDROID_ROOT" in os.environ:  # Termux
                subprocess.run(["pkg", "install", "tor", "-y"])
            else:  # Ubuntu u otras distros Linux
                subprocess.run(["sudo", "apt", "install", "tor", "-y"])

    print("✅ Todas las dependencias están instaladas.\n")


# Configuración de Tor y Proxy
TOR_PROXY = ("127.0.0.1", 9050)
TOR_CONTROL_PORT = 9051
PORT = 5000
SESSION_KEY = hashlib.sha256(get_random_bytes(16)).digest()

# Contador de mensajes
mensaje_count = 0
CAMBIO_IP_CADA = 2  # 🔥 Cambia de IP cada 2 mensajes

def cifrar_mensaje(mensaje):
    """Cifra un mensaje con AES-256 en modo CBC"""
    cipher = AES.new(SESSION_KEY, AES.MODE_CBC)
    iv = cipher.iv
    ciphertext = cipher.encrypt(mensaje.ljust(32).encode())  # Padding
    return base64.b64encode(iv + ciphertext).decode()

def descifrar_mensaje(mensaje_cifrado):
    """Descifra un mensaje cifrado con AES-256 en modo CBC"""
    raw = base64.b64decode(mensaje_cifrado)
    iv = raw[:16]
    ciphertext = raw[16:]
    cipher = AES.new(SESSION_KEY, AES.MODE_CBC, iv)
    return cipher.decrypt(ciphertext).strip().decode()

def cambiar_ip_tor():
    """Cambia la IP de Tor enviando el comando NEWNYM al servicio de control"""
    try:
        with Controller.from_port(port=TOR_CONTROL_PORT) as controller:
            controller.authenticate()  # Conectar con Tor
            controller.signal("NEWNYM")  # Solicitar nueva identidad
            print("\n🔄 Cambio de IP solicitado a Tor...")
    except Exception as e:
        print(f"⚠️ Error al cambiar IP de Tor: {e}")

def servidor():
    """Inicia el servidor y espera conexiones"""
    global mensaje_count
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(("127.0.0.1", PORT))
    server.listen(1)
    print("Esperando conexión...")

    conn, addr = server.accept()
    print("Cliente conectado.")

    while True:
        try:
            data = conn.recv(1024).decode()
            if not data:
                break
            print(f"\nCliente: {descifrar_mensaje(data)}")
            
            # Contar mensajes y cambiar IP si es necesario
            mensaje_count += 1
            if mensaje_count % CAMBIO_IP_CADA == 0:
                cambiar_ip_tor()

            msg = input("Tú: ")
            conn.send(cifrar_mensaje(msg).encode())
        except:
            break

    conn.close()
    server.close()

def cliente(onion_address):
    """Función que actúa como cliente y se conecta al servidor"""
    global mensaje_count
    socks.set_default_proxy(socks.SOCKS5, *TOR_PROXY)
    client = socks.socksocket()
    client.connect((onion_address, PORT))
    print("Conectado al servidor.")

    def recibir():
        """Hilo para recibir mensajes"""
        while True:
            try:
                data = client.recv(1024).decode()
                if not data:
                    break
                print(f"\nServidor: {descifrar_mensaje(data)}")
            except:
                break

    threading.Thread(target=recibir, daemon=True).start()

    while True:
        msg = input("Tú: ")
        client.send(cifrar_mensaje(msg).encode())
        
        # Contar mensajes y cambiar IP si es necesario
        mensaje_count += 1
        if mensaje_count % CAMBIO_IP_CADA == 0:
            cambiar_ip_tor()

def main():
    instalar_dependencias()
    print("🔒 Chat Seguro con Tor y Cifrado AES 🔒")
    username = input("Elige un nombre de usuario único para esta sesión: ")

    print("\n¿Quieres ser el Host o el Cliente?")
    print("[H] Host (crear chat)")
    print("[C] Cliente (conectarse)")
    opcion = input("Selecciona (H/C): ").strip().upper()

    if opcion == "H":
        servidor()
    
    elif opcion == "C":
        onion_address = input("\nIngresa la dirección .onion del host: ")
        cliente(onion_address)
    
    else:
        print("Opción no válida. Saliendo.")

if __name__ == "__main__":
    main()
