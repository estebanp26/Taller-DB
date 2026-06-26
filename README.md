# Taller-DB


# Trabajo Práctico: Análisis de Requerimientos, Modelo Relacional y Normalización
## Riwi Supply

## Contexto del Caso

La empresa Riwi Supply ha venido registrando la información de sus operaciones comerciales en una única hoja de cálculo. Actualmente se almacenan datos relacionados con clientes, sucursales, asesores comerciales, productos, categorías, facturas y pagos en una sola estructura de datos.

La gerencia ha decidido implementar una base de datos relacional que permita mejorar la integridad de la información, reducir la duplicidad de datos y facilitar la generación de reportes comerciales.

---

## Fase 1. Comprensión del Negocio

**¿Qué proceso de negocio representa la información?**

El dataset representa el proceso de ventas comerciales de Riwi Supply, una empresa distribuidora de materiales de construcción, herramientas, pinturas y agregados. El proceso cubre desde la emisión de una factura hasta el registro del pago, pasando por la identificación del cliente, el asesor, la sucursal y los productos vendidos.

**¿Qué eventos se están registrando?**

Se registran transacciones de venta: cada fila representa un ítem (producto) dentro de una factura. Cada factura puede contener múltiples productos. También se registra la fecha de la venta, el método de pago utilizado y el valor unitario de cada producto.

**¿Qué actores participan en el proceso?**

Los actores son: el Cliente (quien compra), el Asesor Comercial (quien gestiona la venta), la Sucursal (donde se realiza la operación) y la empresa Riwi Supply (quien vende y emite la factura).

**¿Qué información parece repetirse?**

Se observa alta redundancia: el nombre del cliente y su ciudad se repiten en cada fila que pertenece a la misma factura. Los datos del asesor (nombre) se repiten en cada venta que gestiona. Los datos de la sucursal (nombre y ciudad) se repiten en cada transacción. El nombre del producto, su categoría y precio unitario se repiten cada vez que ese producto aparece en una factura. El número de factura y método de pago se repiten para cada ítem de la misma factura.

**¿Qué problemas podrían presentarse si la empresa continúa almacenando datos de esta manera?**

- Anomalías de inserción: no se puede registrar un cliente sin que exista una factura.
- Anomalías de actualización: si el precio de un producto cambia, hay que actualizar múltiples filas con riesgo de inconsistencia.
- Anomalías de eliminación: eliminar una factura puede borrar información del cliente o del producto.
- Redundancia masiva que consume espacio y dificulta el mantenimiento.
- Imposibilidad de garantizar integridad referencial.

---

## Fase 2. Identificación de Entidades

A partir del análisis del dataset se identificaron las siguientes entidades:

1. **Cliente** – Persona o empresa que adquiere los productos (¿quién compra?). Campos: CustomerName, CustomerCity.
2. **Asesor Comercial** – Empleado responsable de gestionar la venta (¿quién vende?). Campo: SalesRep.
3. **Sucursal** – Punto físico donde se realiza la operación (¿dónde se vende?). Campos: Branch, BranchCity.
4. **Categoría** – Clasificación del producto. Campo: Category.
5. **Producto** – Bien comercializado por la empresa (¿qué se vende?). Campos: ProductName, UnitPrice, Category.
6. **Factura** – Documento que registra una transacción de venta (¿cómo se registra?). Campos: InvoiceNumber, OrderDate, PaymentMethod.
7. **Detalle de Factura** – Línea específica de un producto dentro de una factura. Campos: Quantity, UnitPrice.

---

## Fase 3. Identificación de Atributos – Diccionario de Datos

