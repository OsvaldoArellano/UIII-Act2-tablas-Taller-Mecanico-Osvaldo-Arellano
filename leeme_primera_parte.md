
1. Crear la carpeta del proyecto

En tu ubicación preferida (ej. Documentos):

Windows / PowerShell:

mkdir UIII_Taller_Mecanico_0446
cd UIII_Taller_Mecanico_0446


Linux / macOS:

mkdir -p UIII_Taller_Mecanico_0446
cd UIII_Taller_Mecanico_0446

2. Abrir VS Code sobre la carpeta

Desde la misma carpeta en terminal:

code .


(o en Windows PowerShell también code .)

Alternativa: abrir VS Code y usar Archivo > Abrir carpeta... y seleccionar UIII_Taller_Mecanico_0446.

3. Abrir terminal en VS Code

Dentro de VS Code:

Atajo: Ctrl + \ (en muchas teclas es Ctrl+ ) — Ctrl+` (la tecla de la tilde invertida).

O menú: Ver > Terminal.

4. Crear carpeta entorno virtual .venv desde terminal de VS Code

Dentro de la carpeta del proyecto ejecuta:

Windows (PowerShell):

python -m venv .venv


Linux / macOS:

python3 -m venv .venv


Esto crea la carpeta .venv con el entorno virtual.

5. Activar el entorno virtual

Windows PowerShell:

.venv\Scripts\Activate.ps1
# o (PowerShell) . .venv\Scripts\Activate.ps1


Windows (cmd):

.venv\Scripts\activate.bat


Linux / macOS:

source .venv/bin/activate


Verifica con python --version o which python / Get-Command python.

6. Activar intérprete de Python en VS Code

Ctrl+Shift+P → escribe Python: Select Interpreter.

Selecciona la que apunta a tu .venv (algo como .venv\Scripts\python o .venv/bin/python).
Esto hace que VS Code use ese intérprete para ejecutar y depurar.

7. Instalar Django

Con el entorno activado:

pip install --upgrade pip
pip install django


(Confirma con django-admin --version).

8. Crear proyecto backend_Taller sin duplicar carpeta

Para evitar crear una carpeta extra, desde UIII_Taller_Mecanico_0446 ejecuta:

django-admin startproject backend_Taller .


Observación: el . al final indica "crear el proyecto en la carpeta actual" — así evitas backend_Taller/backend_Taller.

Estructura mínima tras esto:

UIII_Taller_Mecanico_0446/
  manage.py
  backend_Taller/
    __init__.py
    settings.py
    urls.py
    wsgi.py
    asgi.py

9. Ejecutar servidor en el puerto 8446

Within the activated venv and project folder:

python manage.py runserver 8446


Esto iniciará el servidor en http://127.0.0.1:8446/.

Si quieres permitir conexiones externas (red local):

python manage.py runserver 0.0.0.0:8446

10. Copiar y pegar el link en el navegador

Abre tu navegador y pega:

http://127.0.0.1:8446/


ó

http://localhost:8446/

11. Crear aplicación app_Taller

Con el entorno activado y en la raíz del proyecto:

python manage.py startapp app_Taller


Esto crea app_Taller/ con models.py, views.py, admin.py, apps.py, migrations/, etc.

12. Aquí el models.py (usa el que ya diste)

Copia tu código dentro de app_Taller/models.py (reemplaza su contenido). Por ejemplo:

# app_Taller/models.py
from django.db import models

# ==========================================
# MODELO: CLIENTE
# ==========================================
class Cliente(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    telefono = models.CharField(max_length=20)
    email = models.EmailField(unique=True)
    rfc = models.CharField(max_length=20, unique=True)
    direccion = models.CharField(max_length=200)
    fecha_registro = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"

# ==========================================
# MODELO: SERVICIO  (Pendiente para usar más adelante)
# ==========================================
class Servicio(models.Model):
    nombre_servicio = models.CharField(max_length=100)
    descripcion = models.TextField(blank=True, null=True)
    precio_base = models.DecimalField(max_digits=10, decimal_places=2)
    tiempo_estimado_horas = models.DecimalField(max_digits=5, decimal_places=2)
    aplica_garantia = models.BooleanField(default=False)
    fecha_actualizacion = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.nombre_servicio

# ==========================================
# MODELO: VEHICULO (Pendiente para usar más adelante)
# ==========================================
class Vehiculo(models.Model):
    cliente = models.ForeignKey(
        Cliente, on_delete=models.CASCADE, related_name="vehiculos"
    )
    servicios = models.ManyToManyField(
        Servicio, related_name="vehiculos", blank=True
    )
    matricula = models.CharField(max_length=20, unique=True)
    marca = models.CharField(max_length=50)
    modelo = models.CharField(max_length=50)
    anio = models.PositiveIntegerField()
    kilometraje = models.PositiveIntegerField()
    color = models.CharField(max_length=30)

    def __str__(self):
        return f"{self.marca} {self.modelo} ({self.matricula})"

12.5 Procedimiento para realizar migraciones (makemigrations y migrate)

Primero agrega la app en INSTALLED_APPS (ver 25). Luego:

python manage.py makemigrations app_Taller
python manage.py migrate


Esto crea tablas en la base de datos (por defecto SQLite).

13. Primero trabajamos con el MODELO: CLIENTE

Nos enfocaremos en CRUD de Cliente. Los modelos Servicio y Vehiculo quedan pendientes.

14. views.py de app_Taller — funciones (inicio_taller, agregar_cliente, actualizar_cliente, realizar_actualizacion_cliente, borrar_cliente)

Crea o reemplaza app_Taller/views.py con:

# app_Taller/views.py
from django.shortcuts import render, redirect, get_object_or_404
from .models import Cliente
from django.urls import reverse
from django.utils import timezone

def inicio_taller(request):
    total_clientes = Cliente.objects.count()
    context = {
        "total_clientes": total_clientes,
    }
    return render(request, "inicio.html", context)

def agregar_cliente(request):
    if request.method == "POST":
        # Sin forms.py: coger campos directamente
        nombre = request.POST.get("nombre", "").strip()
        apellido = request.POST.get("apellido", "").strip()
        telefono = request.POST.get("telefono", "").strip()
        email = request.POST.get("email", "").strip()
        rfc = request.POST.get("rfc", "").strip()
        direccion = request.POST.get("direccion", "").strip()

        Cliente.objects.create(
            nombre=nombre,
            apellido=apellido,
            telefono=telefono,
            email=email,
            rfc=rfc,
            direccion=direccion,
            fecha_registro=timezone.now()
        )
        return redirect("ver_cliente")
    return render(request, "cliente/agregar_cliente.html")

def ver_cliente(request):
    clientes = Cliente.objects.all().order_by("-fecha_registro")
    context = {"clientes": clientes}
    return render(request, "cliente/ver_cliente.html", context)

def actualizar_cliente(request, cliente_id):
    cliente = get_object_or_404(Cliente, pk=cliente_id)
    context = {"cliente": cliente}
    return render(request, "cliente/actualizar_cliente.html", context)

def realizar_actualizacion_cliente(request, cliente_id):
    cliente = get_object_or_404(Cliente, pk=cliente_id)
    if request.method == "POST":
        cliente.nombre = request.POST.get("nombre", cliente.nombre).strip()
        cliente.apellido = request.POST.get("apellido", cliente.apellido).strip()
        cliente.telefono = request.POST.get("telefono", cliente.telefono).strip()
        cliente.email = request.POST.get("email", cliente.email).strip()
        cliente.rfc = request.POST.get("rfc", cliente.rfc).strip()
        cliente.direccion = request.POST.get("direccion", cliente.direccion).strip()
        cliente.save()
        return redirect("ver_cliente")
    # Si no es POST, volver al form
    return redirect("actualizar_cliente", cliente_id=cliente.id)

def borrar_cliente(request, cliente_id):
    cliente = get_object_or_404(Cliente, pk=cliente_id)
    if request.method == "POST":
        cliente.delete()
        return redirect("ver_cliente")
    context = {"cliente": cliente}
    return render(request, "cliente/borrar_cliente.html", context)


Notas:

No se valida la entrada (solicitaste no validar).

Usamos vistas basadas en funciones simples.

15. Crear carpeta templates dentro de app_Taller

Estructura:

app_Taller/
  templates/
    base.html
    header.html
    navbar.html
    footer.html
    inicio.html
    cliente/
      agregar_cliente.html
      ver_cliente.html
      actualizar_cliente.html
      borrar_cliente.html


Crea esa carpeta y archivos.

16 & 17. base.html con Bootstrap (CSS & JS) y estructura

app_Taller/templates/base.html:

<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{% block title %}Taller Mecánico{% endblock %}</title>

    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Bootstrap Icons -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css">

    <style>
      /* Colores suaves y diseño moderno */
      :root{
        --primary-soft: #4b94b3;
        --accent-soft: #f0f7fb;
      }
      body{
        background-color: var(--accent-soft);
      }
      .footer-fixed{
        position: fixed;
        bottom: 0;
        left: 0;
        width: 100%;
      }
      .card-soft{
        border-radius: 12px;
        box-shadow: 0 4px 14px rgba(0,0,0,0.06);
      }
      .main-container{
        padding-bottom: 80px; /* espacio por footer fijo */
      }
    </style>

    {% block extra_head %}{% endblock %}
  </head>
  <body>
    {% include "header.html" %}
    {% include "navbar.html" %}

    <main class="container main-container mt-4">
      {% block content %}{% endblock %}
    </main>

    {% include "footer.html" %}

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
  </body>
</html>

18. navbar.html con las opciones y iconos (no en submenus)

app_Taller/templates/navbar.html:

<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
  <div class="container">
    <a class="navbar-brand d-flex align-items-center" href="{% url 'inicio_taller' %}">
      <i class="bi bi-gear-fill fs-4 me-2" style="color:var(--primary-soft)"></i>
      <span class="fw-bold">Sistema de Administración Taller Mecánico</span>
    </a>

    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio_taller' %}"><i class="bi bi-house-door-fill"></i> Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"> <i class="bi bi-people-fill"></i> Cliente</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_cliente' %}">Agregar cliente</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_cliente' %}">Ver cliente</a></li>
            <li><a class="dropdown-item" href="#">Actualizar cliente</a></li>
            <li><a class="dropdown-item" href="#">Borrar cliente</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-car-front-fill"></i> Vehículo</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar vehiculo</a></li>
            <li><a class="dropdown-item" href="#">Ver vehiculo</a></li>
            <li><a class="dropdown-item" href="#">Actualizar vehiculo</a></li>
            <li><a class="dropdown-item" href="#">Borrar vehiculo</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-tools"></i> Servicios</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar servicios</a></li>
            <li><a class="dropdown-item" href="#">Ver servicios</a></li>
            <li><a class="dropdown-item" href="#">Actualizar servicios</a></li>
            <li><a class="dropdown-item" href="#">Borrar servicios</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>

19. footer.html con derechos de autor y fecha y texto fijo

app_Taller/templates/footer.html:

<footer class="footer-fixed bg-white border-top">
  <div class="container py-2 d-flex justify-content-between">
    <div>© {{ now|date:"Y" }} Taller Mecánico</div>
    <div>Creado por Alumno Osvaldo Arellano De La Cruz, Cbtis 128</div>
  </div>
</footer>


Nota: Para que now funcione, activa django.template.context_processors.request (viene por defecto). Alternativamente usa from django.utils import timezone y pásalo en vistas; pero plantilla con {{ now|date:"Y" }} suele funcionar si habilitas django.template.context_processors.request o registra django.template.context_processors.now. Si no funciona, reemplaza con 2025 u otra forma.

20. inicio.html con info y una imagen de Taller Mecanico

app_Taller/templates/inicio.html:

{% extends "base.html" %}
{% block title %}Inicio - Taller{% endblock %}
{% block content %}
<div class="row">
  <div class="col-md-8">
    <div class="card card-soft p-4">
      <h3>Bienvenido al Sistema del Taller</h3>
      <p>Este sistema permite registrar clientes, vehículos y servicios del taller mecánico.</p>
      <p>Total de clientes registrados: <strong>{{ total_clientes }}</strong></p>
    </div>
  </div>
  <div class="col-md-4">
    <div class="card card-soft p-2">
      <img src="https://www.cinepolis.com/content/dam/cinepolis-com/hero/hero-cinepolis.jpg" alt="Taller Mecanico" class="img-fluid rounded">
      <small class="d-block mt-2 text-muted">Imagen tomada desde la red sobre Taller Mecanico</small>
    </div>
  </div>
</div>
{% endblock %}


(Sustituye la URL si está protegida o cambia la imagen por otra si es necesario.)

21. Subcarpeta cliente dentro de templates (ya indicada arriba)

app_Taller/templates/cliente/ con los archivos de CRUD.

22. Templates cliente/*

agregar_cliente.html:

{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-4">
  <h4>Agregar Cliente</h4>
  <form method="post">{% csrf_token %}
    <div class="mb-2">
      <label class="form-label">Nombre</label>
      <input name="nombre" class="form-control" required>
    </div>
    <div class="mb-2">
      <label class="form-label">Apellido</label>
      <input name="apellido" class="form-control" required>
    </div>
    <div class="mb-2">
      <label class="form-label">Teléfono</label>
      <input name="telefono" class="form-control">
    </div>
    <div class="mb-2">
      <label class="form-label">Email</label>
      <input name="email" type="email" class="form-control">
    </div>
    <div class="mb-2">
      <label class="form-label">RFC</label>
      <input name="rfc" class="form-control">
    </div>
    <div class="mb-2">
      <label class="form-label">Dirección</label>
      <input name="direccion" class="form-control">
    </div>
    <button class="btn btn-primary">Guardar</button>
    <a href="{% url 'ver_cliente' %}" class="btn btn-secondary">Volver</a>
  </form>
</div>
{% endblock %}


ver_cliente.html (tabla con botones ver, editar, borrar):

{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-3">
  <div class="d-flex justify-content-between align-items-center mb-3">
    <h4>Clientes</h4>
    <a href="{% url 'agregar_cliente' %}" class="btn btn-success">Agregar cliente</a>
  </div>

  <table class="table table-striped">
    <thead>
      <tr>
        <th>Nombre</th><th>Email</th><th>Teléfono</th><th>RFC</th><th>Dirección</th><th>Acciones</th>
      </tr>
    </thead>
    <tbody>
      {% for c in clientes %}
      <tr>
        <td>{{ c.nombre }} {{ c.apellido }}</td>
        <td>{{ c.email }}</td>
        <td>{{ c.telefono }}</td>
        <td>{{ c.rfc }}</td>
        <td>{{ c.direccion }}</td>
        <td>
          <a class="btn btn-sm btn-info" href="{% url 'actualizar_cliente' c.id %}">Editar</a>
          <a class="btn btn-sm btn-danger" href="{% url 'borrar_cliente' c.id %}">Borrar</a>
        </td>
      </tr>
      {% empty %}
      <tr><td colspan="6">No hay clientes registrados.</td></tr>
      {% endfor %}
    </tbody>
  </table>
</div>
{% endblock %}


actualizar_cliente.html:

{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-4">
  <h4>Actualizar Cliente</h4>
  <form method="post" action="{% url 'realizar_actualizacion_cliente' cliente.id %}">{% csrf_token %}
    <div class="mb-2"><label>Nombre</label><input name="nombre" value="{{ cliente.nombre }}" class="form-control"></div>
    <div class="mb-2"><label>Apellido</label><input name="apellido" value="{{ cliente.apellido }}" class="form-control"></div>
    <div class="mb-2"><label>Teléfono</label><input name="telefono" value="{{ cliente.telefono }}" class="form-control"></div>
    <div class="mb-2"><label>Email</label><input name="email" value="{{ cliente.email }}" class="form-control"></div>
    <div class="mb-2"><label>RFC</label><input name="rfc" value="{{ cliente.rfc }}" class="form-control"></div>
    <div class="mb-2"><label>Dirección</label><input name="direccion" value="{{ cliente.direccion }}" class="form-control"></div>
    <button class="btn btn-primary">Guardar cambios</button>
    <a href="{% url 'ver_cliente' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}


borrar_cliente.html:

{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-4">
  <h4>Eliminar Cliente</h4>
  <p>¿Seguro que quieres eliminar a <strong>{{ cliente.nombre }} {{ cliente.apellido }}</strong>?</p>
  <form method="post">{% csrf_token %}
    <button type="submit" class="btn btn-danger">Sí, eliminar</button>
    <a href="{% url 'ver_cliente' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}

23. No utilizar forms.py

(Ya cumplido: las vistas usan request.POST directo.)

24. urls.py en app_Taller (rutas CRUD)

Crea app_Taller/urls.py con:

# app_Taller/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_taller, name='inicio_taller'),

    # Clientes
    path('cliente/agregar/', views.agregar_cliente, name='agregar_cliente'),
    path('cliente/ver/', views.ver_cliente, name='ver_cliente'),
    path('cliente/actualizar/<int:cliente_id>/', views.actualizar_cliente, name='actualizar_cliente'),
    path('cliente/realizar_actualizacion/<int:cliente_id>/', views.realizar_actualizacion_cliente, name='realizar_actualizacion_cliente'),
    path('cliente/borrar/<int:cliente_id>/', views.borrar_cliente, name='borrar_cliente'),
]

25. Agregar app_Taller en settings.py de backend_Taller

En backend_Taller/settings.py localiza INSTALLED_APPS y añade 'app_Taller',:

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    ...
    'app_Taller',
]


Asegúrate de que DIRS en TEMPLATES permita cargar plantillas desde apps (por defecto Django busca en app/templates). Si quieres templates globales en BASE_DIR / 'templates' agrega:

TEMPLATES = [
    {
        ...
        'DIRS': [BASE_DIR / 'templates'],
        ...
    },
]

26. Configuraciones en backend_Taller/urls.py para enlazar con app_Taller

En backend_Taller/urls.py:

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Taller.urls')),  # rutas de la app (inicio en /)
]


Para archivos estáticos en desarrollo (opcional):

from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)

27. Registrar modelos en admin.py y volver a migrar

app_Taller/admin.py:

from django.contrib import admin
from .models import Cliente  # por ahora sólo Cliente

@admin.register(Cliente)
class ClienteAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'apellido', 'email', 'telefono', 'rfc', 'fecha_registro')
    search_fields = ('nombre', 'apellido', 'email', 'rfc')


Luego:

python manage.py makemigrations app_Taller
python manage.py migrate


Crea superusuario para acceder a admin:

python manage.py createsuperuser
# sigue prompts: email, password...

27 (repetido): Por ahora solo trabajar con “cliente”

Mantén Servicio y Vehiculo en models.py pero no los usamos en templates ni vistas todavía.

28. Estética: colores suaves, páginas sencillas

Usamos Bootstrap + variables CSS en base.html.

Mantén clases bootstrap: card, container, table, btn.

No hagas validaciones JS ni server-side (solicitado).

29. Estructura completa inicial de carpetas y archivos (recomendada)
UIII_Taller_Mecanico_0446/
├─ .venv/
├─ manage.py
├─ backend_Taller/
│  ├─ settings.py
│  ├─ urls.py
│  └─ ...
├─ app_Taller/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ header.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ cliente/
│  │     ├─ agregar_cliente.html
│  │     ├─ ver_cliente.html
│  │     ├─ actualizar_cliente.html
│  │     └─ borrar_cliente.html
│  ├─ static/   (opcional para css/js)
│  ├─ models.py
│  ├─ views.py
│  ├─ urls.py
│  └─ admin.py


Crea carpetas vacías static/ y agrega static/css/style.css si quieres estilos propios.

30. Proyecto totalmente funcional (pasos finales de verificación)

Activar .venv.

python manage.py makemigrations y python manage.py migrate.

python manage.py createsuperuser (opcional).

python manage.py runserver 8446.

Ir a http://127.0.0.1:8446/ — deberías ver la página de inicio.

Ir a http://127.0.0.1:8446/cliente/agregar/ para agregar cliente.

Ver clientes en http://127.0.0.1:8446/cliente/ver/.

31. Finalmente ejecutar servidor en el puerto 8446

(Ya visto en el paso 9)

python manage.py runserver 8446

Extras / consejos rápidos

Si no ves cambios en plantillas borra cache del navegador o reinicia el servidor.

Si email o rfc duplicados lanzan error al crear, recuerda que están marcados como unique=True — evitar duplicados en pruebas.

Para CSS propio: colocar archivos en app_Taller/static/app_Taller/css/ y en base.html incluir {% load static %} y <link href="{% static 'app_Taller/css/style.css' %}" rel="stylesheet">.

Para pasar la fecha en footer con now, puedes usar {% now "Y" %} directamente: © {% now "Y" %}.
