# UIII-Act2-Segunda-Parte-Producto
crear prompt con IA  para la tabla de Producto


##PARTE 1: Crear la estructura del proyecto

1️⃣## Abre VS Code


2️⃣## Abre una terminal en VS Code.


3️⃣## Crea el proyecto Django y la app:

django-admin startproject backend_Farmacia UIII_Farmacia_0777
cd UIII_Farmacia_0777
python manage.py startapp app_farmacia


##La estructura quedará así:

UIII_Farmacia_0777/
│
├── backend_Farmacia/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── app_farmacia/
│   ├── migrations/
│   ├── templates/
│   │   ├── producto/
│   │   │   ├── agregar_producto.html
│   │   │   ├── ver_producto.html
│   │   │   ├── actualizar_producto.html
│   │   │   ├── borrar_tabla.html
│   │   └── navbar.html
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views.py
│
└── manage.py

##PARTE 2: Configuración del proyecto

Abre backend_Farmacia/settings.py y agrega la app:

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_farmacia',
]


También asegúrate de tener configurado correctamente las plantillas:

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'app_farmacia' / 'templates'],
        'APP_DIRS': True,
        ...
    },
]



PARTE 3: Modelo Producto en models.py
from django.db import models

    class Proveedor(models.Model):
    nombre = models.CharField(max_length=100)
    telefono = models.CharField(max_length=20)

    def __str__(self):
        return self.nombre


    class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    tipo_producto = models.CharField(max_length=50)
    fecha_caducidad = models.DateField()
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    proveedor = models.ForeignKey(Proveedor, on_delete=models.CASCADE, related_name='productos')

    def __str__(self):
        return f"{self.nombre} - {self.tipo_producto}"

🧩 ##PARTE 4: Migraciones

En la terminal ejecuta:

python manage.py makemigrations
python manage.py migrate



##PARTE 5: Registrar modelos en admin.py

      from django.contrib import admin
      from .models import Producto, Proveedor
        dmin.site.register(Proveedor)
        admin.site.register(Producto)
    
##PARTE 6: Vistas CRUD (views.py)

    from django.shortcuts import render, redirect, get_object_or_404
    from .models import Producto, Proveedor

# Crear producto
    def agregar_producto(request):
    if request.method == 'POST':
        nombre = request.POST['nombre']
        tipo_producto = request.POST['tipo_producto']
        fecha_caducidad = request.POST['fecha_caducidad']
        precio = request.POST['precio']
        proveedor_id = request.POST['proveedor']
        proveedor = Proveedor.objects.get(id=proveedor_id)
        Producto.objects.create(
            nombre=nombre,
            tipo_producto=tipo_producto,
            fecha_caducidad=fecha_caducidad,
            precio=precio,
            proveedor=proveedor
        )
        return redirect('ver_producto')
    proveedores = Proveedor.objects.all()
    return render(request, 'producto/agregar_producto.html', {'proveedores': proveedores})


# Leer (ver productos)
    def ver_producto(request):
    productos = Producto.objects.all()
    return render(request, 'producto/ver_producto.html', {'productos': productos})


# Actualizar producto
    def actualizar_producto(request, id):
    producto = get_object_or_404(Producto, id=id)
    proveedores = Proveedor.objects.all()
    if request.method == 'POST':
        producto.nombre = request.POST['nombre']
        producto.tipo_producto = request.POST['tipo_producto']
        producto.fecha_caducidad = request.POST['fecha_caducidad']
        producto.precio = request.POST['precio']
        proveedor_id = request.POST['proveedor']
        producto.proveedor = Proveedor.objects.get(id=proveedor_id)
        producto.save()
        return redirect('ver_producto')
    return render(request, 'producto/actualizar_producto.html', {'producto': producto, 'proveedores': proveedores})


# Borrar producto
    def borrar_producto(request, id):
    producto = get_object_or_404(Producto, id=id)
    if request.method == 'POST':
        producto.delete()
        return redirect('ver_producto')
    return render(request, 'producto/borrar_tabla.html', {'producto': producto})


