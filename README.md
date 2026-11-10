# Laboratorio de aplicación inteligente adaptativa

## Identificación

- Integrantes: Francisco Alfaro FLores




## Objetivo y alcance

Complete un recomendador de recursos de aprendizaje que represente los intereses
de cada usuario y actualice su perfil al recibir la interacción **Me interesa**.
El núcleo se implementa en Python y se ejecuta por consola. Una interfaz Angular
puede incorporarse como extensión después de esta actividad.

El catálogo, los perfiles, las validaciones, la aplicación por consola y el registro
de evidencias se proporcionan. Complete únicamente los cuatro bloques señalados
en `motor.py` y documente su interpretación. No modifique los datos de las pruebas
ni la aplicación para obtener un resultado esperado.

La representación usa **solo la columna etiquetas**. Títulos y descripciones sirven
para presentar el catálogo. Todos los recursos continúan disponibles después de
una interacción; no se excluye el recurso valorado. Esto permite comparar el mismo
conjunto de 12 candidatos antes y después de la actualización del perfil.

## Preparación

Use Python 3.10 o posterior. Abra una terminal en esta carpeta.

```bash
python -m venv .venv
```

En Windows puede usar `py -m venv .venv`.

Activación en PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Activación en CMD:

```bat
.venv\Scripts\activate.bat
```

Activación en macOS o Linux:

```bash
source .venv/bin/activate
```

Instalación y comprobación:

```bash
python -m pip install -r requirements.txt
python -c "import sys; print(sys.executable)"
python app.py --comprobar
```

La comprobación funciona con los cuatro bloques sin completar. Debe informar
12 recursos, 4 usuarios y una matriz TF-IDF de 12 filas. Para esta ejecución no
activé el entorno: invoqué directamente `.\.venv\Scripts\python.exe`. Versiones
utilizadas: Python 3.15.2, NumPy 2.5.4, pandas 3.1.1 y scikit-learn 1.9.1. Las
dependencias y sus versiones completas están registradas en `requirements.txt`.

```bash
python -m pip freeze > requirements.txt
```

## Funciones que debe completar

| Función | Entrada principal | Salida y requisito |
|---|---|---|
| `crear_perfil` | Texto de intereses y vectorizador ya ajustado | Vector NumPy de una dimensión obtenido con `transform`; no ajustar nuevamente el vocabulario. |
| `calcular_puntuaciones` | Vector del perfil y matriz de recursos | Un valor de similitud del coseno por recurso. |
| `recomendar` | Perfil y catálogo indexado | Hasta 3 resultados con puntuación positiva; orden descendente y desempate por `id` ascendente. |
| `actualizar_perfil` | Perfil anterior y vector del recurso valorado | Vector normalizado de la combinación `0.4 * perfil + 0.6 * recurso`. |

Use `construir_resultado` al construir cada recomendación. Esta función proporciona
`id`, `titulo`, `puntuacion` y una `explicacion` basada en términos compartidos.

Un perfil vacío o compuesto por palabras fuera del vocabulario produce un vector
nulo. En esos casos deben obtenerse puntuaciones cero y una lista de recomendaciones
vacía; la aplicación ya proporciona el mensaje informativo.

El parámetro 0.6 es una elección didáctica que da mayor peso a la interacción reciente.
No es un valor validado experimentalmente. Una similitud no es una probabilidad de
interés. El ejercicio no entrena un modelo supervisado ni demuestra comprensión
semántica de los recursos.

## Ejecución después de completar el motor

Modo interactivo:

```bash
python app.py
```

Seleccione un usuario. Puede consultar el catálogo, solicitar recomendaciones y
registrar **Me interesa** indicando el ID de cualquier recurso, aunque no aparezca
entre los tres primeros. Un ID desconocido muestra un aviso y no actualiza el perfil.
Los perfiles y sus historiales de interacciones se conservan por separado durante
la ejecución. La consola muestra el historial del usuario activo. Al reiniciar se
cargan nuevamente los intereses iniciales del JSON y los historiales quedan vacíos.

