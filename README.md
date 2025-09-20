# TP.Integradorentrega1
import datetime

class Mensaje:
    def __init__(self, remitente, destinatario, asunto, cuerpo):
        self._remitente = remitente
        self._destinatario = destinatario
        self._asunto = asunto
        self._cuerpo = cuerpo
        self._fecha = datetime.datetime.now()
        self._leido = False

    def marcar_como_leido(self):
        self._leido = True

    @property
    def remitente(self):
        return self._remitente

    @property
    def asunto(self):
        return self._asunto

    @property
    def cuerpo(self):
        return self._cuerpo

    @property
    def fecha(self):
        return self._fecha

    @property
    def leido(self):
        return self._leido

    def __str__(self):
        return f"De: {self._remitente}\nAsunto: {self._asunto}\nFecha: {self._fecha.strftime('%Y-%m-%d %H:%M:%S')}"

CARPETA:
class Carpeta:
    def __init__(self, nombre):
        self._nombre = nombre
        self._mensajes = []

    def agregar_mensaje(self, mensaje):
        self._mensajes.append(mensaje)
        
    def eliminar_mensaje(self, mensaje):
        try:
            self._mensajes.remove(mensaje)
            return True
        except ValueError:
            return False

    def listar_mensajes(self):
        return self._mensajes

    @property
    def nombre(self):
        return self._nombre

    def __len__(self):
        return len(self._mensajes)

USUARIO:
class Usuario:
    def __init__(self, email, nombre, contrasena):
        self._email = email
        self._nombre = nombre
        self._contrasena = contrasena # Idealmente, se usaría un hash para la contraseña
        self._bandeja_entrada = Carpeta("Bandeja de Entrada")
        self._bandeja_salida = Carpeta("Enviados")

    def enviar_mensaje(self, servidor, destinatario_email, asunto, cuerpo):
        if servidor.enviar_mensaje(self, destinatario_email, asunto, cuerpo):
            print(f"Mensaje enviado a {destinatario_email} correctamente.")
            return True
        else:
            print(f"Error: no se pudo enviar el mensaje a {destinatario_email}.")
            return False

    def recibir_mensaje(self, mensaje):
        self._bandeja_entrada.agregar_mensaje(mensaje)

    def listar_bandeja_entrada(self):
        return self._bandeja_entrada.listar_mensajes()
    
    def listar_bandeja_salida(self):
        return self._bandeja_salida.listar_mensajes()

    def get_bandeja_entrada(self):
        return self._bandeja_entrada

    def get_bandeja_salida(self):
        return self._bandeja_salida
        
    @property
    def email(self):
        return self._email

    def __str__(self):
        return f"Usuario: {self._nombre} <{self._email}>"

SERVIDOR CORREO
class ServidorCorreo:
    def __init__(self):
        self._usuarios = {}
        self._mensajes_pendientes = []

    def registrar_usuario(self, usuario):
        if usuario.email in self._usuarios:
            print(f"Error: El usuario {usuario.email} ya existe.")
            return False
        self._usuarios[usuario.email] = usuario
        print(f"Usuario {usuario.email} registrado con éxito.")
        return True

    def enviar_mensaje(self, remitente, destinatario_email, asunto, cuerpo):
        destinatario = self._usuarios.get(destinatario_email)
        if not destinatario:
            print(f"Error: Destinatario {destinatario_email} no encontrado.")
            return False

        mensaje = Mensaje(remitente.email, destinatario_email, asunto, cuerpo)
        self._mensajes_pendientes.append((destinatario, mensaje))
        remitente.get_bandeja_salida().agregar_mensaje(mensaje)
        return True

    def entregar_mensajes_pendientes(self):
        if not self._mensajes_pendientes:
            print("No hay mensajes pendientes para entregar.")
            return

        print(f"Entregando {len(self._mensajes_pendientes)} mensajes pendientes...")
        for destinatario, mensaje in self._mensajes_pendientes:
            destinatario.recibir_mensaje(mensaje)
        
        self._mensajes_pendientes = []
        print("Entrega de mensajes completada.")

SIMULAR CLIENTE:
# 1. Crear el servidor
servidor = ServidorCorreo()

# 2. Crear y registrar usuarios
usuario1 = Usuario("ana@correo.com", "Ana García", "pass123")
usuario2 = Usuario("juan@correo.com", "Juan Pérez", "pass456")

servidor.registrar_usuario(usuario1)
servidor.registrar_usuario(usuario2)

# 3. Ana envía un mensaje a Juan
usuario1.enviar_mensaje(servidor, "juan@correo.com", "Hola, Juan", "Este es mi primer correo en el nuevo sistema.")

# 4. El servidor entrega los mensajes
servidor.entregar_mensajes_pendientes()

# 5. Juan revisa su bandeja de entrada
mensajes_juan = usuario2.listar_bandeja_entrada()
print("\n--- Mensajes en la bandeja de entrada de Juan ---")
for msg in mensajes_juan:
    print(msg)

# 6. Ana revisa su bandeja de salida
mensajes_ana = usuario1.listar_bandeja_salida()
print("\n--- Mensajes en la bandeja de enviados de Ana ---")
for msg in mensajes_ana:
    print(msg)
    