##PARTE 7: URLs de la app (app_farmacia/urls.py)

    from django.urls import path
    from . import views

    urlpatterns = [
    path('agregar/', views.agregar_producto, name='agregar_producto'),
    path('ver/', views.ver_producto, name='ver_producto'),
    path('actualizar/<int:id>/', views.actualizar_producto, name='actualizar_producto'),
    path('borrar/<int:id>/', views.borrar_producto, name='borrar_producto'),
    ]

Y en backend_Farmacia/urls.py:

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
    path('admin/', admin.site.urls),
    path('producto/', include('app_farmacia.urls')),
    ]

##PARTE 8: Templates HTML (app_farmacia/templates/producto/)

📘  agregar_producto.html

    {% include 'navbar.html' %}
    <h2>Agregar Producto</h2>
    <form method="POST">
      {% csrf_token %}
      <label>Nombre:</label><input type="text" name="nombre"><br>
     <label>Tipo:</label><input type="text" name="tipo_producto"><br>
    <label>Fecha Caducidad:</label><input type="date" name="fecha_caducidad"><br>
    <label>Precio:</label><input type="number" step="0.01" name="precio"><br>
    <label>Proveedor:</label>
    <select name="proveedor">
    {% for p in proveedores %}
    <option value="{{ p.id }}">{{ p.nombre }}</option>
    {% endfor %}
    </select><br>
    <button type="submit">Guardar</button>
    </form>

📗 ver_producto.html


    {% include 'navbar.html' %}
    <h2>Lista de Productos</h2>
    <table border="1">
    <tr>
    <th>Nombre</th>
    <th>Tipo</th>
    <th>Caducidad</th>
    <th>Precio</th>
    <th>Proveedor</th>
    <th>Acciones</th>
    </tr>
    {% for p in productos %}
    <tr>
    <td>{{ p.nombre }}</td>
    <td>{{ p.tipo_producto }}</td>
    <td>{{ p.fecha_caducidad }}</td>
    <td>{{ p.precio }}</td>
    <td>{{ p.proveedor.nombre }}</td>
    <td>
      <a href="{% url 'actualizar_producto' p.id %}">Editar</a> |
      <a href="{% url 'borrar_producto' p.id %}">Borrar</a>
    </td>
    </tr>
    {% endfor %}
    </table>

📙 actualizar_producto.html

    {% include 'navbar.html' %}
    <h2>Actualizar Producto</h2>
    <form method="POST">
      {% csrf_token %}
    <label>Nombre:</label><input type="text" name="nombre" value="{{ producto.nombre }}"><br>
    <label>Tipo:</label><input type="text" name="tipo_producto" value="{{ producto.tipo_producto }}"><br>
    <label>Fecha Caducidad:</label><input type="date" name="fecha_caducidad" value="{{ producto.fecha_caducidad }}"><br>
    <label>Precio:</label><input type="number" step="0.01" name="precio" value="{{ producto.precio }}"><br>
    <label>Proveedor:</label>
    <select name="proveedor">
    {% for p in proveedores %}
    <option value="{{ p.id }}" {% if p.id == producto.proveedor.id %}selected{% endif %}>{{ p.nombre }}</option>
    {% endfor %}
    </select><br>
    <button type="submit">Actualizar</button>
    </form>

📕 borrar_tabla.html

    {% include 'navbar.html' %}
    <h2>¿Deseas eliminar este producto?</h2>
    <p>{{ producto.nombre }} - {{ producto.tipo_producto }}</p>
    <form method="POST">
     {% csrf_token %}
    <button type="submit">Sí, borrar</button>
      <a href="{% url 'ver_producto' %}">Cancelar</a>
    </form>

📒 navbar.html


    <nav style="background-color:#cce5ff; padding:10px;">
    <a href="{% url 'agregar_producto' %}">Agregar Producto</a> |
    <a href="{% url 'ver_producto' %}">Ver Productos</a>
    </nav>
    <hr>


##PARTE 9: Ejecutar servidor en puerto 0080
python manage.py runserver 0.0.0.0:0080


##PARTE 10: Subir el proyecto a GitHub

1️⃣ Crea un repositorio vacío en GitHub (por ejemplo: Farmacia-Django)
2️⃣ En VS Code terminal:

    git init
    git add .
    git commit -m "Proyecto Farmacia CRUD Producto"
    git branch -M main
    git remote add origin https://github.com/TU_USUARIO/Farmacia-Django.git
    git push -u origin main
    
    
