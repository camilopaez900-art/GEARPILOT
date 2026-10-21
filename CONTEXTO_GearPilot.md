# Contexto del proyecto

> Peguen este archivo completo al inicio de cada conversación nueva con la IA.
> Actualícenlo cada vez que terminen o publiquen un módulo.

## Proyecto

**Nombre:** GearPilot

**Descripción:** Aplicación web para organizar y administrar la información de un taller de mecánica automotriz. Permite registrar clientes, vehículos, mecánicos, órdenes de trabajo, cotizaciones y pagos, y consultar el historial de arreglos. El administrador registra el ingreso de vehículos por placa y documento, asigna mecánico, cotiza y actualiza el estado de la orden (En proceso o Completado). El cliente consulta el historial de arreglos, sus pagos y el mecánico asignado.

**No incluye:** pagos en línea (los pagos solo se registran, no se cobran desde la app), React, Angular, Django, Spring, Docker ni ORM. Las citas con validación de disponibilidad (Módulo 6) solo se hacen si sobra tiempo, y la vista móvil completa del cliente es lo primero que se recorta.

**Repositorio:** [enlace del repositorio del equipo en GitHub]

**Aplicación publicada:** [enlace donde está publicada la aplicación en internet, tipo https://USUARIO.pythonanywhere.com]

## Nivel del equipo

Somos estudiantes principiantes. Sabemos fundamentos de programación en Python y Java, programación orientada a objetos (clases, encapsulamiento, herencia), listas y HTML básico. No sabemos bases de datos ni desarrollo web con backend. Nunca hemos publicado una aplicación. Trabajamos en Windows con Visual Studio Code.

Tiempo disponible: 4 semanas, 4 horas por semana por persona, 2 integrantes (32 horas combinadas).

## Integrantes

| Integrante | Usuario de GitHub | Responsable de |
|---|---|---|
| | | |
| | | |
| | | |

## Herramientas elegidas

- **Frontend:** HTML + CSS + JavaScript básico (a partir del prototipo `index.html`), servido con plantillas Jinja2 (incluidas en Flask)
- **Backend:** Python + Flask
- **Base de datos:** SQLite (módulo `sqlite3` de Python, archivo `gearpilot.db`)
- **Publicación:** PythonAnywhere (plan gratis Beginner)
- **Editor:** Visual Studio Code

Apoyo: Git + GitHub para trabajar en equipo y subir el código al servidor. El archivo `.db` no se sube a GitHub (`.gitignore`); se crea con `init_db.py`.

## Forma de organizar el código

Clases para las reglas del negocio y funciones para la base de datos y las páginas:

- `modelos.py`: clases POO (`Persona`, `Administrador`, `Cliente`, `Mecanico`, `MecanicoJefe`, `Vehiculo`, `OrdenTrabajo`) con atributos privados, getters/setters y validaciones (por ejemplo, el setter de `documento`: solo dígitos, máximo 3 intentos).
- `db.py`: funciones sueltas con SQL directo (`sqlite3`, siempre con marcadores `?`), sin ORM.
- `app.py`: rutas de Flask (funciones).

Por qué: Flask funciona con funciones, y las reglas de la ficha (validaciones, herencia) encajan naturalmente como clases.

## Datos que guarda el sistema

| Nombre | Campos | Se relaciona con |
|---|---|---|
| administradores | id, documento, nombre, telefono, correo, contrasena (hash) | Ninguna (solo inicia sesión y administra) |
| clientes | id, documento (único), nombre, telefono, correo, contrasena (hash) | Tiene muchos vehiculos |
| mecanicos | id, documento, nombre, telefono, correo, especialidad, es_jefe (0 o 1) | Tiene muchas ordenes_trabajo |
| vehiculos | id, placa (única), marca, modelo, anio, cliente_id | Pertenece a un cliente; tiene muchas ordenes_trabajo y citas |
| ordenes_trabajo | id, vehiculo_id, mecanico_id, motivo, diagnostico, estado (Diagnóstico, Cotizado, En proceso, Completado), respuesta_cotizacion (Pendiente, Aceptada, Rechazada), fecha_ingreso, fecha_cierre | Un vehiculo, un mecanico; tiene muchos cotizacion_items y pagos |
| cotizacion_items | id, orden_id, concepto, tipo (Repuesto o Servicio), valor | Pertenece a una orden de trabajo (el total se calcula con SUM, no se guarda) |
| pagos | id, orden_id, monto, metodo, fecha | Pertenece a una orden de trabajo (saldo pendiente = total − suma de pagos) |
| citas | id, vehiculo_id, fecha, hora, motivo (con UNIQUE en fecha y hora) | Pertenece a un vehículo (Módulo 6, opcional) |

## Estructura de carpetas

```
gearpilot/
├── app.py            rutas de Flask
├── db.py             funciones SQL
├── modelos.py        clases POO
├── schema.sql        CREATE TABLE de todas las tablas
├── init_db.py        crea gearpilot.db desde schema.sql
├── requirements.txt
├── .gitignore        venv/, *.db, __pycache__/
├── templates/        base.html + una página por vista
└── static/estilos.css
```

## Módulos

| # | Módulo | Estado | Publicado | Responsable |
|---|---|---|---|---|
| 0 | Página mínima publicada (Hola) + base de datos | Pendiente | No | |
| 1 | Login y panel del administrador | Pendiente | No | |
| 2 | Clientes y vehículos | Pendiente | No | |
| 3 | Mecánicos y órdenes de trabajo | Pendiente | No | |
| 4 | Cotización y pagos | Pendiente | No | |
| 5 | Portal del cliente (solo consulta) | Pendiente | No | |
| 6 | Citas (opcional) | Pendiente | No | |

Estados posibles: Pendiente, En curso, Terminado.
Un módulo solo está terminado cuando funciona en la versión publicada.

Plan por semana: S1 = Módulo 0; S2 = Módulos 1 y 2; S3 = Módulos 3 y 4; S4 = Módulo 5 y cierre.

## Módulo actual

[en qué módulo están trabajando y en qué tarea van]

## Problemas pendientes

- Confirmar con el profesor el cambio de herencia (clase base `Persona`; `Administrador`, `Cliente` y `Mecanico` heredan de ella, `MecanicoJefe` hereda de `Mecanico`) frente a la ficha original (`Mecanico` y `MecanicoJefe` hijas de `Administrador`).
- Si el tiempo real es de 4 horas combinadas por semana (no por persona), rehacer la ruta: el alcance llegaría solo hasta el Módulo 3.

## Decisiones importantes

- 2026-10-20: Backend en Python con Flask, base de datos SQLite, publicación en PythonAnywhere; una herramienta por capa.
- 2026-10-20: Login simple con usuario y contraseña (contraseña cifrada con `werkzeug.security`) para administrador y cliente.
- 2026-10-20: `MecanicoJefe` es un mecánico con `es_jefe = 1`, sin tabla propia.
- 2026-10-20: Una orden de trabajo puede tener varios pagos (anticipo y saldo), y un cliente puede tener varios vehículos.
- 2026-10-20: El total de la cotización no se guarda; se calcula sumando los ítems.
- 2026-10-20: SQL directo con `sqlite3` y marcadores `?`, sin ORM. El archivo `.db` no se sube a GitHub.
- 2026-10-20: Orden de recorte si falta tiempo: primero Citas (Módulo 6), luego la vista móvil del cliente (Módulo 5 reducido a una página de consulta), luego las pantallas de mecánico jefe y las estadísticas extra del panel.