| Entidad | Atributo | Tipo de dato | PK | FK | Descripción |
|---|---|---|---|---|---|
| Cliente | id_cliente | INT | Sí | No | Identificador único del cliente |
| Cliente | nombre_cliente | VARCHAR(150) | No | No | Nombre o razón social |
| Cliente | ciudad_cliente | VARCHAR(100) | No | No | Ciudad donde está el cliente |
| Asesor Comercial | id_asesor | INT | Sí | No | Identificador único del asesor |
| Asesor Comercial | nombre_asesor | VARCHAR(100) | No | No | Nombre completo del asesor |
| Sucursal | id_sucursal | INT | Sí | No | Identificador único de la sucursal |
| Sucursal | nombre_sucursal | VARCHAR(100) | No | No | Nombre de la sucursal |
| Sucursal | ciudad_sucursal | VARCHAR(100) | No | No | Ciudad donde se ubica la sucursal |
| Categoría | id_categoria | INT | Sí | No | Identificador único de la categoría |
| Categoría | nombre_categoria | VARCHAR(80) | No | No | Nombre de la categoría |
| Producto | id_producto | INT | Sí | No | Identificador único del producto |
| Producto | nombre_producto | VARCHAR(150) | No | No | Nombre del producto |
| Producto | precio_unitario | DECIMAL(12,2) | No | No | Precio unitario del producto |
| Producto | id_categoria | INT | No | Sí | FK → Categoría |
| Factura | id_factura | VARCHAR(20) | Sí | No | Número de factura (ej. FAC-2001) |
| Factura | fecha | DATE | No | No | Fecha de emisión |
| Factura | metodo_pago | VARCHAR(50) | No | No | Transferencia / Tarjeta / Efectivo / Crédito |
| Factura | id_cliente | INT | No | Sí | FK → Cliente |
| Factura | id_asesor | INT | No | Sí | FK → Asesor Comercial |
| Factura | id_sucursal | INT | No | Sí | FK → Sucursal |
| Detalle Factura | id_detalle | INT | Sí | No | Identificador único del detalle |
| Detalle Factura | id_factura | VARCHAR(20) | No | Sí | FK → Factura |
| Detalle Factura | id_producto | INT | No | Sí | FK → Producto |
| Detalle Factura | cantidad | INT | No | No | Unidades vendidas |
| Detalle Factura | precio_unitario_venta | DECIMAL(12,2) | No | No | Precio al momento de la venta |

---

## Fase 4. Identificación de Relaciones

**Relaciones 1 a muchos (1:N):**

- Categoría 1:N Producto (una categoría agrupa muchos productos).
- Cliente 1:N Factura (un cliente puede tener muchas facturas).
- Asesor 1:N Factura (un asesor puede gestionar muchas facturas).
- Sucursal 1:N Factura (una sucursal emite muchas facturas).
- Factura 1:N Detalle_Factura (una factura contiene muchos ítems).

**Relaciones muchos a muchos (N:M) resueltas:**

- Factura N:M Producto, resuelta mediante la entidad intermedia Detalle_Factura. Una factura puede tener muchos productos y un producto puede aparecer en muchas facturas.

**Dependencias funcionales principales:**

- InvoiceNumber → OrderDate, PaymentMethod, CustomerName, SalesRep, Branch
- ProductName → UnitPrice, Category
- SalesRep → Branch (el asesor pertenece a una sucursal)
- Branch → BranchCity
- CustomerName → CustomerCity

**Modelo Entidad-Relación preliminar (descripción textual):**

```
CLIENTE (id_cliente PK, nombre_cliente, ciudad_cliente)
ASESOR (id_asesor PK, nombre_asesor)
SUCURSAL (id_sucursal PK, nombre_sucursal, ciudad_sucursal)
CATEGORIA (id_categoria PK, nombre_categoria)
PRODUCTO (id_producto PK, nombre_producto, precio_unitario, id_categoria FK)
FACTURA (id_factura PK, fecha, metodo_pago, id_cliente FK, id_asesor FK, id_sucursal FK)
DETALLE_FACTURA (id_detalle PK, id_factura FK, id_producto FK, cantidad, precio_unitario_venta)
```

---

## Fase 5. Revisión y Refinamiento del Modelo

**¿Existen atributos duplicados?**
No. Cada atributo reside en una única tabla tras la descomposición.

**¿Existen dependencias parciales?**
No. Todas las tablas tienen PK simple y cada atributo depende completamente de esa PK.

**¿Existen dependencias transitivas?**
No. Se separaron las dependencias transitivas (ej. Branch → BranchCity se movió a la tabla Sucursal; Category se separó en su propia tabla).

**¿Todas las entidades representan conceptos del negocio?**
Sí. Cliente, Asesor, Sucursal, Categoría, Producto, Factura y Detalle_Factura son conceptos reales del proceso comercial.

