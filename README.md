# Sistema-Software-FJ
"""Software FJ - Sistema Integral de Clientes, Servicios y Reservas.
Proyecto académico: Programación Orientada a Objetos y manejo de excepciones.
"""
from abc import ABC, abstractmethod
from datetime import datetime
import logging

# Registro de eventos y errores en un archivo de log.
logging.basicConfig(
    filename="software_fj.log",
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    encoding="utf-8",
)


# Excepciones personalizadas para controlar errores del sistema.
class ErrorSistema(Exception):
    """Excepción base para errores controlados del sistema."""


class DatosInvalidosError(ErrorSistema):
    pass


class ReservaError(ErrorSistema):
    pass


# Clase abstracta que representa entidades generales del sistema.
class Entidad(ABC):
    def __init__(self, identificador):
        if not identificador or not str(identificador).strip():
            raise DatosInvalidosError("El identificador no puede estar vacío.")
        self.identificador = str(identificador).strip()

    @abstractmethod
    def describir(self):
        """Cada entidad debe implementar una descripción."""
        pass


class Cliente(Entidad):
    def __init__(self, identificador, nombre, correo, telefono):
        super().__init__(identificador)
        self.__nombre = self._validar_texto(nombre, "nombre")
        self.__correo = self._validar_correo(correo)
        self.__telefono = self._validar_texto(telefono, "teléfono")

    @staticmethod
    def _validar_texto(valor, campo):
        if not isinstance(valor, str) or not valor.strip():
            raise DatosInvalidosError(f"El {campo} no puede estar vacío.")
        return valor.strip()

    @staticmethod
    def _validar_correo(correo):
        if not isinstance(correo, str) or "@" not in correo or "." not in correo.split("@")[-1]:
            raise DatosInvalidosError("El correo electrónico no tiene un formato válido.")
        return correo.strip()

    @property
    def nombre(self):
        return self.__nombre

    @property
    def correo(self):
        return self.__correo

    @property
    def telefono(self):
        return self.__telefono

    def describir(self):
        return f"Cliente: {self.nombre} | Correo: {self.correo} | Teléfono: {self.telefono}"


# Clase abstracta de servicios; los servicios concretos calculan sus propios precios.
class Servicio(Entidad, ABC):
    def __init__(self, identificador, nombre, tarifa):
        super().__init__(identificador)
        if not isinstance(tarifa, (int, float)) or tarifa <= 0:
            raise DatosInvalidosError("La tarifa debe ser un número positivo.")
        self.nombre = nombre.strip() if isinstance(nombre, str) and nombre.strip() else "Servicio"
        self.tarifa = float(tarifa)

    @abstractmethod
    def calcular_costo(self, duracion, descuento=0, impuesto=0):
        pass

    @abstractmethod
    def validar_parametros(self, duracion):
        pass


class ReservaSala(Servicio):
    def validar_parametros(self, duracion):
        if not isinstance(duracion, (int, float)) or duracion <= 0:
            raise DatosInvalidosError("La duración de la reserva de sala debe ser mayor que cero.")

    def calcular_costo(self, duracion, descuento=0, impuesto=0):
        self.validar_parametros(duracion)
        return calcular_total(self.tarifa * duracion, descuento, impuesto)

    def describir(self):
        return f"Reserva de sala: {self.nombre} (tarifa por hora: ${self.tarifa:,.0f})"


class AlquilerEquipo(Servicio):
    def validar_parametros(self, duracion):
        if not isinstance(duracion, (int, float)) or duracion <= 0:
            raise DatosInvalidosError("La duración del alquiler debe ser mayor que cero.")

    def calcular_costo(self, duracion, descuento=0, impuesto=0):
        self.validar_parametros(duracion)
        return calcular_total(self.tarifa * duracion, descuento, impuesto)

    def describir(self):
        return f"Alquiler de equipo: {self.nombre} (tarifa por unidad de tiempo: ${self.tarifa:,.0f})"