ESTRUCTURA COMPLETA DEL PROYECTO
UIII_Farmacia_0777/
│
├── backend_Farmacia/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── app_farmacia/
│   ├── migrations/
│   │   └── __init__.py
│   ├── templates/
│   │   ├── producto/
│   │   │   ├── agregar_producto.html
│   │   │   ├── ver_producto.html
│   │   │   ├── actualizar_producto.html
│   │   │   └── borrar_tabla.html
│   │   └── navbar.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── urls.py
│   └── views.py
│
└── manage.py



##CONTENIDO DE ARCHIVOS
manage.py
#!/usr/bin/env python

    """Django's command-line utility for administrative tasks."""
    import os
    import sys

    def main():
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'backend_Farmacia.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django."
        ) from exc
    execute_from_command_line(sys.argv)

    if __name__ == '__main__':
    main()

    

##backend_Farmacia/settings.py

    from pathlib import Path

    BASE_DIR = Path(__file__).resolve().parent.parent

    SECRET_KEY = 'django-insecure-yourkeyhere'
    DEBUG = True
    ALLOWED_HOSTS = ['*']

    INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_farmacia',
    ]

    MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ]
    
    ROOT_URLCONF = 'backend_Farmacia.urls'

    TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'app_farmacia' / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
    ]

    WSGI_APPLICATION = 'backend_Farmacia.wsgi.application'

    DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
    }

    AUTH_PASSWORD_VALIDATORS = []

    LANGUAGE_CODE = 'es-mx'
    TIME_ZONE = 'UTC'
    USE_I18N = True
    USE_TZ = True

    STATIC_URL = '/static/'
    DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'


  ##backend_Farmacia/urls.py

  
    from django.contrib import admin
    from django.urls import path, include
    urlpatterns = [
    path('admin/', admin.site.urls),
    path('producto/', include('app_farmacia.urls')),
    ]

    app_farmacia/models.py
    from django.db import models

    class Proveedor(models.Model):
    nombre = models.CharField(max_length=100)
    telefono = models.CharField(max_length=20)

    def __str__(self):
        return self.nombre

    class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    tipo_producto = models.CharField(max_length=50)
    fecha_caducidad = models.DateField()
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    proveedor = models.ForeignKey(Proveedor, on_delete=models.CASCADE, related_name='productos')

    def __str__(self):
        return f"{self.nombre} - {self.tipo_producto}"

    app_farmacia/admin.py
    from django.contrib import admin
    from .models import Producto, Proveedor

    admin.site.register(Proveedor)
    admin.site.register(Producto)

##app_farmacia/views.py

    from django.shortcuts import render, redirect, get_object_or_404
    from .models import Producto, Proveedor

    def agregar_producto(request):
    if request.method == 'POST':
        nombre = request.POST['nombre']
        tipo_producto = request.POST['tipo_producto']
        fecha_caducidad = request.POST['fecha_caducidad']
        precio = request.POST['precio']
        proveedor_id = request.POST['proveedor']
        proveedor = Proveedor.objects.get(id=proveedor_id)
        Producto.objects.create(
            nombre=nombre,
            tipo_producto=tipo_producto,
            fecha_caducidad=fecha_caducidad,
            precio=precio,
            proveedor=proveedor
        )
        return redirect('ver_producto')
    proveedores = Proveedor.objects.all()
    return render(request, 'producto/agregar_producto.html', {'proveedores': proveedores})

    def ver_producto(request):
    productos = Producto.objects.all()
    return render(request, 'producto/ver_producto.html', {'productos': productos})

    def actualizar_producto(request, id):
    producto = get_object_or_404(Producto, id=id)
    proveedores = Proveedor.objects.all()
    if request.method == 'POST':
        producto.nombre = request.POST['nombre']
        producto.tipo_producto = request.POST['tipo_producto']
        producto.fecha_caducidad = request.POST['fecha_caducidad']
        producto.precio = request.POST['precio']
        proveedor_id = request.POST['proveedor']
        producto.proveedor = Proveedor.objects.get(id=proveedor_id)
        producto.save()
        return redirect('ver_producto')
    return render(request, 'producto/actualizar_producto.html', {'producto': producto, 'proveedores': proveedores})

    def borrar_producto(request, id):
    producto = get_object_or_404(Producto, id=id)
    if request.method == 'POST':
        producto.delete()
        return redirect('ver_producto')
    return render(request, 'producto/borrar_tabla.html', {'producto': producto})

