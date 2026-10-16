# Guion MLOps — 3 oradores + demo (≈ 9:20)

Basado en la PoC real: `ValennCoronel/TP-Arquitectura_MLOps` (Customer Churn).

**Regla de oro de la charla:** distinguir siempre qué es **REAL** (corre hoy), **MOCK** (simulado) y **ROADMAP** (diseñado, no construido). Es lo que le da credibilidad.

| Componente | Estado | Evidencia en el repo |
|---|---|---|
| Datos | **Mock** | `make_classification` sintético (3.000 muestras, 25% churn) |
| Entrenamiento + tracking | **Real** | `src/train.py`: LR vs Random Forest, MLflow local (`./mlruns`) |
| Selección de campeón | **Real** | Mejor F1 → `models/best_model.pkl` |
| API de inferencia | **Real** | `src/app.py`: FastAPI `/health` y `/predict`, validación Pydantic |
| Docker | **Real** | `Dockerfile`: python 3.11-slim, usuario no-root, healthcheck |
| CI | **Real** | GitHub Actions: flake8 + pytest en push/PR |
| CD a AWS | **Parcial** | `docker build` real; ECR + deploy (EC2/ECS/App Runner) **comentado** |
| Infra como código | **Código listo** | Terraform: S3, IAM, Security Group, EC2 t2.micro |
| Model Registry | Roadmap | Comentado en `train.py` |
| Monitoreo / drift | Roadmap | Evidently + Prometheus, descrito en README |
| Re-entrenamiento automático | Roadmap | No existe |
| Autoscaling | Roadmap | Hoy una sola EC2; ECS Fargate + ALB propuesto |

**Números reales de la corrida local** (dataset sintético): Regresión logística accuracy 0,745 · F1 0,349 · AUC 0,660. Random Forest accuracy 0,835 · F1 0,548 · AUC 0,865. Ganó Random Forest. Las métricas pueden variar levemente según la versión de librerías; volver a correr antes de presentar.

**Reparto**
- **Persona A:** slides 1, 2, 3, 9 y 10 (≈ 2:55)
- **Persona B:** slides 4, 5, 6 (≈ 2:40) y maneja la terminal en la demo
- **Persona C:** slides 7 y 8 (≈ 1:45) y narra la demo
- **Demo** (≈ 2:00): B maneja, C narra

---

## Slide 1 · Título — A (0:25)
> Buenas. Somos el equipo de plataforma de ML. Vamos a mostrarles una prueba de concepto end-to-end de MLOps sobre un caso concreto: predecir qué clientes se van a ir, el churn.
> Y vamos a ser muy claros en una cosa: en cada parte les vamos a decir qué está **realmente construido**, qué es **simulado**, y qué es **roadmap**.

## Slide 2 · El problema — A (0:40)
> Un modelo en un notebook no genera valor. Pasa lo de siempre: no sabemos con qué datos ni parámetros se entrenó el modelo bueno, el deploy es manual y depende de una persona, nadie monitorea qué pasa en producción, y la infraestructura se arma a clicks y no se puede reproducir.
> Un paper clásico de Google, de 2015, ya lo decía: el código del modelo es una parte chica del sistema; el resto es datos, infra y operación.
> Nuestro caso es churn: predecir qué clientes van a abandonar para poder actuar antes.

## Slide 3 · El ciclo y dónde estamos — A (0:50)
> MLOps es DevOps más datos más modelos, y es un ciclo. Miren los colores.
> En naranja, los **datos**: son sintéticos, no tenemos feature store; eso es mock.
> En verde, **entrenar y validar**: eso es real. Entrenamos dos modelos con MLflow y validamos con métricas, tests y linter.
> En celeste, **desplegar**: Docker y GitHub Actions funcionan, pero el último paso, el deploy a AWS, está escrito y comentado.
> En gris, **monitorear y re-entrenar**: roadmap.
> Ahora B les muestra la parte real.

## Slide 4 · Entrenamiento y tracking — B (0:50)
> Empecemos por lo que ya funciona. El script de entrenamiento genera el dataset, entrena una regresión logística y un Random Forest, y registra todo en MLflow: parámetros, métricas y artefactos de cada corrida.
> Después elige automáticamente el campeón por F1 y lo exporta a un archivo que la API va a servir.
> En las barras: el Random Forest gana en las tres métricas. Con F1 de 0,55 contra 0,35, y AUC de 0,86 contra 0,66.
> Una aclaración honesta: los datos son sintéticos, así que estos números muestran que el pipeline funciona, no que el modelo sea bueno para un negocio real.

## Slide 5 · Drift — B (1:00)
> Ahora, lo que todavía **no** tenemos, y por qué importa: el monitoreo de drift.
> La nube celeste son los datos con los que entrenamos; la otra es producción. Mientras muevo el slider, los datos se alejan y los puntos rojos son casos que el modelo nunca vio.
> Con un modelo que mueve, supongamos, veinte mil dólares por día en decisiones, veinte días sin darnos cuenta son unos dieciséis mil dólares de pérdida, y crece al cuadrado. Son supuestos ilustrativos, no mediciones.
> Con monitoreo, la alerta salta en un par de días. Por eso en el roadmap está Evidently y Prometheus.