class Asesoria(Servicio):
    def validar_parametros(self, duracion):
        if not isinstance(duracion, (int, float)) or duracion <= 0:
            raise DatosInvalidosError("La duración de la asesoría debe ser mayor que cero.")

    def calcular_costo(self, duracion, descuento=0, impuesto=0):
        self.validar_parametros(duracion)
        return calcular_total(self.tarifa * duracion, descuento, impuesto)

    def describir(self):
        return f"Asesoría especializada: {self.nombre} (tarifa por hora: ${self.tarifa:,.0f})"


# Simulación de sobrecarga: el mismo método admite diferentes parámetros opcionales.
def calcular_total(base, descuento=0, impuesto=0):
    """Calcula costo con opción de descuento e impuesto (parámetros opcionales)."""
    if not isinstance(base, (int, float)) or base < 0:
        raise DatosInvalidosError("El valor base no puede ser negativo.")
    if not isinstance(descuento, (int, float)) or not 0 <= descuento <= 100:
        raise DatosInvalidosError("El descuento debe estar entre 0 y 100%.")
    if not isinstance(impuesto, (int, float)) or not 0 <= impuesto <= 100:
        raise DatosInvalidosError("El impuesto debe estar entre 0 y 100%.")
    valor_descuento = base * descuento / 100
    subtotal = base - valor_descuento
    return round(subtotal + subtotal * impuesto / 100, 2)


class Reserva:
    ESTADOS = ("Pendiente", "Confirmada", "Cancelada")

    def __init__(self, identificador, cliente, servicio, duracion, descuento=0, impuesto=0):
        if not isinstance(cliente, Cliente):
            raise DatosInvalidosError("Debe asociarse una instancia válida de Cliente.")
        if not isinstance(servicio, Servicio):
            raise DatosInvalidosError("Debe asociarse una instancia válida de Servicio.")
        servicio.validar_parametros(duracion)
        self.identificador = str(identificador)
        self.cliente = cliente
        self.servicio = servicio
        self.duracion = duracion
        self.descuento = descuento
        self.impuesto = impuesto
        self.estado = "Pendiente"
        self.fecha = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    def confirmar(self):
        if self.estado != "Pendiente":
            raise ReservaError("Solo se puede confirmar una reserva pendiente.")
        self.estado = "Confirmada"
        logging.info("Reserva %s confirmada.", self.identificador)

    def cancelar(self):
        if self.estado == "Cancelada":
            raise ReservaError("La reserva ya está cancelada.")
        self.estado = "Cancelada"
        logging.info("Reserva %s cancelada.", self.identificador)

    def procesar(self):
        try:
            if self.estado != "Confirmada":
                raise ReservaError("La reserva debe estar confirmada para procesarse.")
            total = self.servicio.calcular_costo(self.duracion, self.descuento, self.impuesto)
        except ErrorSistema as error:
            logging.exception("Error al procesar reserva %s", self.identificador)
            raise ReservaError("No fue posible procesar la reserva.") from error
        else:
            logging.info("Reserva %s procesada. Total: %.2f", self.identificador, total)
            return total
        finally:
            logging.info("Finalizó intento de procesamiento de reserva %s.", self.identificador)

    def describir(self):
        return (f"Reserva #{self.identificador} | {self.cliente.nombre} | {self.servicio.nombre} | "
                f"Duración: {self.duracion} | Estado: {self.estado} | Fecha: {self.fecha}")


class SistemaFJ:
    def __init__(self):
        self.clientes = []
        self.servicios = []
        self.reservas = []

    def registrar_cliente(self, cliente):
        if any(c.identificador == cliente.identificador for c in self.clientes):
            raise DatosInvalidosError("Ya existe un cliente con ese identificador.")
        self.clientes.append(cliente)
        logging.info("Cliente registrado: %s", cliente.identificador)

    def registrar_servicio(self, servicio):
        if any(s.identificador == servicio.identificador for s in self.servicios):
            raise DatosInvalidosError("Ya existe un servicio con ese identificador.")
        self.servicios.append(servicio)
        logging.info("Servicio registrado: %s", servicio.identificador)

    def crear_reserva(self, reserva):
        if any(r.identificador == reserva.identificador for r in self.reservas):
            raise ReservaError("Ya existe una reserva con ese identificador.")
        self.reservas.append(reserva)
        logging.info("Reserva creada: %s", reserva.identificador)

    def mostrar_resumen(self):
        print("\n--- CLIENTES ---")
        for cliente in self.clientes:
            print(cliente.describir())
        print("\n--- SERVICIOS ---")
        for servicio in self.servicios:
            print(servicio.describir())
        print("\n--- RESERVAS ---")
        for reserva in self.reservas:
            print(reserva.describir())