**¿Las relaciones reflejan correctamente la realidad?**
Sí. La relación N:M entre Factura y Producto está correctamente resuelta con Detalle_Factura. El modelo final es el mismo presentado en la Fase 4 sin cambios adicionales necesarios.

---

## Fase 6. Normalización

**Primera Forma Normal (1FN):**

El dataset original tiene grupos repetitivos: múltiples productos por factura aparecen como filas separadas con todos los datos de cabecera repetidos. Para cumplir 1FN se garantiza atomicidad: cada celda contiene un único valor. Se verifica que no existen columnas multivaluadas. La estructura de tabla plana ya cumple 1FN en cuanto a atomicidad, pero aún contiene dependencias funcionales que se resuelven en las siguientes formas normales.

**Segunda Forma Normal (2FN):**

La clave compuesta del dataset original podría considerarse (InvoiceNumber, ProductName). Atributos como OrderDate, CustomerName, SalesRep, Branch y PaymentMethod dependen solo de InvoiceNumber (dependencia parcial). Atributos como UnitPrice y Category dependen solo de ProductName (dependencia parcial). Para eliminar estas dependencias parciales se crean tablas separadas: Factura con clave InvoiceNumber y Producto con clave id_producto. La tabla intermedia Detalle_Factura mantiene solo Quantity y el precio de venta.

**Tercera Forma Normal (3FN):**

Se identifican y eliminan dependencias transitivas:
- En la tabla Factura, Branch → BranchCity (ciudad depende de la sucursal, no de la factura) – se crea tabla Sucursal.
- CustomerCity depende de CustomerName, no de la factura – se crea tabla Cliente.
- SalesRep es una entidad independiente – se crea tabla Asesor.
- En la tabla Producto, Category es un atributo que puede repetirse y evolucionar – se crea tabla Categoría.

Resultado: el modelo en 3FN es el esquema de 7 tablas descrito en las fases anteriores.

---

## Fase 7. Modelo Relacional Final

```
CLIENTE(id_cliente INT PK, nombre_cliente VARCHAR(150) NOT NULL, ciudad_cliente VARCHAR(100))
ASESOR(id_asesor INT PK, nombre_asesor VARCHAR(100) NOT NULL)
SUCURSAL(id_sucursal INT PK, nombre_sucursal VARCHAR(100) NOT NULL, ciudad_sucursal VARCHAR(100))
CATEGORIA(id_categoria INT PK, nombre_categoria VARCHAR(80) NOT NULL UNIQUE)
PRODUCTO(id_producto INT PK, nombre_producto VARCHAR(150) NOT NULL, precio_unitario DECIMAL(12,2) NOT NULL, id_categoria INT FK → CATEGORIA)
FACTURA(id_factura VARCHAR(20) PK, fecha DATE NOT NULL, metodo_pago VARCHAR(50), id_cliente INT FK → CLIENTE, id_asesor INT FK → ASESOR, id_sucursal INT FK → SUCURSAL)
DETALLE_FACTURA(id_detalle INT PK, id_factura VARCHAR(20) FK → FACTURA, id_producto INT FK → PRODUCTO, cantidad INT NOT NULL, precio_unitario_venta DECIMAL(12,2) NOT NULL)
```

---

## Fase 8. Script SQL – Creación e Inserción

### Script de creación de la base de datos y tablas

