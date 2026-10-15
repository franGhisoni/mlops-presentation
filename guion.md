# Guion MLOps — 3 oradores + demo (≈ 9:20, con ~40 s de margen)

**Reparto**
- **Persona A** — abre y cierra: slides 1, 2, 3, 9 y 10 (≈ 2:55)
- **Persona B** — costo, drift y despliegues: slides 4, 5, 6 (≈ 2:40) y maneja la pantalla en la demo
- **Persona C** — AWS y ROI: slides 7 y 8 (≈ 1:45) y narra la demo
- **Demo** — slide 9 de la web (≈ 2:00): B maneja, C narra
- Las otras 3 personas del equipo (6 en total) quedan para preguntas y apoyo técnico, o repartirse los roles al revés si prefieren que presente la otra mitad.

Ritmo de lectura: ~130 palabras por minuto. Los datos en **[corchetes]** hay que reemplazarlos por números y casos reales del equipo. Las cifras del deck son ilustrativas.

---

## Slide 1 · Título — A (0:25)
> Buenas. Somos el equipo de plataforma de ML y vamos a contarles en diez minutos por qué MLOps es la forma más barata, más rápida y más segura de operar modelos.
> Primero una pregunta para todos: ¿cuánto tarda hoy en llegar un modelo del notebook a producción? Quédense con ese número, porque al final lo vamos a comparar.

## Slide 2 · El problema — A (0:40)
> Hoy entrenamos y desplegamos a mano. Eso tiene cuatro costos.
> Uno: entrenamos sin versionar. **[Caso real: "el mes pasado nadie pudo reproducir el modelo bueno"]**.
> Dos: un deploy manual nos lleva dos o tres días y depende de una sola persona.
> Tres: nadie mira el modelo una vez en producción. Nos enteramos cuando se queja el cliente.
> Y cuatro: tenemos GPUs prendidas veinticuatro siete por las dudas.
> Y ojo: un paper clásico de Google, de 2015, ya lo decía. El código del modelo es una parte chica del sistema. Lo demás es datos, infraestructura y operación. Y esa parte es la que nos está costando.

## Slide 3 · El ciclo MLOps — A (0:50)
> MLOps es DevOps más datos más modelos. Y lo importante es que es un ciclo, no una línea.
> Arrancamos con datos versionados y validados. Entrenamos con pipelines reproducibles. Validamos con tests, no solo del código sino de los datos y del modelo. Desplegamos con CI/CD y canary. Monitoreamos drift, latencia y calidad. Y cuando algo se degrada, re-entrenamos automáticamente.
> Cada vuelta del ciclo es automática, auditable y repetible. Eso es lo que nos va a dar ahorro y tranquilidad. **Ahora B les muestra cuánto.**

## Slide 4 · Costo y riesgo de entrenar — B (0:50)
> Empecemos por entrenar. Entrenar a ciegas es caro por tres razones.
> Primera: usamos instancias on-demand. Con Spot, AWS publica descuentos de hasta noventa por ciento. El riesgo es que se interrumpan, pero con checkpoints no perdemos horas de GPU.
> Segunda: sin tracking de experimentos repetimos corridas porque no sabemos qué ya probamos.
> Tercera: sin cache de features recalculamos lo mismo una y otra vez.
> En las barras, naranja es hoy y verde es con MLOps. Son índices ilustrativos: el cómputo baja a un tercio, los reentrenos repetidos casi desaparecen.

## Slide 5 · Drift — B (1:00)
> Ahora el riesgo que más duele: el drift. El modelo no se rompe con un error, se pudre en silencio, porque el mundo cambia y los datos de producción dejan de parecerse a los de entrenamiento.
> Miren la nube celeste: son los datos con los que entrenamos. La otra es producción. Mientras muevo este slider, ven cómo se va alejando y los puntos se ponen rojos: datos que el modelo nunca vio.
> Si tardamos veinte días en notarlo, con un modelo que mueve **[veinte mil dólares por día]**, perdemos unos dieciséis mil. Con cuarenta y cinco días, más de ochenta mil. Y crece al cuadrado.
> Con monitoreo automático, la alerta salta en un par de días y se dispara el re-entrenamiento: la pérdida es de centavos. Esa diferencia es la que justifica el monitoreo solo.

