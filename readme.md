# 💳Mini Proyecto SQL: Sistema de Pagos

Este proyecto simula un sistema básico de pagos para practicar SQL a nivel profesional. Está diseñado para demostrar habilidades en modelado de datos, escritura de procedimientos almacenados, funciones, triggers y buenas prácticas en SQL Server.

---

## 📌Objetivo

Construir un sistema que gestione pagos de clientes, incluyendo los métodos de pago utilizados y la generación automática de facturas.

---

## 🧱Estructura de la base de datos

La base de datos se llama: **SistemaPagos**

### Tablas principales:

| Tabla          | Descripción                            |
|----------------|----------------------------------------|
| `clientes`     | Información de los clientes            |
| `metodos_pago` | Métodos de pago disponibles            |
| `pagos`        | Registro de pagos realizados           |
| `facturas`     | Facturas generadas por cada pago       |

---

### 📊Diagrama relacional básico

- Un cliente puede tener muchos pagos.
- Cada pago utiliza un método de pago.
- Cada pago **confirmado** genera una factura automáticamente.

---

## 🔧 Scripts implementados

### 1. Creación de tablas

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY IDENTITY,
    nombre NVARCHAR(100),
    correo NVARCHAR(100)
);

CREATE TABLE metodos_pago (
    id INT PRIMARY KEY IDENTITY,
    nombre_metodo NVARCHAR(50),
    tipo NVARCHAR(50)
);

CREATE TABLE pagos (
    id INT PRIMARY KEY IDENTITY,
    cliente_id INT,
    metodo_pago_id INT,
    monto DECIMAL(10,2),
    fecha_pago DATETIME,
    estatus NVARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES clientes(id),
    FOREIGN KEY (metodo_pago_id) REFERENCES metodos_pago(id)
);

CREATE TABLE facturas (
    id INT PRIMARY KEY IDENTITY,
    pago_id INT,
    descripcion NVARCHAR(255),
    total DECIMAL(10,2),
    fecha_emision DATETIME,
    FOREIGN KEY (pago_id) REFERENCES pagos(id)
);

### 2. Procedimiento almacenado


CREATE PROCEDURE sp_realizar_pago
    @cliente_id INT,
    @metodo_pago_id INT,
    @monto DECIMAL(10,2),
    @fecha_pago DATETIME,
    @estatus NVARCHAR(50)
AS
BEGIN
    INSERT INTO pagos (cliente_id, metodo_pago_id, monto, fecha_pago, estatus)
    VALUES (@cliente_id, @metodo_pago_id, @monto, @fecha_pago, @estatus);
END

### 3. Función

CREATE FUNCTION fn_historial_pagos (@cliente_id INT)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        p.id AS pago_id,
        p.monto,
        p.fecha_pago,
        p.estatus,
        mp.nombre_metodo,
        f.descripcion AS descripcion_factura,
        f.total AS total_factura,
        f.fecha_emision
    FROM pagos p
    INNER JOIN metodos_pago mp ON p.metodo_pago_id = mp.id
    LEFT JOIN facturas f ON f.pago_id = p.id
    WHERE p.cliente_id = @cliente_id
);

### 4. Trigger

CREATE TRIGGER trg_generar_factura
ON pagos
AFTER INSERT
AS
BEGIN
    INSERT INTO facturas (pago_id, descripcion, total, fecha_emision)
    SELECT 
        i.id,
        'Factura generada automáticamente',
        i.monto,
        i.fecha_pago
    FROM inserted i
    WHERE i.estatus = 'Confirmado';
END