```sql
CREATE DATABASE riwi_supply;
USE riwi_supply;

CREATE TABLE cliente (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nombre_cliente VARCHAR(150) NOT NULL,
    ciudad_cliente VARCHAR(100)
);

CREATE TABLE asesor (
    id_asesor INT AUTO_INCREMENT PRIMARY KEY,
    nombre_asesor VARCHAR(100) NOT NULL
);

CREATE TABLE sucursal (
    id_sucursal INT AUTO_INCREMENT PRIMARY KEY,
    nombre_sucursal VARCHAR(100) NOT NULL,
    ciudad_sucursal VARCHAR(100)
);

CREATE TABLE categoria (
    id_categoria INT AUTO_INCREMENT PRIMARY KEY,
    nombre_categoria VARCHAR(80) NOT NULL UNIQUE
);

CREATE TABLE producto (
    id_producto INT AUTO_INCREMENT PRIMARY KEY,
    nombre_producto VARCHAR(150) NOT NULL,
    precio_unitario DECIMAL(12,2) NOT NULL,
    id_categoria INT,
    FOREIGN KEY (id_categoria) REFERENCES categoria(id_categoria)
);

CREATE TABLE factura (
    id_factura VARCHAR(20) PRIMARY KEY,
    fecha DATE NOT NULL,
    metodo_pago VARCHAR(50),
    id_cliente INT,
    id_asesor INT,
    id_sucursal INT,
    FOREIGN KEY (id_cliente) REFERENCES cliente(id_cliente),
    FOREIGN KEY (id_asesor) REFERENCES asesor(id_asesor),
    FOREIGN KEY (id_sucursal) REFERENCES sucursal(id_sucursal)
);

CREATE TABLE detalle_factura (
    id_detalle INT AUTO_INCREMENT PRIMARY KEY,
    id_factura VARCHAR(20),
    id_producto INT,
    cantidad INT NOT NULL,
    precio_unitario_venta DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (id_factura) REFERENCES factura(id_factura),
    FOREIGN KEY (id_producto) REFERENCES producto(id_producto)
);
```

### Script de inserción de datos

```sql
INSERT INTO cliente VALUES
(1,'Constructora Andina SAS','Bogotá'),
(2,'Ferretería El Tornillo','Medellín'),
(3,'Inversiones Caribe SAS','Barranquilla'),
(4,'Metalúrgica del Norte','Cali'),
(5,'Obras y Vías SAS','Bucaramanga'),
(6,'Constructora Pacífico','Cali'),
(7,'Ferretería Central','Cartagena');

INSERT INTO asesor VALUES
(1,'Carlos Pérez'),
(2,'Laura Gómez'),
(3,'Andrés Ruiz'),
(4,'Diana Torres'),
(5,'Miguel Castro');

INSERT INTO sucursal VALUES
(1,'Centro','Bogotá'),
(2,'Norte','Medellín'),
(3,'Costa','Barranquilla'),
(4,'Occidente','Cali'),
(5,'Oriente','Bucaramanga');

INSERT INTO categoria VALUES
(1,'Construcción'),
(2,'Herramientas'),
(3,'Pinturas'),
(4,'Agregados');

INSERT INTO producto VALUES
(1,'Cemento Gris 50kg',32000,1),
(2,'Varilla 3/8',28000,1),
(3,'Taladro Industrial',450000,2),
(4,'Pintura Blanca 5GL',95000,3),
(5,'Rodillo Profesional',18000,3),
(6,'Disco de Corte',12000,2),
(7,'Broca SDS',15000,2),
(8,'Arena Lavada',25000,4),
(9,'Grava Triturada',27000,4);

INSERT INTO factura VALUES
('FAC-2001','2026-05-01','Transferencia',1,1,1),
('FAC-2002','2026-05-02','Tarjeta',2,2,2),
('FAC-2003','2026-05-03','Crédito',3,3,3),
('FAC-2004','2026-05-04','Efectivo',4,4,4),
('FAC-2005','2026-05-05','Transferencia',1,1,1),
('FAC-2006','2026-05-06','Tarjeta',2,2,2),
('FAC-2007','2026-05-07','Crédito',5,5,5),
('FAC-2008','2026-05-08','Transferencia',6,4,4),
('FAC-2009','2026-05-09','Tarjeta',7,3,3);

INSERT INTO detalle_factura (id_factura,id_producto,cantidad,precio_unitario_venta) VALUES
('FAC-2001',1,100,32000),
('FAC-2001',2,50,28000),
('FAC-2002',3,3,450000),
('FAC-2003',4,20,95000),
('FAC-2003',5,10,18000),
('FAC-2004',6,40,12000),
('FAC-2005',1,120,32000),
('FAC-2006',7,25,15000),
('FAC-2007',8,60,25000),
('FAC-2007',9,80,27000),
('FAC-2008',4,15,95000),
('FAC-2009',3,2,450000);
```

---

## Fase 9. Consultas SQL

