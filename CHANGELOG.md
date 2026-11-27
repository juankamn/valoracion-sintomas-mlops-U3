# CHANGELOG

Cambios de la propuesta inicial a esta propuesta. El problema es el mismo: estimar, a partir de síntomas, si es posible que un paciente tenga una enfermedad, tanto si hay muchos casos como si hay pocos.

## Qué cambió

### Aclaraciones para llevar la propuesta a la práctica

Se precisa cómo preparar las variables compartidas para las enfermedades huérfanas y se aclara que la alerta genérica debe validarse. También se detalla la separación de la prueba para clases pequeñas y se presentan los umbrales como puntos de partida.

Se distingue el servicio simulado de la unidad 2 del modelo futuro: el contrato deberá actualizarse si se agregan variables, confianza o alertas. Se añaden indicaciones iniciales sobre almacenamiento, monitoreo y comparación entre pandas y Spark. Las reglas del ejercicio son una simulación y requieren validación antes de cualquier uso real.

### Cada etapa nombra la herramienta y el motivo

La propuesta inicial describía el flujo y nombraba la regresión logística, la potenciación de gradiente y el contenedor. El resto de las etapas no decía con qué se hacían.

Esta propuesta asigna una herramienta a cada paso: Git para el manifiesto, Python y pandas para la tabla, scikit-learn para los dos modelos, pytest para el contrato del servicio, Docker para la imagen, GitHub Actions para publicarla solo si las pruebas pasan, y GitHub Container Registry para que el computador del médico y el servidor usen la misma imagen.

### Las suposiciones quedan en la etapa, con su implicación

La propuesta inicial reunía las restricciones al principio: privacidad, etiqueta tardía, desbalance, solapamiento, códigos distintos, error asimétrico y enfermedad ausente.

Esta propuesta conserva esas restricciones y, además, en cada etapa dice qué se está suponiendo y qué pasa si no se cumple. Por ejemplo: si el identificador del paciente cambia, la métrica queda inflada y el candidato no se promueve; si la huérfana no se expresa en las variables ya entrenadas, no se le pone nombre.

### El médico tiene dos formas de uso, con el mismo contrato

La propuesta inicial decía que el médico podía correr el contenedor en su equipo o que el hospital lo publicaba en su red.

Esta propuesta separa las dos formas y las deja en el mismo registro. Si los recursos son bajos, el médico ejecuta la imagen en su computador. Si el modelo está alojado, llama a la misma API en un servidor del hospital o en una instancia pequeña. La versión de la imagen es la versión del modelo.

### El resumen del médico pasa de cuatro estados a cinco

La propuesta inicial devolvía `NO ENFERMO`, `ENFERMEDAD LEVE`, `ENFERMEDAD AGUDA` y `ENFERMEDAD CRÓNICA`.

Esta propuesta añade `ENFERMEDAD TERMINAL`. La regla terminal se evalúa antes que la crónica: 180 días o más, dolor 8 o más y 6 síntomas o más. Un cuadro de 240 días con ese dolor y esos síntomas no se marca crónico. La puerta de pruebas exige un caso de cada estado y el rechazo de los valores fuera de rango.

### La puerta de promoción incluye las pruebas del servicio

La propuesta inicial promovía el modelo por la métrica del régimen y por una muestra de falsos negativos revisada por un médico. Las pruebas del servicio se mencionaban aparte.

Esta propuesta mantiene esa métrica y esa revisión. Además, la imagen solo se publica si pytest pasa: los cinco casos fijos y los rangos inválidos. Se propone añadir pruebas sobre la imagen construida; en la unidad 2 las pruebas corren en el entorno de Actions antes de construirla.

### El registro es un manifiesto

La propuesta inicial guardaba la versión con sus datos, métricas y umbrales, y pedía no desplegar un archivo suelto.

Esta propuesta escribe ese paquete como un manifiesto en Git: conjunto `vN`, código, métricas y umbrales. La imagen cita ese manifiesto.

### El monitoreo incluye el reporte de predicciones

La propuesta inicial vigilaba latencia, errores, proporción de estados, síntomas fuera del diccionario y el desplazamiento de la fiebre alta o de los cuadros de 90 días o más. La sensibilidad real seguía esperando a la etiqueta.

Esta propuesta conserva esas alertas y añade el reporte que escribe el servicio: conteo de los cinco estados, últimas cinco consultas y fecha de la última predicción. Una consulta rechazada por rango no entra al reporte.

### Los lotes grandes tienen orden y, si no caben, se parten

La propuesta inicial recorría ingesta, reentrenamiento trimestral de las comunes y reentrenamiento de las huérfanas cuando llegaban confirmaciones, sin nombrar un ordenador de tareas.

Esta propuesta usa Airflow para esos tres trabajos, en ese orden, y solo reemplaza el modelo si vuelve a pasar la puerta. Spark parte las filas cuando el lote de historias no cabe en una máquina. Si cabe, sigue pandas. La consulta de un paciente no pasa por Airflow ni por Spark.

## Qué no cambió

- El problema y la salida: una estimación con confianza, no un diagnóstico.
- Los dos caminos: comunes con muchos confirmados, huérfanas con pocos, y alerta genérica por debajo del mínimo.
- Los mínimos de partida: 200 para una clase común, de 20 a 199 para una huérfana, y por debajo de 20 sin nombre.
- La partición por paciente, con un bloque de otro período que no elige umbrales.
- La privacidad: los identificadores directos no entran al entrenamiento.
- La etiqueta tardía: un episodio sin diagnóstico requiere confirmación o revisión antes de etiquetarlo; cumplir 12 meses no basta por sí solo.
- La exactitud global no es puerta. En huérfanas se mira la sensibilidad de la alerta.
- Quedan fuera el texto libre, la imagen diagnóstica y la certificación como dispositivo médico.
