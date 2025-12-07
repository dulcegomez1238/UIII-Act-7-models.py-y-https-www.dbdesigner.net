# UIII-Act-7-models.py-y-https-www.dbdesigner.net-

## DBdesigner:

models.py 



  Modelo para SISTEMA DE GESTION DE CLÍNICAS VETERINARIAS
from django.db import models

class Propietario(models.Model):
    # Django crea automáticamente un id (id_propietario) como Primary Key
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    direccion = models.CharField(max_length=255)
    telefono = models.CharField(max_length=20)
    email = models.EmailField(max_length=100) # Usamos EmailField para validación
    fecha_registro = models.DateField(auto_now_add=True) # auto_now_add registra la fecha de creación
    dni = models.CharField(max_length=20, unique=True) # unique para evitar DNI duplicados
    ocupacion = models.CharField(max_length=100)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"

class Veterinario(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    especialidad = models.CharField(max_length=100)
    telefono = models.CharField(max_length=20)
    email = models.EmailField(max_length=100)
    licencia_veterinaria = models.CharField(max_length=50, unique=True)
    fecha_contratacion = models.DateField()
    salario = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return f"Dr. {self.nombre} {self.apellido} - {self.especialidad}"

class Vacuna(models.Model):
    nombre_vacuna = models.CharField(max_length=100)
    descripcion = models.TextField()
    laboratorio = models.CharField(max_length=100)
    fecha_vencimiento_lote = models.DateField()
    tipo_enfermedad = models.CharField(max_length=100)

    def __str__(self):
        return self.nombre_vacuna

class Mascota(models.Model):
    propietario = models.ForeignKey(Propietario, on_delete=models.CASCADE, related_name='mascotas')
    nombre_mascota = models.CharField(max_length=100)
    especie = models.CharField(max_length=50)
    raza = models.CharField(max_length=50)
    fecha_nacimiento = models.DateField()
    genero = models.CharField(max_length=1) # CHAR(1)
    peso_kg = models.DecimalField(max_digits=5, decimal_places=2)
    chip_identificacion = models.CharField(max_length=50, blank=True, null=True) # Opcional
    color = models.CharField(max_length=50)
    esterilizado = models.BooleanField(default=False)

    def __str__(self):
        return f"{self.nombre_mascota} ({self.especie})"

class Consulta(models.Model):
    mascota = models.ForeignKey(Mascota, on_delete=models.CASCADE, related_name='consultas')
    veterinario = models.ForeignKey(Veterinario, on_delete=models.SET_NULL, null=True)
    fecha_consulta = models.DateTimeField(auto_now_add=True)
    motivo_consulta = models.TextField()
    diagnostico = models.TextField()
    tratamiento = models.TextField()
    peso_mascota_consulta = models.DecimalField(max_digits=5, decimal_places=2)
    observaciones = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"Consulta {self.mascota.nombre_mascota} - {self.fecha_consulta.date()}"

class HistorialVacunacion(models.Model):
    mascota = models.ForeignKey(Mascota, on_delete=models.CASCADE, related_name='vacunas')
    vacuna = models.ForeignKey(Vacuna, on_delete=models.CASCADE)
    veterinario_aplico = models.ForeignKey(Veterinario, on_delete=models.SET_NULL, null=True)
    fecha_aplicacion = models.DateField(auto_now_add=True)
    fecha_proxima_dosis = models.DateField(blank=True, null=True)
    numero_lote = models.CharField(max_length=50)
    comentarios = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"{self.vacuna.nombre_vacuna} aplicada a {self.mascota.nombre_mascota}"

class FacturaVeterinaria(models.Model):
    propietario = models.ForeignKey(Propietario, on_delete=models.CASCADE)
    consulta_asociada = models.ForeignKey(Consulta, on_delete=models.CASCADE)
    fecha_emision = models.DateField(auto_now_add=True)
    total_factura = models.DecimalField(max_digits=10, decimal_places=2)
    estado_pago = models.CharField(max_length=50, choices=[('PAGADO', 'Pagado'), ('PENDIENTE', 'Pendiente')])
    metodo_pago = models.CharField(max_length=50)
    fecha_vencimiento = models.DateField()

    def __str__(self):
        return f"Factura #{self.id} - {self.propietario.nombre}"


## SQL


Propietario {
	id int pk null
	nombre varchar(100) null
	apellido varchar(100) null
	direccion varchar(255) null
	telefono varchar(20) null
	email varchar(100) null
	fecha_registro date null
	dni varchar(20) null
	ocupacion varchar(100) null
}

Mascota {
	id int pk null
	nombre_mascota varchar(100) null
	especie varchar(50) null
	raza varchar(50) null
	fecha_nacimiento date null
	genero char(1) null
	peso_kg decimal(5,2) null
	propietario_id int null > Propietario.id
	chip_identificacion varchar(50) null
	color varchar(50) null
	esterilizado boolean null
}

Veterinario {
	id int pk null
	nombre varchar(100) null
	apellido varchar(100) null
	especialidad varchar(100) null
	telefono varchar(20) null
	email varchar(100) null
	licencia_veterinaria varchar(50) null
	fecha_contratacion date null
	salario decimal(10,2) null
}

Consulta {
	id int pk null
	mascota_id int null > Mascota.id
	veterinario_id int null > Veterinario.id
	fecha_consulta datetime null
	motivo_consulta text null
	diagnostico text null
	tratamiento text null
	peso_mascota_consulta decimal(5,2) null
	observaciones text null
}

Vacuna {
	id int pk null
	nombre_vacuna varchar(100) null
	descripcion text null
	laboratorio varchar(100) null
	fecha_vencimiento_lote date null
	tipo_enfermedad varchar(100) null
}

HistorialVacunacion {
	id int pk null
	mascota_id int null > Mascota.id
	vacuna_id int null > Vacuna.id
	fecha_aplicacion date null
	fecha_proxima_dosis date null
	veterinario_aplico_id int null > Veterinario.id
	numero_lote varchar(50) null
	comentarios text null
}

FacturaVeterinaria {
	fecha_emision date null
	total_factura decimal(10,2) null
	estado_pago varchar(50) null
	id int pk null
	propietario_id int null > Propietario.id
	metodo_pago varchar(50) null
	consulta_asociada_id int null > Consulta.id
	fecha_vencimiento date null
}