## Slide 6 · Despliegues automatizados — B (0:50)
> Tercer ahorro: los despliegues. Cada cubo que ven es un commit viajando por el pipeline.
> En cada commit corren tests de código, de datos y de modelo. Los cubos rojos son cambios que los tests frenaron antes de llegar a producción. Ese contador de "frenados" es plata que no perdimos y clientes que no se enteraron.
> Los que pasan hacen canary al cinco por ciento, se comparan contra el modelo actual y, si todo está bien, siguen. Si la métrica cae, rollback automático.
> Pasamos de dos días a quince minutos, sin pasos manuales. **C les cuenta sobre qué infraestructura corre todo esto.**

## Slide 7 · Infra escalable en AWS — C (0:45)
> Todo esto corre en AWS, y lo clave es que pagamos por lo que usamos, no por el pico.
> Esta es la carga de tráfico en vivo. A medida que sube, se prenden instancias; cuando baja, se apagan. Arriba ven el ahorro contra tener todo provisionado para el pico: ronda el cincuenta por ciento.
> La arquitectura es simple: S3 para datos, SageMaker Pipelines para entrenar, Model Registry para versionar, CodePipeline y ECR para CI/CD, endpoints con autoscaling, Model Monitor y CloudWatch para el drift, y EventBridge para disparar el re-entrenamiento. Todo servicios gestionados, no tenemos que operar servidores.

## Slide 8 · ROI — C (1:00)
> Juntemos todo en un número. Esta calculadora suma tres ahorros: cómputo, horas del equipo y drift evitado.
> Con los números de hoy, **[ocho modelos, doce mil dólares al mes de cómputo, sesenta horas semanales en tareas manuales]**, da alrededor de ciento noventa mil dólares al año.
> Si me dejan mover los sliders, ven que aunque seamos más conservadores el payback sigue siendo de pocos meses, contra una inversión de adopción de unos treinta mil dólares.
> Son supuestos editables, no promesas: lo importante es que el orden de magnitud es claro. **Y ahora, en vez de seguir contando, se lo mostramos.**

## Demo en vivo — B maneja, C narra (2:00)
> **C:** Lo que van a ver es el flujo completo, de un commit a producción.
> **B:** Hago un cambio en el código del modelo y lo subo. *[git push]*
> **C:** Eso dispara el pipeline: tests, entrenamiento y evaluación, sin que nadie toque nada.
> **B:** Acá ya aparece la nueva versión en el Model Registry, con sus métricas y la aprobación.
> **C:** Se despliega con canary al cinco por ciento. Miren cómo se comparan las métricas con el modelo anterior.
> **B:** Ahora simulamos drift en los datos de entrada. *[correr script de drift]*
> **C:** Model Monitor lo detecta, salta la alerta y se dispara el re-entrenamiento automático. Todo lo que contamos recién, en dos minutos.
>
> **Plan B si falla:** tener un video grabado de la demo en una pestaña. Decir: "Para no depender del wifi, les muestro la grabación".

## Slide 9 · Equipo y roadmap — A (0:30)
> Esto lo hacemos entre los seis. Cada uno toca la plataforma: data engineering, ML engineering, data science, DevOps y producto. Nadie es cuello de botella.
> El plan son noventa días. Mes uno: fundaciones, datos versionados y pipelines reproducibles. Mes dos: delivery, CI/CD, registry y endpoints. Mes tres: operación, monitoreo de drift y re-entrenamiento automático.

## Slide 10 · El pedido — A (0:30)
> Lo que pedimos es concreto: aprobar un piloto de noventa días con un modelo y el equipo completo.
> Y nos comprometemos a tres métricas: deploy en menos de una hora, contra los dos o tres días de hoy; cómputo treinta por ciento más barato; y drift detectado en menos de cuarenta y ocho horas.
> Acuérdense de la pregunta del principio: ¿cuánto tarda hoy llegar a producción? Queremos que la respuesta sea "una hora".
> Gracias. ¿Preguntas?

---

## Preguntas probables (para el equipo de apoyo)
- **¿Por qué SageMaker y no armarlo a mano?** Servicios gestionados = menos operación; podemos migrar piezas si hace falta.
- **¿Y si Spot se interrumpe?** Checkpoints cada N pasos; el job retoma desde el último, no desde cero.
- **¿Cuánto cuesta el monitoreo?** Es un porcentaje chico del cómputo; se compara contra una sola incidencia de drift. **[Dato real]**
- **¿Y si el re-entrenamiento automático empeora el modelo?** Pasa por el mismo pipeline: tests, comparación y canary antes de reemplazar.
- **¿De dónde salen los números?** Son supuestos editables en la calculadora; el piloto sirve justamente para medirlos.
