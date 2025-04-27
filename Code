import pandas as pd
from datetime import datetime
import numpy as np

class Bono:
    """
    Representa un bono financiero.

    Atributos:
        fecha_liquidacion (datetime): Fecha de liquidación del bono.
        fecha_vencimiento (datetime): Fecha de vencimiento del bono.
        tir (float): Tasa interna de retorno.
        cupon (float): Valor del cupón.
        valor_par (float): Valor par del bono.
    """

    def __init__(self, fecha_liquidacion: str, fecha_vencimiento: str,
                 tir: float, cupon: float, valor_par: float):
        """
        Inicializa un bono con sus atributos básicos.
        """
        self.fecha_liquidacion = datetime.strptime(fecha_liquidacion, "%d-%m-%Y")
        self.fecha_vencimiento = datetime.strptime(fecha_vencimiento, "%d-%m-%Y")
        self.tir = tir
        self.cupon = cupon
        self.valor_par = valor_par

    def generar_fechas_pago(self) -> list:
        """
        Genera las fechas de pago del bono.
        """
        fechas_pago = []
        fecha_actual = self.fecha_liquidacion.replace(
            day=self.fecha_vencimiento.day, month=self.fecha_vencimiento.month)

        if fecha_actual < self.fecha_liquidacion:
            fecha_actual += pd.DateOffset(years=1)

        while fecha_actual <= self.fecha_vencimiento:
            fechas_pago.append(fecha_actual)
            fecha_actual += pd.DateOffset(years=1)

        return fechas_pago

    def contar_dias_desde_liquidacion(self, fecha_pago, fecha_anterior) -> int:
        """
        Cuenta los días entre dos fechas.
        """
        return (fecha_pago - fecha_anterior).days

    @staticmethod
    def es_bisiesto(año: int) -> bool:
        """
        Determina si un año es bisiesto.
        """
        return (año % 4 == 0 and año % 100 != 0) or año % 400 == 0

    def evaluar_año_bisiesto(self, fecha) -> int:
        """
        Evalúa si el año de una fecha es bisiesto.
        """
        return 366 if self.es_bisiesto(fecha.year) else 365

    def calcular_factor_descuento(self, division_acumulada: float) -> float:
        """
        Calcula el factor de descuento basado en la TIR.
        """
        return 1 / (1 + self.tir) ** division_acumulada

    def generar_dataframe(self) -> pd.DataFrame:
        """
        Genera un DataFrame con información relevante del bono.

        Returns:
            pd.DataFrame: DataFrame con las fechas de pago, días desde la liquidación,
                          base, división acumulada y factor de descuento.
        """
        fechas_pago = self.generar_fechas_pago()

        # Calcular días desde la liquidación
        dias_desde_liquidacion = [self.contar_dias_desde_liquidacion(fechas_pago[0], self.fecha_liquidacion)]
        dias_desde_liquidacion.extend(
            self.contar_dias_desde_liquidacion(fechas_pago[i], fechas_pago[i-1])
            for i in range(1, len(fechas_pago))
        )

        bases = [self.evaluar_año_bisiesto(fecha) for fecha in fechas_pago]

        # Calcular la división y acumulación
        division_acumulada = []
        acumulado = 0
        for dias, base in zip(dias_desde_liquidacion, bases):
            acumulado += dias / base
            division_acumulada.append(acumulado)

        factores_descuento = [self.calcular_factor_descuento(div) for div in division_acumulada]

        data = {
            'Fecha Pago Cupón': [fecha.strftime('%d-%m-%Y') for fecha in fechas_pago],
            'Días desde Liquidación': dias_desde_liquidacion,
            'Base': bases,
            'División Acumulada': division_acumulada,
            'Factor de Descuento': factores_descuento
        }

        return pd.DataFrame(data)

    def flujos_descontados(self) -> list:
        """
        Calcula los flujos descontados del bono.
        """
        factores_descuento = self.generar_dataframe()['Factor de Descuento'].tolist()
        flujos = [factor * self.cupon for factor in factores_descuento]
        flujos[-1] += factores_descuento[-1] * self.valor_par
        return flujos

    def precio_sucio(self) -> float:
        """
        Calcula el precio sucio del bono.
        """
        return sum(self.flujos_descontados())

    def fecha_pago_anterior(self):
        """
        Obtiene la fecha de pago anterior a la fecha de liquidación.
        """
        fecha_anterior = self.fecha_liquidacion.replace(
            day=self.fecha_vencimiento.day, month=self.fecha_vencimiento.month
        )
        if fecha_anterior > self.fecha_liquidacion:
            fecha_anterior -= pd.DateOffset(years=1)
        return fecha_anterior

    def intereses_devengados(self) -> float:
        """
        Calcula los intereses devengados del bono.
        """
        fecha_anterior = self.fecha_pago_anterior()
        primera_fecha_pago = self.generar_fechas_pago()[0]
        dias_entre_fechas = (self.fecha_liquidacion - fecha_anterior).days
        años = list(range(fecha_anterior.year, primera_fecha_pago.year + 1))
        base = 366 if any(self.es_bisiesto(año) for año in años) else 365
        return self.cupon * (dias_entre_fechas / base)

    def calcular_TIR(self, precio_sucio_actual: float, estimado_inicial: float = 0.05,
                     max_iteraciones: int = 1000, tolerancia: float = 1e-6) -> float:
        """
        Calcula la TIR del bono usando el método de Newton.
        """
        def objetivo(tasa):
            self.tir = tasa
            return self.precio_sucio() - precio_sucio_actual

        tasa_estimada = estimado_inicial
        for _ in range(max_iteraciones):
            f_val = objetivo(tasa_estimada)
            if abs(f_val) < tolerancia:
                return tasa_estimada
            f_prime = (objetivo(tasa_estimada + 1e-5) - f_val) / 1e-5
            tasa_estimada -= f_val / f_prime

        raise ValueError("No se pudo encontrar la TIR después de las máximas iteraciones.")

    def duracion_macaulay(self) -> float:
        """
        Calcula la duración de Macaulay del bono utilizando el método Actual/actual.
        """
        flujos = self.flujos_descontados()
        fechas_pago = self.generar_fechas_pago()

        # Calcular años desde la liquidación para cada fecha de pago
        años_desde_liquidacion = []
        for fecha in fechas_pago:
            start = self.fecha_liquidacion
            end = fecha
            total_years = 0

            while start < end:
                next_year_start = datetime(start.year + 1, 1, 1)
                if next_year_start > end:
                    total_years += (end - start).days / self.evaluar_año_bisiesto(start)
                else:
                    total_years += (next_year_start - start).days / self.evaluar_año_bisiesto(start)
                start = next_year_start

            años_desde_liquidacion.append(total_years)

        duracion = sum([fd * año for fd, año in zip(flujos, años_desde_liquidacion)])
        return duracion / self.precio_sucio()



    def duracion_modificada(self) -> float:
        """
        Calcula la duración modificada del bono.
        """
        macaulay_duration = self.duracion_macaulay()
        return macaulay_duration / (1 + self.tir)




###################### Uso ######################
bono = Bono("05-09-2024", "18-09-2030", 0.09596, 7.75, 100)
df = bono.generar_dataframe()
print(df)
print("\nFlujos Descontados:", bono.flujos_descontados())
print("Precio Sucio:", bono.precio_sucio())