def ejecutar_operacion(numero, descripcion, funcion):
    """Ejecuta una operación de prueba y registra errores sin detener el programa."""
    print(f"\nOperación {numero}: {descripcion}")
    try:
        resultado = funcion()
    except Exception as error:
        logging.exception("Operación %s falló: %s", numero, descripcion)
        print(f"Error controlado: {error}")
    else:
        if resultado is not None:
            print(resultado)
        print("Operación ejecutada correctamente.")
    finally:
        print("Fin de la operación.")


def main():
    sistema = SistemaFJ()
    print("=== SOFTWARE FJ: GESTIÓN DE CLIENTES, SERVICIOS Y RESERVAS ===")

    # Operación 1: registro de cliente válido.
    cliente1 = Cliente("C001", "Ana Pérez", "ana@email.com", "3001112233")
    ejecutar_operacion(1, "Registrar cliente válido", lambda: sistema.registrar_cliente(cliente1))

    # Operación 2: cliente inválido (correo incorrecto).
    ejecutar_operacion(2, "Intentar registrar cliente con correo inválido",
                       lambda: Cliente("C002", "Luis", "correo-invalido", "3002223344"))

    # Operación 3: registrar un segundo cliente válido.
    cliente2 = Cliente("C003", "Carlos Gómez", "carlos@email.com", "3105556677")
    ejecutar_operacion(3, "Registrar segundo cliente", lambda: sistema.registrar_cliente(cliente2))

    # Operación 4: servicio de sala.
    sala = ReservaSala("S001", "Sala de reuniones", 50000)
    ejecutar_operacion(4, "Crear y registrar servicio de sala", lambda: sistema.registrar_servicio(sala))

    # Operación 5: alquiler de equipo.
    equipo = AlquilerEquipo("S002", "Proyector", 30000)
    ejecutar_operacion(5, "Crear y registrar alquiler de equipo", lambda: sistema.registrar_servicio(equipo))

    # Operación 6: asesoría especializada.
    asesoria = Asesoria("S003", "Asesoría en software", 80000)
    ejecutar_operacion(6, "Crear y registrar asesoría", lambda: sistema.registrar_servicio(asesoria))

    # Operación 7: servicio inválido, tarifa negativa.
    ejecutar_operacion(7, "Intentar crear servicio con tarifa inválida",
                       lambda: ReservaSala("S004", "Sala inválida", -10))

    # Operación 8: reserva válida, confirmación y procesamiento.
    reserva1 = Reserva("R001", cliente1, sala, 2, descuento=10, impuesto=19)
    ejecutar_operacion(8, "Crear, confirmar y procesar reserva válida",
                       lambda: (sistema.crear_reserva(reserva1), reserva1.confirmar(),
                                f"Total a pagar: ${reserva1.procesar():,.2f}")[2])

    # Operación 9: reserva con duración inválida.
    ejecutar_operacion(9, "Intentar crear reserva con duración cero",
                       lambda: Reserva("R002", cliente2, equipo, 0))

    # Operación 10: reserva válida cancelada; intentar procesarla genera error controlado.
    reserva2 = Reserva("R003", cliente2, asesoria, 1)
    def crear_cancelar_y_procesar():
        sistema.crear_reserva(reserva2)
        reserva2.cancelar()
        return reserva2.procesar()
    ejecutar_operacion(10, "Crear, cancelar y tratar de procesar reserva cancelada", crear_cancelar_y_procesar)

    sistema.mostrar_resumen()
    print("\nSe completó la simulación. Revise software_fj.log para consultar eventos y errores.")
    logging.info("Ejecución del programa finalizada.")


if __name__ == "__main__":
    main()
