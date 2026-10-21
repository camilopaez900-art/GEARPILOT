# GearPilot: informe y ruta de construcción

Equipo: Naren Dimate y Camilo Páez · Plazo: 4 semanas · Windows + VS Code

## 0. Supuestos y decisiones

- **Tiempo:** entendí 4 horas por semana **por persona**, es decir 8 horas combinadas por semana y 32 en total. Si son 4 horas entre los dos, el plan solo alcanza hasta el Módulo 3 (ver sección 6).
- **Lenguaje:** Python (lo eligieron).
- **Login:** usuario y contraseña simple, para administrador y cliente.
- **Publicación:** obligatoria, con enlace público.
- **Recorte acordado:** si falta tiempo, lo primero que sale es la vista móvil del cliente.

## 1. Herramientas (una por capa)

| Capa | Herramienta | Por qué es la más fácil para ustedes |
|---|---|---|
| a) Frontend | **HTML + CSS + un poco de JavaScript** (su prototipo), servido con plantillas **Jinja2** | Ya saben HTML y el prototipo está hecho. Jinja viene incluido con Flask y solo agrega unas pocas marcas como `{{ nombre }}` dentro del HTML. |
| b) Backend | **Python + Flask** | Es el framework web más pequeño de Python: una app mínima son 6 líneas. Se escribe con las funciones y clases que ya conocen. |
| c) Base de datos | **SQLite** | Viene incluida en Python (módulo `sqlite3`). No se instala nada, no hay servidor ni usuario o contraseña de base de datos: es un archivo `.db`. |
| d) Publicación | **PythonAnywhere (plan gratis "Beginner")** | Está pensado para Flask y SQLite funciona tal cual. Es un panel web con clics, sin terminal de servidor ni Docker. Obtienen un enlace `https://SUUSUARIO.pythonanywhere.com`. |

Complemento: **Git + GitHub** (gratis) para trabajar los dos sin pisarse y para subir el código al servidor. Es una herramienta de apoyo, no una capa de la app.

> Nota: los límites del plan gratis de PythonAnywhere pueden cambiar. Léanlos en su página al crear la cuenta. Hoy la app gratis debe "extenderse" pulsando un botón en el panel cada cierto tiempo. Háganlo antes de la sustentación.

### Qué tienen que aprender (y cuánto)

- **Flask:** rutas (`@app.route`), recibir un formulario (`request.form`), mostrar una página (`render_template`), sesión de usuario (`session`). Unas 2 horas de práctica.
- **SQL básico:** `CREATE TABLE`, `INSERT`, `SELECT ... WHERE`, `UPDATE`, `JOIN` simple. Unas 2 a 3 horas.
- **Git básico:** `add`, `commit`, `push`, `pull`.
- **Jinja:** `{{ variable }}`, `{% for x in lista %}`, `{% if %}`, `{% extends "base.html" %}`.

### Instalación paso a paso (Windows)

**1. Python**
1. Entren a python.org → Downloads → descarguen la versión estable más reciente para Windows.
2. En el instalador, **marquen "Add python.exe to PATH"** antes de pulsar *Install Now*. Es el error de instalación más común.
3. Abran VS Code → menú *Terminal → New Terminal* y escriban `python --version`. Debe mostrar un número de versión.
4. En VS Code, pestaña Extensions (Ctrl+Shift+X): instalen **Python** (Microsoft) y **SQLite Viewer** (para abrir el archivo `.db` y ver las tablas).

