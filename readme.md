# UIII-Act-7-models.py-y-https-www.dbdesigner.net-

## DBdesigner:
<img width="1344" height="679" alt="image" src="https://github.com/user-attachments/assets/df1a22c3-c184-4803-a09e-2fcad6e49171" />


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

-- 1. Tabla Propietario
CREATE TABLE Propietario (
    id INT PRIMARY KEY NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    direccion VARCHAR(255) NOT NULL,
    telefono VARCHAR(20) NOT NULL,
    email VARCHAR(100) NOT NULL,
    fecha_registro DATE NOT NULL,
    dni VARCHAR(20) UNIQUE NOT NULL, -- DNI como único
    ocupacion VARCHAR(100) NOT NULL
);

---

-- 2. Tabla Veterinario
CREATE TABLE Veterinario (
    id INT PRIMARY KEY NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    especialidad VARCHAR(100) NOT NULL,
    telefono VARCHAR(20) NOT NULL,
    email VARCHAR(100) NOT NULL,
    licencia_veterinaria VARCHAR(50) UNIQUE NOT NULL, -- Licencia como única
    fecha_contratacion DATE NOT NULL,
    salario DECIMAL(10,2) NOT NULL
);

---

-- 3. Tabla Vacuna
CREATE TABLE Vacuna (
    id INT PRIMARY KEY NOT NULL,
    nombre_vacuna VARCHAR(100) NOT NULL,
    descripcion TEXT NOT NULL,
    laboratorio VARCHAR(100) NOT NULL,
    fecha_vencimiento_lote DATE NOT NULL,
    tipo_enfermedad VARCHAR(100) NOT NULL
);

---

-- 4. Tabla Mascota (Depende de Propietario)
CREATE TABLE Mascota (
    id INT PRIMARY KEY NOT NULL,
    propietario_id INT NOT NULL,
    nombre_mascota VARCHAR(100) NOT NULL,
    especie VARCHAR(50) NOT NULL,
    raza VARCHAR(50) NOT NULL,
    fecha_nacimiento DATE NOT NULL,
    genero CHAR(1) NOT NULL,
    peso_kg DECIMAL(5,2) NOT NULL,
    chip_identificacion VARCHAR(50) NULL, -- Campo opcional
    color VARCHAR(50) NOT NULL,
    esterilizado BOOLEAN NOT NULL,
    FOREIGN KEY (propietario_id) REFERENCES Propietario(id) ON DELETE CASCADE
);

---

-- 5. Tabla Consulta (Depende de Mascota y Veterinario)
CREATE TABLE Consulta (
    id INT PRIMARY KEY NOT NULL,
    mascota_id INT NOT NULL,
    veterinario_id INT NULL, -- Es NULLable debido a ON DELETE SET NULL en Django
    fecha_consulta DATETIME NOT NULL,
    motivo_consulta TEXT NOT NULL,
    diagnostico TEXT NOT NULL,
    tratamiento TEXT NOT NULL,
    peso_mascota_consulta DECIMAL(5,2) NOT NULL,
    observaciones TEXT NULL, -- Campo opcional
    FOREIGN KEY (mascota_id) REFERENCES Mascota(id) ON DELETE CASCADE,
    FOREIGN KEY (veterinario_id) REFERENCES Veterinario(id) ON DELETE SET NULL
);

---

-- 6. Tabla HistorialVacunacion (Depende de Mascota, Vacuna y Veterinario)
CREATE TABLE HistorialVacunacion (
    id INT PRIMARY KEY NOT NULL,
    mascota_id INT NOT NULL,
    vacuna_id INT NOT NULL,
    veterinario_aplico_id INT NULL, -- Es NULLable debido a ON DELETE SET NULL en Django
    fecha_aplicacion DATE NOT NULL,
    fecha_proxima_dosis DATE NULL, -- Campo opcional
    numero_lote VARCHAR(50) NOT NULL,
    comentarios TEXT NULL, -- Campo opcional
    FOREIGN KEY (mascota_id) REFERENCES Mascota(id) ON DELETE CASCADE,
    FOREIGN KEY (vacuna_id) REFERENCES Vacuna(id) ON DELETE CASCADE,
    FOREIGN KEY (veterinario_aplico_id) REFERENCES Veterinario(id) ON DELETE SET NULL
);

---

-- 7. Tabla FacturaVeterinaria (Depende de Propietario y Consulta)
CREATE TABLE FacturaVeterinaria (
    id INT PRIMARY KEY NOT NULL,
    propietario_id INT NOT NULL,
    consulta_asociada_id INT NOT NULL,
    fecha_emision DATE NOT NULL,
    total_factura DECIMAL(10,2) NOT NULL,
    estado_pago VARCHAR(50) NOT NULL,
    metodo_pago VARCHAR(50) NOT NULL,
    fecha_vencimiento DATE NOT NULL,
    FOREIGN KEY (propietario_id) REFERENCES Propietario(id) ON DELETE CASCADE,
    FOREIGN KEY (consulta_asociada_id) REFERENCES Consulta(id) ON DELETE CASCADE
);
