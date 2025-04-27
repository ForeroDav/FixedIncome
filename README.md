# FixedIncome - Bono
Aplicación de la Lógica: Clase Bono en Python
Para abordar el desafío de la valoración de bonos y el análisis de sus métricas clave, se desarrolló una clase orientada a objetos en Python llamada Bono. Esta clase encapsula los atributos y comportamientos fundamentales de un bono de cupón fijo, permitiendo realizar cálculos financieros complejos de manera estructurada y reutilizable.

1. Inicialización y Atributos (__init__)

La clase se inicializa con los datos esenciales del bono:

fecha_liquidacion: La fecha en que se realiza la transacción (formato "DD-MM-YYYY").
fecha_vencimiento: La fecha en que el bono madura y devuelve el principal (formato "DD-MM-YYYY").
tir (Tasa Interna de Retorno / Yield to Maturity): La tasa de descuento utilizada para valorar los flujos futuros.
cupon: El monto del pago de interés periódico (en este caso, se asume anual).
valor_par: El valor nominal o principal del bono que se devuelve al vencimiento.
Internamente, las fechas se convierten a objetos datetime para facilitar los cálculos.

2. Generación de Fechas de Pago (generar_fechas_pago)

Este método determina las fechas futuras en las que el bono pagará sus cupones. La lógica implementada asume pagos anuales, calculando la primera fecha de pago como el aniversario de la fecha de vencimiento después de la fecha de liquidación, y continuando anualmente hasta la fecha de vencimiento inclusive. Utiliza pandas.DateOffset para manejar correctamente los años bisiestos al sumar años.

3. Cálculos Fundamentales (DataFrame, Precios)

generar_dataframe(): Este es uno de los métodos centrales. Calcula y organiza en un DataFrame de pandas toda la información relevante para la valoración:
Calcula los días entre fechas de pago consecutivas (y desde la liquidación hasta el primer pago).
Determina la base de días del año (365 o 366) para cada periodo, crucial para la convención Actual/Actual.
Calcula los años fraccionarios acumulados desde la liquidación hasta cada fecha de pago.
Calcula el factor de descuento para cada fecha de pago usando la tir y los años fraccionarios.
Define el flujo de caja para cada fecha (cupón, y cupón + principal en el vencimiento).
Calcula el flujo descontado (valor presente) de cada flujo.
precio_sucio(): Simplemente suma todos los flujos descontados calculados en generar_dataframe() para obtener el valor presente total del bono, incluyendo los intereses acumulados.
fecha_pago_anterior(): Calcula la fecha teórica del pago de cupón inmediatamente anterior a la fecha de liquidación.
intereses_devengados(): Calcula el "cupón corrido", es decir, la porción del cupón que se ha acumulado entre la última fecha de pago y la fecha de liquidación, usando la convención Actual/Actual (días reales transcurridos / días reales en el periodo de cupón).
Precio Limpio: Aunque no hay un método dedicado, se calcula trivialmente como precio_sucio() - intereses_devengados().
4. Métricas de Sensibilidad (Duración)

duracion_macaulay(): Calcula la duración de Macaulay, que representa el promedio ponderado del tiempo (en años) hasta recibir cada flujo de caja, donde la ponderación es el valor presente de cada flujo. Es una medida de la vida promedio ponderada del bono y se calcula usando los años fraccionarios acumulados y los flujos descontados del DataFrame.
duracion_modificada(): Deriva de la duración de Macaulay y mide la sensibilidad porcentual del precio del bono ante un cambio del 1% en la Tasa Interna de Retorno (TIR). Se calcula como Duración Macaulay / (1 + TIR / k), donde 'k' es la frecuencia de pago anual (en este caso, k=1).
5. Cálculo Inverso (TIR)

calcular_TIR(precio_limpio_objetivo): Este método permite encontrar la Tasa Interna de Retorno (Yield to Maturity) que iguala el precio calculado del bono a un precio_limpio_objetivo dado (por ejemplo, un precio observado en el mercado). Utiliza el método numérico de Newton-Raphson para encontrar iterativamente la tasa que hace que la diferencia entre el precio sucio calculado y el precio sucio objetivo (precio limpio objetivo + intereses devengados) sea cercana a cero.
En resumen, la clase Bono ofrece una herramienta robusta y modular para analizar bonos de cupón fijo, implementando cálculos financieros estándar y proporcionando métricas esenciales para la toma de decisiones.
