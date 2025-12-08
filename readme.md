# UIII-Act-7-models.py-y-https-www.dbdesigner.net-

## DBdesigner:
<img width="1344" height="679" alt="image" src="https://github.com/user-attachments/assets/df1a22c3-c184-4803-a09e-2fcad6e49171" />


from django.db import models


# =======================
# PROPIETARIO
# =======================
    class Propietario(models.Model):
    id_propietario = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    direccion = models.CharField(max_length=255, null=True, blank=True)
    telefono = models.CharField(max_length=20, null=True, blank=True)
    email = models.CharField(max_length=100, null=True, blank=True)
    fecha_registro = models.DateField(null=True, blank=True)
    dni = models.CharField(max_length=20, null=True, blank=True)
    ocupacion = models.CharField(max_length=100, null=True, blank=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"


# =======================
# MASCOTA
# =======================
class Mascota(models.Model):
    id_mascota = models.AutoField(primary_key=True)
    nombre_mascota = models.CharField(max_length=100)
    especie = models.CharField(max_length=50, null=True, blank=True)
    raza = models.CharField(max_length=50, null=True, blank=True)
    fecha_nacimiento = models.DateField(null=True, blank=True)
    genero = models.CharField(max_length=1, null=True, blank=True)
    peso_kg = models.DecimalField(max_digits=5, decimal_places=2, null=True, blank=True)
    chip_identificacion = models.CharField(max_length=50, null=True, blank=True)
    color = models.CharField(max_length=50, null=True, blank=True)
    esterilizado = models.BooleanField(default=False)

    # FK → Propietario
    id_propietario = models.ForeignKey(
        Propietario,
        on_delete=models.SET_NULL,
        null=True,
        related_name='mascotas'
    )

    def __str__(self):
        return self.nombre_mascota


# =======================
# VETERINARIO
# =======================
class Veterinario(models.Model):
    id_veterinario = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    especialidad = models.CharField(max_length=100, null=True, blank=True)
    telefono = models.CharField(max_length=20, null=True, blank=True)
    email = models.CharField(max_length=100, null=True, blank=True)
    licencia_veterinaria = models.CharField(max_length=50, null=True, blank=True)
    fecha_contratacion = models.DateField(null=True, blank=True)
    salario = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"


# =======================
# CONSULTA
# =======================
class Consulta(models.Model):
    id_consulta = models.AutoField(primary_key=True)

    # FK → Mascota
    id_mascota = models.ForeignKey(
        Mascota,
        on_delete=models.CASCADE,
        related_name='consultas'
    )

    # FK → Veterinario
    id_veterinario = models.ForeignKey(
        Veterinario,
        on_delete=models.SET_NULL,
        null=True,
        related_name='consultas'
    )

    fecha_consulta = models.DateTimeField()
    motivo_consulta = models.TextField(null=True, blank=True)
    diagnostico = models.TextField(null=True, blank=True)
    tratamiento = models.TextField(null=True, blank=True)
    peso_mascota_consulta = models.DecimalField(max_digits=5, decimal_places=2, null=True, blank=True)
    observaciones = models.TextField(null=True, blank=True)

    def __str__(self):
        return f"Consulta #{self.id_consulta} - {self.id_mascota}"


# =======================
# VACUNA
# =======================
class Vacuna(models.Model):
    id_vacuna = models.AutoField(primary_key=True)
    nombre_vacuna = models.CharField(max_length=100)
    descripcion = models.TextField(null=True, blank=True)
    laboratorio = models.CharField(max_length=100, null=True, blank=True)
    fecha_vencimiento_lote = models.DateField(null=True, blank=True)
    tipo_enfermedad = models.CharField(max_length=100, null=True, blank=True)

    def __str__(self):
        return self.nombre_vacuna


# =======================
# HISTORIAL VACUNACIÓN
# =======================
class Historial_Vacunacion(models.Model):
    id_historial_vac = models.AutoField(primary_key=True)

    # FK → Mascota
    id_mascota = models.ForeignKey(
        Mascota,
        on_delete=models.CASCADE,
        related_name='historial_vacunas'
    )

    # FK → Vacuna
    id_vacuna = models.ForeignKey(
        Vacuna,
        on_delete=models.CASCADE,
        related_name='historial'
    )

    # FK → Veterinario
    id_veterinario_aplico = models.ForeignKey(
        Veterinario,
        on_delete=models.SET_NULL,
        null=True,
        related_name='vacunas_aplicadas'
    )

    fecha_aplicacion = models.DateField()
    fecha_proxima_dosis = models.DateField(null=True, blank=True)
    numero_lote = models.CharField(max_length=50, null=True, blank=True)
    comentarios = models.TextField(null=True, blank=True)

    def __str__(self):
        return f"Vacuna {self.id_vacuna} aplicada a {self.id_mascota}"


# =======================
# FACTURA VETERINARIA
# =======================
class Factura_Veterinaria(models.Model):
    id_factura = models.AutoField(primary_key=True)

    # FK → Propietario
    id_propietario = models.ForeignKey(
        Propietario,
        on_delete=models.SET_NULL,
        null=True,
        related_name='facturas'
    )

    # FK → Consulta
    id_consulta_asociada = models.ForeignKey(
        Consulta,
        on_delete=models.SET_NULL,
        null=True,
        related_name='facturas'
    )

    fecha_emision = models.DateField()
    total_factura = models.DecimalField(max_digits=10, decimal_places=2)
    estado_pago = models.CharField(max_length=50)
    metodo_pago = models.CharField(max_length=50, null=True, blank=True)
    fecha_vencimiento = models.DateField(null=True, blank=True)

    def __str__(self):
        return f"Factura #{self.id_factura}"