Demostración reproducible:

```bash
python app.py --demo
```

Este comando parte de los perfiles iniciales, ejecuta T1-T4 y genera:

- `evidencias/reporte_demo.json`: perfiles, historiales, recomendaciones y comprobaciones.
- `evidencias/puntuaciones_antes_despues.csv`: los mismos 12 recursos con sus
  puntuaciones anteriores, posteriores y diferencias, incluyendo ceros.

Que la aplicación muestre **Cumple** es una comprobación orientativa. Revise las
puntuaciones y explique qué evidencia respalda cada resultado.

## Pruebas funcionales y evidencias

| Caso | Entrada o acción | Resultado esperado | Resultado observado y evidencia | Estado |
|---|---|---|---|---|
| T1 | Recomendar para u1 y u2 sin interacciones | R02 primero para u1; R07 primero para u2; orden descendente, máximo 3 y puntuaciones positivas. | Cumple: u1 R02 (1.0000), R12 (0.4965), R03 (0.4326); u2 R07 (1.0000), R08 (0.5677), R09 (0.3087). | Cumple |
| T2 | u1 indica Me interesa sobre R04 y se recalcula | Cambia el perfil; R04 pasa al primer lugar; aumenta la puntuación de R05; se conservan los 12 candidatos. | Cumple: R04 pasa primero (0.8752); R05 sube de 0.0000 a 0.4378; cambia el perfil y quedan 12 candidatos. | Cumple |
| T3 | Consultar u2 después de la interacción de u1 | El perfil, el historial y las puntuaciones de u2 se mantienen. | Cumple: perfil, historial, puntuaciones y recomendaciones de u2 no cambian. | Cumple |
| T4 | Recomendar para u3 y u4 | Cero puntuaciones positivas, lista vacía y mensaje informativo para ambos. | Cumple: u3 (intereses vacíos) y u4 (zzxyz123) reciben lista vacía, puntuaciones cero y el mensaje informativo. | Cumple |

### Comparación de la adaptación

- Perfil de u1 antes de valorar R04: `pandas=0.753247`, `datos=0.465091`,
  `python=0.465091`.
- Perfil de u1 después de valorar R04: `python=0.637116`, `api=0.446269`,
  `web=0.446269`, `pandas=0.376477`, `datos=0.232455`.
- Puntuación de R05 antes y después: `0.000000` y `0.437777`.
- Por qué ese cambio es evidencia de adaptación: la acción sobre R04 incorpora
  `api` y `web` al perfil actualizado; R05 comparte esos términos, por lo que su
  puntuación sube. A la vez R04 queda primero. Los 12 recursos siguen siendo
  candidatos.
- Problemas detectados y acciones realizadas: no se detectaron fallos en T1-T4.
  Los cuatro casos pasaron; los resultados completos están en `evidencias/`.

No sustituya estas observaciones por cifras inventadas. Use los resultados de su ejecución.

## Preguntas evaluadas

1. ¿Qué información describe el contenido de los recursos y qué información constituye el perfil del usuario? Identifique ambas en los archivos del proyecto.
   Respuesta: El contenido de cada recurso se describe en `datos/recursos.csv`;
   el motor utiliza la columna `etiquetas` para representarlo con TF-IDF. El
   título y la descripción solo presentan el recurso. El perfil inicial del
   usuario es el texto `intereses` de `datos/perfiles.json`; `crear_perfil` lo
   transforma al mismo espacio vectorial y la aplicación conserva ese vector
   como estado en memoria.