**2. Carpeta del proyecto y Flask**
En la terminal de VS Code:
```
mkdir gearpilot
cd gearpilot
python -m venv venv
venv\Scripts\activate
pip install flask
pip freeze > requirements.txt
```
- `venv` es una "caja" privada de librerías para este proyecto. Cuando esté activa verán `(venv)` al inicio de la línea.
- Si Windows dice que los scripts están deshabilitados, ejecuten una vez: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` y repitan la activación.

**3. Git y GitHub**
1. Instalen Git desde git-scm.com (todo en "Next").
2. Creen una cuenta en github.com. **Uno** de ustedes crea un repositorio privado llamado `gearpilot` e invita al otro (*Settings → Collaborators*).
3. En la terminal: `git init`, `git add .`, `git commit -m "inicio"`, y sigan las 3 líneas que GitHub muestra para conectar y hacer `push`.
4. Creen un archivo `.gitignore` con estas líneas: `venv/`, `*.db`, `__pycache__/`. La base de datos **no** se sube a GitHub.

**4. PythonAnywhere**
1. Creen una cuenta en pythonanywhere.com, plan **Beginner** (gratis). El nombre de usuario será parte de su enlace.
2. Pestaña **Web → Add a new web app → Flask** → elijan la versión de Python más reciente disponible.
3. Cada vez que cambien código: en la pestaña **Consoles → Bash** hacen `git pull` y en **Web** pulsan el botón verde **Reload**.

## 2. Diseño de la base de datos

Siete tablas y una opcional. Cada fila de una tabla es un registro (como un objeto) y las columnas son sus atributos.

```
clientes 1───< vehiculos 1───< ordenes_trabajo >───1 mecanicos
    │                │               │
    │                └───< citas     ├───< cotizacion_items
    │                                └───< pagos