##app_farmacia/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
    path('agregar/', views.agregar_producto, name='agregar_producto'),
    path('ver/', views.ver_producto, name='ver_producto'),
    path('actualizar/<int:id>/', views.actualizar_producto, name='actualizar_producto'),
    path('borrar/<int:id>/', views.borrar_producto, name='borrar_producto'),
    ]

##app_farmacia/templates/navbar.html


    <nav style="background-color:#cce5ff; padding:10px;">
    <a href="{% url 'agregar_producto' %}">Agregar Producto</a> |
     <a href="{% url 'ver_producto' %}">Ver Productos</a>
    </nav>
    <hr>

##app_farmacia/templates/producto/agregar_producto.html


    {% include 'navbar.html' %}
    <h2>Agregar Producto</h2>
    <form method="POST">
     {% csrf_token %}
    <label>Nombre:</label><input type="text" name="nombre"><br>
    <label>Tipo:</label><input type="text" name="tipo_producto"><br>
    <label>Fecha Caducidad:</label><input type="date" name="fecha_caducidad"><br>
    <label>Precio:</label><input type="number" step="0.01" name="precio"><br>
    <label>Proveedor:</label>
    <select name="proveedor">
    {% for p in proveedores %}
    <option value="{{ p.id }}">{{ p.nombre }}</option>
    {% endfor %}
    </select><br>
    <button type="submit">Guardar</button>
    </form>

##app_farmacia/templates/producto/ver_producto.html



    {% include 'navbar.html' %}
    <h2>Lista de Productos</h2>
    <table border="1">
     <tr>
    <th>Nombre</th>
    <th>Tipo</th>
    <th>Caducidad</th>
    <th>Precio</th>
    <th>Proveedor</th>
    <th>Acciones</th>
    </tr>
    {% for p in productos %}
    <tr>
    <td>{{ p.nombre }}</td>
    <td>{{ p.tipo_producto }}</td>
    <td>{{ p.fecha_caducidad }}</td>
    <td>{{ p.precio }}</td>
    <td>{{ p.proveedor.nombre }}</td>
    <td>
      <a href="{% url 'actualizar_producto' p.id %}">Editar</a> |
      <a href="{% url 'borrar_producto' p.id %}">Borrar</a>
    </td>
    </tr>
    {% endfor %}
    </table>

##app_farmacia/templates/producto/actualizar_producto.html


    {% include 'navbar.html' %}
    <h2>Actualizar Producto</h2>
    <form method="POST">
    {% csrf_token %}
    <label>Nombre:</label><input type="text" name="nombre" value="{{ producto.nombre }}"><br>
    <label>Tipo:</label><input type="text" name="tipo_producto" value="{{ producto.tipo_producto }}"><br>
    <label>Fecha Caducidad:</label><input type="date" name="fecha_caducidad" value="{{ producto.fecha_caducidad }}"><br>
    <label>Precio:</label><input type="number" step="0.01" name="precio" value="{{ producto.precio }}"><br>
    <label>Proveedor:</label>
    <select name="proveedor">
    {% for p in proveedores %}
    <option value="{{ p.id }}" {% if p.id == producto.proveedor.id %}selected{% endif %}>{{ p.nombre }}</option>
    {% endfor %}
    </select><br>
    <button type="submit">Actualizar</button>
    </form>

##app_farmacia/templates/producto/borrar_tabla.html



    {% include 'navbar.html' %}
      <h2>¿Deseas eliminar este producto?</h2>
    <p>{{ producto.nombre }} - {{ producto.tipo_producto }}</p>
    <form method="POST">
    {% csrf_token %}
    <button type="submit">Sí, borrar</button>
    <a href="{% url 'ver_producto' %}">Cancelar</a>
    </form>


✅ Con esto ya tienes el proyecto completo, funcional y listo para ejecutar.

Para ejecutarlo:

python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser  # si quieres acceder a admin
python manage.py runserver 0.0.0.0:0080








    