## Slide 6 · CI/CD — B (0:50)
> Integración continua, real. En cada push y pull request corre flake8 y pytest. Cada cubo es un commit; los rojos son cambios que el linter o los tests frenan.
> Si pasa en main, se construye la imagen Docker. El Dockerfile usa un usuario sin privilegios y tiene healthcheck, así que lo que corre en nuestra máquina es lo mismo que correría en producción.
> La estación naranja, el deploy a AWS con ECR, ya está escrita en el workflow, pero comentada hasta conectar credenciales por OIDC. **C les cuenta la infraestructura.**

## Slide 7 · Infraestructura como código — C (0:45)
> La infraestructura está definida en Terraform. Un `terraform apply` crea: un bucket S3 versionado y cifrado para los artefactos, un rol IAM sin claves hardcodeadas, un Security Group, y una instancia EC2 t2.micro con Docker ya instalado.
> Es reproducible y revisable en un pull request, que es lo que buscamos.
> Seamos claros: hoy es **una sola instancia**. El camino a producción es ECS Fargate con un balanceador y autoscaling horizontal, y eso es roadmap.

## Slide 8 · Proyección — C (1:00)
> Esta calculadora es una **proyección**, no una medición. Suma tres fuentes de ahorro que aparecen al escalar: cómputo con Spot y autoscaling, horas manuales y pérdidas por drift evitadas.
> Con supuestos de ejemplo, **[ocho modelos, doce mil dólares mensuales de cómputo, sesenta horas manuales por semana]**, da alrededor de ciento noventa mil dólares al año, con payback de pocos meses.
> Lo importante es el orden de magnitud y que cada supuesto se puede discutir moviendo los sliders. **Ahora, en vez de seguir contando, lo corremos.**

## Demo en vivo — B maneja, C narra (2:00)
Antes de empezar: terminal en la carpeta del repo, entorno virtual activado, dependencias instaladas. Tener el video de respaldo en otra pestaña.

| Paso | Comando | Qué decir (C) |
|---|---|---|
| 1 | `python -m src.train` | "Entrena los dos modelos y elige el campeón por F1." |
| 2 | `mlflow ui` → http://127.0.0.1:5000 | "Cada corrida con sus parámetros y métricas, comparables." |
| 3 | `uvicorn src.app:app --port 8000` → http://localhost:8000/docs → POST `/predict` con `{"tenure_months":12,"monthly_charges":65.5,"total_charges":786,"num_support_tickets":2,"online_security":1}` | "Probabilidad de churn, nivel de riesgo y latencia." |
| 4 | `docker build -t customer-churn-api .` y `docker run -p 8000:8000 customer-churn-api` → `/health` | "Mismo servicio, empaquetado." |
| 5 | Mostrar la pestaña Actions de GitHub con el workflow | "En cada push corre esto." |

> **Plan B si falla:** "Para no depender de la red, les muestro la grabación."

## Slide 9 · Roadmap — A (0:30)
> El plan de noventa días sale del propio README. Mes uno: deploy real, con CD a AWS, MLflow remoto y Model Registry. Mes dos: observar, con Prometheus y detección de drift. Mes tres: escalar, con re-entrenamiento automático, autoscaling y Feature Store.

## Slide 10 · El pedido — A (0:30)
> Lo que pedimos es aprobar el paso de PoC a producción en noventa días, con tres metas: CD automático a AWS en cada push a main, drift detectado en menos de cuarenta y ocho horas, y serving con autoscaling.
> Gracias. ¿Preguntas?

---

## Antes de presentar (checklist)
- [ ] **Arreglar el linter**: `flake8 src tests` hoy reporta 6 violaciones (E501 línea > 100 en `app.py:66`, `train.py:31,86,228`; E303/E302 en `app.py:147-150`). Con eso, el job de CI **falla** en GitHub Actions. Conviene dejarlo en verde antes de mostrar la pestaña Actions.
- [ ] Correr `python -m src.train` una vez antes y revisar que los números del slide 4 coincidan con lo que muestra su MLflow.
- [ ] Reemplazar los datos entre **[corchetes]** del slide 8 con números del equipo.
- [ ] Docker Desktop encendido si van a hacer el paso 4.

## Preguntas probables
- **¿Por qué los datos son sintéticos?** Para que la PoC corra sin dependencias externas; en producción se reemplaza por un Feature Store o un data warehouse.
- **¿Esto ya está en AWS?** El Terraform y el pipeline están escritos; el deploy a AWS está comentado hasta conectar credenciales.
- **¿Cómo van a detectar drift?** Evidently AI sobre las predicciones y entradas logueadas, más Prometheus para métricas de la API. Es roadmap.
- **¿Y si el re-entrenamiento automático empeora el modelo?** Pasaría por el mismo pipeline: métricas, tests y comparación contra el campeón antes de reemplazar.
- **¿De dónde salen los números del ROI?** Son supuestos editables; el piloto sirve para medirlos.