administradores (aparte, solo para iniciar sesión y administrar)
```
`1───<` significa "uno a muchos": un cliente tiene varios vehículos.

| Tabla | Columnas | Explicación sencilla |
|---|---|---|
| **administradores** | id, documento, nombre, telefono, correo, contrasena | Quienes entran al panel. La contraseña se guarda **cifrada** (hash), nunca en texto. |
| **clientes** | id, documento (único), nombre, telefono, correo, contrasena | Dueños de vehículos. El documento no se repite, así se busca "cliente existente". |
| **mecanicos** | id, documento, nombre, telefono, correo, especialidad, es_jefe (0 o 1) | `MecanicoJefe` es un mecánico con `es_jefe = 1`. Así no necesitan una tabla más. |
| **vehiculos** | id, placa (única), marca, modelo, anio, cliente_id | `cliente_id` apunta al dueño. Es la "llave foránea": un número que enlaza con otra tabla. |
| **ordenes_trabajo** | id, vehiculo_id, mecanico_id, motivo, diagnostico, estado, respuesta_cotizacion, fecha_ingreso, fecha_cierre | El centro del sistema. `estado`: Diagnóstico, Cotizado, En proceso, Completado. `respuesta_cotizacion`: Pendiente, Aceptada, Rechazada. |
| **cotizacion_items** | id, orden_id, concepto, tipo (Repuesto o Servicio), valor | Cada línea de la cotización. El **total no se guarda**: se calcula sumando (es su `calcularTotal()`), así nunca queda desactualizado. |
| **pagos** | id, orden_id, monto, metodo, fecha | Una orden puede tener varios pagos (anticipo y saldo). El saldo pendiente = total − suma de pagos. |
| **citas** | id, vehiculo_id, fecha, hora, motivo | Restricción `UNIQUE(fecha, hora)`: si ya existe, la cita se rechaza. Los horarios posibles (8:00, 9:00...) se definen en una lista en el código. |

### Cambios recomendados a su ficha (confírmenlo con el profesor)

1. **Herencia:** en la ficha, `Mecanico` y `MecanicoJefe` heredan de `Administrador`. Una prueba "ES UN" falla aquí, porque un mecánico no es un administrador y heredaría la contraseña y los permisos de administración. Propuesta más sólida: una clase base **`Persona`** (documento, nombre, telefono, correo) de la que heredan `Administrador`, `Cliente` y `Mecanico`, y `MecanicoJefe` hereda de `Mecanico`. Sigue siendo herencia, solo con mejor justificación. Si el profesor ya aprobó la original, pueden mantenerla; la base de datos no cambia.
2. **"Una OrdenTrabajo tiene un Pago":** su propio prototipo muestra anticipo y saldo, o sea varios pagos. Usen uno a muchos.
3. **"Un Cliente tiene un vehículo":** en la práctica son varios. Usen uno a muchos.

## 3. Módulos y orden

Cada módulo termina con pantalla + lógica + datos funcionando y **publicado**.

| # | Módulo | Qué contiene | Por qué en este lugar |
|---|---|---|---|
| 0 | **Hola publicado** | Una página "Hola" en internet | Resuelve primero lo más desconocido (el despliegue). |
| 1 | **Login y panel del administrador** | Iniciar sesión, clase `Administrador` con el setter de documento (3 intentos) | Es una sola tabla y una sola clase. Aprenden formularios, SQL y sesión con lo mínimo. |
| 2 | **Clientes y vehículos** | Buscar por placa + documento, registrar cliente nuevo, registrar vehículo, ver historial | Reutiliza lo del Módulo 1 y agrega la primera relación entre tablas. |
| 3 | **Mecánicos y órdenes de trabajo** | Registrar mecánicos, crear orden, asignar mecánico, diagnóstico, cambiar estado | Une tres tablas (vehículo, mecánico, orden). Necesita que el 2 funcione. |
| 4 | **Cotización y pagos** | Ítems de cotización, total calculado, aceptar o rechazar, anticipo y saldo | Es el más delicado (cálculos y reglas). Necesita órdenes existentes. |
| 5 | **Portal del cliente (solo consulta)** | Login de cliente; ver historial, pagos y mecánico | Solo lee datos que ya existen, por eso va al final. |
| 6 | **Citas** *(opcional)* | Agendar con validación de disponibilidad y horarios alternativos | Nada depende de él, así que es lo primero que se puede quitar. |

## 4. Cómo organizar el código

Usen **clases para las reglas del negocio y funciones para hablar con la base de datos y las páginas**:

- `modelos.py`: las clases (`Persona`, `Administrador`, `Mecanico`...) con atributos privados, getters/setters y validaciones. Aquí se ve la POO que van a sustentar.
- `db.py`: funciones sueltas como `buscar_vehiculo(placa)` o `crear_orden(...)`. Son SQL envuelto en funciones simples.
- `app.py`: las rutas de Flask, también funciones.

**Por qué esta mezcla:** Flask está hecho con funciones, así que forzar todo en clases lo complicaría. En cambio las reglas ("documento solo números, 3 intentos") calzan naturalmente como setters en una clase, que es lo que su ficha pide. No usen un ORM (SQLAlchemy): es otra cosa más que aprender; usen SQL directo con `sqlite3`.

Estructura de carpetas:
```
gearpilot/
├── app.py            rutas
├── db.py             funciones SQL
├── modelos.py        clases POO
├── schema.sql        CREATE TABLE de todas las tablas
├── init_db.py        crea gearpilot.db desde schema.sql
├── requirements.txt
├── templates/        base.html + una página por vista
└── static/estilos.css
```

**Cómo aprovechar su prototipo:** el bloque `<style>` de `index.html` pasa a `static/estilos.css`. El menú, el encabezado y las tarjetas pasan a `templates/base.html` (se escriben una vez). Cada sección (Panel, Ingreso, Orden, Cliente) pasa a su propio archivo `.html`, y los datos fijos ("KXR 482", "$433.000") se reemplazan por `{{ variables }}` y bucles. El selector Administrador/Cliente se reemplaza por el login.

## 5. Ruta semana a semana

Responsables: **N** = Naren, **C** = Camilo. Cada tarea ≤ 2 h por persona.

### Semana 1: Módulo 0 + base de datos (≈ 8 h combinadas)

**Módulo 0: Hola publicado**

| Tarea | Resp. | Tiempo |
|---|---|---|
| T1 Instalar Python, VS Code + extensiones, verificar `python --version` | N y C (cada uno en su PC) | 1,5 h |
| T2 Crear repositorio en GitHub, instalar Git, primer `push` | C | 1 h |
| T3 Carpeta `gearpilot`, entorno virtual, Flask, `app.py` con "Hola" y correrlo con `flask run` | N | 1,5 h |
| T4 Crear cuenta en PythonAnywhere, crear web app Flask, clonar el repo con `git clone` en la consola Bash, apuntar el archivo WSGI a `app.py`, Reload | C (N acompaña) | 2 h |
| T5 Escribir `schema.sql` con las 8 tablas de la sección 2 y `init_db.py`; ejecutarlo en local y en el servidor | N | 2 h |
| T6 Practicar 5 consultas SQL en el visor SQLite (INSERT, SELECT, WHERE, JOIN) | C | 1,5 h |

- **Debe funcionar:** abrir `https://SUUSUARIO.pythonanywhere.com` desde el celular y ver "Hola". Existe un `gearpilot.db` con las tablas.
- **Cómo comprobarlo:** pidan a otra persona que abra el enlace.
- **Error común:** cambian el código, hacen `git pull` y no ven nada nuevo, porque olvidaron pulsar **Reload** en la pestaña Web. Segundo error: la ruta del archivo WSGI apunta a la carpeta equivocada; si sale "Something went wrong", lean el *error log* en esa pestaña.