**1. Listar todos los clientes:**
```sql
SELECT * FROM cliente;
```

**2. Mostrar todos los productos con su categoría:**
```sql
SELECT p.nombre_producto, p.precio_unitario, c.nombre_categoria
FROM producto p
JOIN categoria c ON p.id_categoria = c.id_categoria;
```

**3. Consultar las facturas emitidas por ciudad (ciudad de sucursal):**
```sql
SELECT s.ciudad_sucursal, f.id_factura, f.fecha
FROM factura f
JOIN sucursal s ON f.id_sucursal = s.id_sucursal
ORDER BY s.ciudad_sucursal;
```

**4. Obtener el total vendido por cliente:**
```sql
SELECT cl.nombre_cliente, SUM(df.cantidad * df.precio_unitario_venta) AS total_vendido
FROM cliente cl
JOIN factura f ON cl.id_cliente = f.id_cliente
JOIN detalle_factura df ON f.id_factura = df.id_factura
GROUP BY cl.id_cliente, cl.nombre_cliente
ORDER BY total_vendido DESC;
```

**5. Obtener el total vendido por categoría:**
```sql
SELECT cat.nombre_categoria, SUM(df.cantidad * df.precio_unitario_venta) AS total_categoria
FROM categoria cat
JOIN producto p ON cat.id_categoria = p.id_categoria
JOIN detalle_factura df ON p.id_producto = df.id_producto
GROUP BY cat.id_categoria, cat.nombre_categoria
ORDER BY total_categoria DESC;
```

**6. Mostrar las facturas atendidas por cada asesor comercial:**
```sql
SELECT a.nombre_asesor, f.id_factura, f.fecha
FROM asesor a
JOIN factura f ON a.id_asesor = f.id_asesor
ORDER BY a.nombre_asesor;
```

**7. Consultar los productos más vendidos (por cantidad total):**
```sql
SELECT p.nombre_producto, SUM(df.cantidad) AS total_unidades
FROM producto p
JOIN detalle_factura df ON p.id_producto = df.id_producto
GROUP BY p.id_producto, p.nombre_producto
ORDER BY total_unidades DESC;
```

**8. Mostrar las sucursales y la cantidad de facturas gestionadas:**
```sql
SELECT s.nombre_sucursal, s.ciudad_sucursal, COUNT(f.id_factura) AS total_facturas
FROM sucursal s
LEFT JOIN factura f ON s.id_sucursal = f.id_sucursal
GROUP BY s.id_sucursal, s.nombre_sucursal, s.ciudad_sucursal
ORDER BY total_facturas DESC;
```

**9. Consultar ventas realizadas mediante un método de pago específico (ejemplo: Transferencia):**
```sql
SELECT f.id_factura, f.fecha, cl.nombre_cliente, f.metodo_pago
FROM factura f
JOIN cliente cl ON f.id_cliente = cl.id_cliente
WHERE f.metodo_pago = 'Transferencia';
```

**10. Obtener el valor total de cada factura:**
```sql
SELECT f.id_factura, f.fecha, cl.nombre_cliente, SUM(df.cantidad * df.precio_unitario_venta) AS valor_total
FROM factura f
JOIN cliente cl ON f.id_cliente = cl.id_cliente
JOIN detalle_factura df ON f.id_factura = df.id_factura
GROUP BY f.id_factura, f.fecha, cl.nombre_cliente
ORDER BY f.id_factura;
```

---

## Conclusiones y Lecciones Aprendidas

El proceso de normalización demostró que almacenar toda la información en una sola hoja de cálculo genera redundancia severa y anomalías de modificación. Al descomponer los datos en 7 tablas normalizadas (Cliente, Asesor, Sucursal, Categoría, Producto, Factura y Detalle_Factura) se logró:

- Eliminar toda duplicidad de datos.
- Garantizar integridad referencial mediante claves foráneas.
- Facilitar el mantenimiento y escalabilidad del sistema.
- Posibilitar consultas analíticas complejas mediante JOINs.

La entidad Detalle_Factura fue clave para resolver la relación muchos a muchos entre Factura y Producto. La lección principal es que un buen diseño de base de datos comienza siempre con un análisis profundo del negocio antes de escribir cualquier línea de código.