2. ¿Por qué se utiliza fit_transform sobre el catálogo y transform sobre los intereses del usuario? ¿Qué problema causaría ajustar un vectorizador diferente para cada perfil?
   Respuesta: `fit_transform` aprende del catálogo el vocabulario, los pesos
   IDF y transforma sus etiquetas; `transform` convierte los intereses con ese
   mismo vocabulario y esos mismos pesos. Si se ajustara un vectorizador por
   perfil, las dimensiones, el orden y los pesos de las características podrían
   diferir, haciendo inválida o no comparable la similitud con los vectores del
   catálogo.
3. ¿Por qué el mismo catálogo puede producir recomendaciones distintas para u1 y u2? ¿Una similitud de 0,80 significa que existe un 80 % de probabilidad de que el recurso guste? Justifique.
   Respuesta: Cada perfil tiene términos y pesos distintos, así que la similitud
   coseno con las mismas filas del catálogo cambia: en la prueba u1 prioriza R02
   y u2 prioriza R07. Un valor de 0.80 no significa una probabilidad del 80 %:
   es una medida de similitud entre vectores TF-IDF, no una probabilidad
   calibrada; no se entrenó un modelo con resultados de aceptación/rechazo.
4. ¿En qué se diferencia modificar manualmente los intereses iniciales de la adaptación implementada? Identifique el evento que provoca el cambio, el estado modificado y su efecto observable.
   Respuesta: Editar `intereses` en el JSON cambia manualmente la semilla que se
   vuelve a cargar al iniciar la aplicación. La adaptación ocurre cuando el
   usuario elige **Me interesa** para un ID válido: `registrar_interes` actualiza
   solo el vector activo de ese usuario con la combinación normalizada
   `0.4 * perfil + 0.6 * recurso` y agrega el evento a su historial. El cambio
   reordena las recomendaciones durante esa ejecución; al reiniciar se carga
   otra vez el JSON inicial.
5. Relacione su implementación con las etapas del ciclo MAPE-K estudiado en clase: Monitorizar, Analizar, Planificar, Ejecutar y Conocimiento. Indique una función, operación o dato de su programa para cada etapa.
   Respuesta: **Monitorizar:** la opción de consola **Me interesa** y el ID del
   recurso capturan la interacción. **Analizar:** la vectorización TF-IDF y
   `calcular_puntuaciones` calculan afinidad mediante similitud coseno.
   **Planificar:** `recomendar` ordena las afinidades y selecciona hasta tres
   resultados positivos. **Ejecutar:** `actualizar_perfil` aplica la
   retroalimentación y la consola muestra el nuevo ranking. **Conocimiento:**
   el catálogo y su vocabulario TF-IDF, junto con los perfiles y el historial
   por usuario, son el estado utilizado por la aplicación durante la sesión.
6. Indique una limitación del mecanismo implementado y proponga una mejora concreta. Explique qué dato adicional o qué cambio del algoritmo exigiría esa mejora.
   Respuesta: El método depende de coincidencias literales de etiquetas y del
   peso fijo 0.6; no comprende sinónimos ni modela explícitamente el desinterés.
   Una mejora concreta sería permitir también la acción **No me interesa** y
   guardarla junto con el ID en el historial. El recomendador podría penalizar
   la similitud con recursos rechazados además de premiar los aceptados. Esto
   exige recopilar etiquetas de retroalimentación positiva/negativa por usuario
   y modificar la puntuación para considerar ambas señales.

## Archivos de la entrega

Entregue `app.py`, `motor.py`, `datos/recursos.csv`, `datos/perfiles.json`,
`requirements.txt`, `.gitignore`, este README completado y los dos archivos de
`evidencias/`. No incluya `.venv`, `__pycache__` ni dependencias instaladas.

### Reproducción desde una copia nueva

Compruebe que otra persona puede crear el entorno, instalar las dependencias y
ejecutar los comandos anteriores utilizando únicamente los archivos entregados.
No se requieren instrucciones adicionales: cree un entorno con Python 3.10 o
posterior, instale `requirements.txt` y ejecute `python app.py --comprobar`,
`python app.py --demo` o `python app.py`.