### Semana 2: Módulos 1 y 2

**Módulo 1: Login y panel del administrador** (≈ 3 h en total)

| Tarea | Resp. | Tiempo |
|---|---|---|
| T1 Mover estilos y plantilla base del prototipo a `static/` y `templates/base.html` | C | 1,5 h |
| T2 Clase `Administrador` (hereda de `Persona`) con setter de `documento` (solo dígitos, 3 intentos) | N | 2 h |
| T3 Ruta `/login`, comparar contraseña cifrada con `werkzeug.security`, guardar en `session`, página del Panel | N y C | 2 h |

- **Debe funcionar:** entrar con un administrador creado en la base y ver el panel. Una contraseña incorrecta muestra aviso. Escribir letras en el documento muestra el aviso "Intento X de 3".
- **Comprobar:** probar usuario correcto, contraseña mala, y documento `12ab` tres veces.
- **Error común:** guardar la contraseña en texto plano o compararla directamente. Usen `generate_password_hash` al crear y `check_password_hash` al validar. Otro: olvidar `app.secret_key`, sin el cual `session` falla.

**Módulo 2: Clientes y vehículos** (≈ 5 h)

| Tarea | Resp. | Tiempo |
|---|---|---|
| T1 Funciones en `db.py`: `buscar_cliente(documento)`, `buscar_vehiculo(placa)`, `crear_cliente`, `crear_vehiculo` | N | 2 h |
| T2 Página "Ingreso de vehículo": formulario placa + documento, resultado "ya registrado" con historial | C | 2 h |
| T3 Formulario de cliente y vehículo nuevos cuando la placa no existe | N | 2 h |
| T4 Clases `Cliente` y `Vehiculo` con validaciones; revisar que `modelos.py` las use | C | 1 h |

- **Debe funcionar:** buscar una placa existente muestra al dueño; una nueva abre el registro y luego aparece en el sistema.
- **Comprobar:** registrar un vehículo, reiniciar el servidor y buscarlo de nuevo (verifica que se guardó de verdad).
- **Error común:** olvidar `conexion.commit()` después de un `INSERT`: el programa parece funcionar pero el dato no queda guardado. Segundo: armar el SQL pegando texto (`"... WHERE placa='" + placa + "'"`); usen siempre `?` como marcador: `execute("... WHERE placa = ?", (placa,))`. Evita errores y ataques.

### Semana 3: Módulos 3 y 4

**Módulo 3: Mecánicos y órdenes** (≈ 4 h)

| Tarea | Resp. | Tiempo |
|---|---|---|
| T1 Clases `Mecanico`, `MecanicoJefe` y `OrdenTrabajo` | N | 1,5 h |
| T2 Registro y listado de mecánicos | C | 1,5 h |
| T3 "Crear orden de trabajo" (vehículo + mecánico + motivo) y tabla de órdenes activas en el Panel | N | 2 h |
| T4 Pantalla de la orden: diagnóstico y botón de cambio de estado ("En proceso" / "Completado") | C | 2 h |

- **Debe funcionar:** crear una orden, ver que aparece en el Panel con su mecánico, y cambiar su estado.
- **Comprobar:** el contador "Órdenes en proceso" del panel cambia al completar una (cálculo con `SELECT COUNT(*)`, no un número fijo).
- **Error común:** unir tablas mal y ver filas repetidas o ninguna. Usen `JOIN ... ON ordenes.vehiculo_id = vehiculos.id` y prueben con 2 órdenes antes de seguir.

**Módulo 4: Cotización y pagos** (≈ 4 h)

| Tarea | Resp. | Tiempo |
|---|---|---|
| T1 Agregar y listar ítems de cotización por orden | C | 2 h |
| T2 `calcularTotal()` en `OrdenTrabajo`: suma de ítems | N | 1 h |
| T3 Aceptar / rechazar cotización | N | 1 h |
| T4 Registrar pagos (anticipo, saldo) y mostrar saldo pendiente | C | 2 h |

- **Debe funcionar:** la orden de ejemplo da $433.000, con anticipo de $200.000 y saldo de $233.000.
- **Comprobar:** agregar un ítem y verificar que total y saldo se actualizan solos.
- **Error común:** guardar el total en una columna y que quede desactualizado. Calculen siempre con `SUM`. Otro: no validar que el pago no supere el saldo.

### Semana 4: Módulo 5 + cierre

**Módulo 5: Portal del cliente, solo consulta** (≈ 4 h)

| Tarea | Resp. | Tiempo |
|---|---|---|
| T1 Login de cliente (documento + contraseña) y ruta protegida | N | 1,5 h |
| T2 Pantallas "Vehículos" (historial de arreglos), "Pagos" (historial y mecánico asignado) | C | 2 h |
| T3 Filtrar: un cliente solo ve **sus** datos | N | 1 h |

- **Debe funcionar:** un cliente entra y ve su historial y pagos, y no los de otro.
- **Comprobar:** entrar con dos clientes distintos y confirmar que ven datos diferentes.
- **Error común:** mostrar datos por `id` en la URL sin verificar al dueño (cambiando `/orden/5` a `/orden/6` verían otra). Siempre filtren por el cliente de la sesión.

**Cierre (≈ 4 h)**

| Tarea | Resp. | Tiempo |
|---|---|---|
| Probar todo el flujo completo en el sitio publicado, corregir errores | N y C | 2 h |
| Cargar datos de ejemplo y preparar la sustentación (cada uno explica las clases y archivos que escribió) | N y C | 2 h |

**Módulo 6 (Citas)** solo si sobra tiempo: ver sección 6.

## 6. Qué recortar

Con 32 horas combinadas el proyecto completo **no cabe**, y menos con la curva de aprendizaje. Orden de recorte:

1. **Módulo 6, Citas** (disponibilidad y alternativas): fuera. Dejen solo la ficha en el informe como trabajo futuro.
2. **Vista móvil del cliente** (lo que ustedes eligieron recortar primero): si el Módulo 4 no está terminado al final de la semana 3, el Módulo 5 se reduce a una sola página de consulta con login, sin las tres pestañas.
3. **Mecánico jefe**: queda como una casilla `es_jefe`, sin pantallas propias.
4. **Panel**: las estadísticas ("Citas de hoy", etc.) pueden ser solo dos contadores reales.

Si en realidad son 4 horas combinadas por semana: el alcance realista es Módulos 0, 1, 2 y 3. Avísennos para rehacer la ruta.

## 7. Cómo seguir trabajando con este plan

Pidan el código **un módulo a la vez** ("vamos con el Módulo 1"). Para cada uno les entregamos el código por archivos, con explicaciones de las partes importantes, para que cualquiera de los dos pueda sustentarlas.
