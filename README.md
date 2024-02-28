/*drop database ExamenJPALLAN*/
Create database ExamenJPALLAN;
Use ExamenJPALLAN;

CREATE TABLE categorias (
    id_categoria char(36),
    nombre_categoria VARCHAR(50) UNIQUE
);

CREATE TABLE proveedores (
    id_proveedor char(36),
    nombre_proveedor VARCHAR(50) UNIQUE,
    direccion_proveedor VARCHAR(50),
    telefono_proveedor VARCHAR(10)
);

CREATE TABLE productos (
    id_producto char(36),
    nombre_producto VARCHAR(50) NOT NULL,
    descripcion_producto VARCHAR(255),
    precio_unitario DECIMAL(10,2) NOT NULL,
    id_categoria INT NOT NULL,
    id_proveedor INT NOT NULL
);


CREATE TABLE inventarios (
   id_inventario char(36),
   id_producto INT,
   cantidad_disponible INT CHECK(cantidad_disponible >= 0),
   fecha_ingreso DATE
   );

CREATE TABLE movimientos_inventarios (
   id_movimiento_inventario char(36),
   id_inventario INT,
   tipo_movimiento ENUM('Entrada', 'Salida'),
   cantidad INT CHECK(cantidad >= 0), 
   fecha_movimiento DATE
   );
 
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES
(UUID(), 'Electrónicos'),
(UUID(), 'Ropa'),
(UUID(), 'Hogar'),
(UUID(), 'Alimentos'),
(UUID(), 'Juguetes'),
(UUID(), 'Libros'),
(UUID(), 'Herramientas'),
(UUID(), 'Deportes'),
(UUID(), 'Muebles'),
(UUID(), 'Automóviles');


INSERT INTO proveedores (id_proveedor, nombre_proveedor, direccion_proveedor, telefono_proveedor) VALUES
(UUID(), 'McDonald\'s', '123 Main St', '1234567890'),
(UUID(), 'Nike', '456 Oak Ave', '9876543210'),
(UUID(), 'Sony', '789 Maple Blvd', '1112223333'),
(UUID(), 'Coca-Cola', '101 Pine Ln', '4445556666'),
(UUID(), 'Amazon', '202 Cedar Rd', '7778889999'),
(UUID(), 'Adidas', '303 Elm Ct', '1231231234'),
(UUID(), 'Samsung', '404 Birch Dr', '9879879876'),
(UUID(), 'Apple', '505 Spruce Pl', '5556667777'),
(UUID(), 'Pepsi', '606 Pineapple Blvd', '9990001111'),
(UUID(), 'Walmart', '707 Orange Ln', '1230004567');


INSERT INTO productos (id_producto, nombre_producto, descripcion_producto, precio_unitario, id_categoria, id_proveedor) VALUES
(UUID(), 'Hamburguesa', 'Deliciosa hamburguesa con queso', 4.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'McDonald\'s')),
(UUID(), 'Zapatillas', 'Zapatillas deportivas de última generación', 89.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Nike')),
(UUID(), 'Televisor LED', 'Televisor LED de 55 pulgadas', 599.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Sony')),
(UUID(), 'Refresco', 'Refresco de cola en lata', 1.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Coca-Cola')),
(UUID(), 'Kindle', 'E-reader para libros electrónicos', 129.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Amazon')),
(UUID(), 'Zapatillas deportivas', 'Zapatillas deportivas para correr', 79.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Adidas')),
(UUID(), 'Smartphone Galaxy', 'Smartphone Samsung Galaxy', 699.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Samsung')),
(UUID(), 'iPhone 13', 'iPhone 13 con pantalla OLED', 999.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Apple')),
(UUID(), 'Pepsi Max', 'Refresco de cola sin azúcar', 2.49, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Pepsi')),
(UUID(), 'TV 4K', 'Televisor 4K de 65 pulgadas', 899.99, UUID(), (SELECT id_proveedor FROM proveedores WHERE nombre_proveedor = 'Walmart'));


INSERT INTO inventarios (id_inventario, id_producto, cantidad_disponible, fecha_ingreso) VALUES
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Hamburguesa'), 100, '2054-01-01'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Zapatillas'), 150, '2064-01-02'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Televisor LED'), 200, '2634-01-03'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Refresco'), 50, '2085-01-04'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Kindle'), 80, '2754-01-05'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Zapatillas deportivas'), 120, '2624-01-06'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Smartphone Galaxy'), 30, '2624-01-07'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'iPhone 13'), 70, '2624-01-08'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'Pepsi Max'), 90, '2924-01-09'),
(UUID(), (SELECT id_producto FROM productos WHERE nombre_producto = 'TV 4K'), 110, '4624-01-10');

INSERT INTO movimientos_inventarios (id_movimiento_inventario, id_inventario, tipo_movimiento, cantidad, fecha_movimiento) VALUES
(UUID(), UUID(), 'Entrada', 20, '2024-02-01'),
(UUID(), UUID(), 'Salida', 10, '2024-02-02'),
(UUID(), UUID(), 'Entrada', 30, '2024-02-03'),
(UUID(), UUID(), 'Salida', 5, '2024-02-04'),
(UUID(), UUID(), 'Entrada', 15, '2024-02-05'),
(UUID(), UUID(), 'Salida', 25, '2024-02-06'),
(UUID(), UUID(), 'Entrada', 10, '2024-02-07'),
(UUID(), UUID(), 'Salida', 20, '2024-02-08'),
(UUID(), UUID(), 'Entrada', 5, '2024-02-09'),
(UUID(), UUID(), 'Salida', 15, '2024-02-10');

DELIMITER //
CREATE TRIGGER actualizar_inventario
AFTER INSERT ON movimientos_inventarios
FOR EACH ROW
BEGIN
    DECLARE cantidad_movimiento INT;
    SET cantidad_movimiento = NEW.cantidad;
    IF NEW.tipo_movimiento = 'Entrada' THEN
        UPDATE inventarios
        SET cantidad_disponible = cantidad_disponible + cantidad_movimiento
        WHERE id_inventario = NEW.id_inventario;
    ELSE
        UPDATE inventarios
        SET cantidad_disponible = cantidad_disponible - cantidad_movimiento
        WHERE id_inventario = NEW.id_inventario;
    END IF;
END //
DELIMITER ;
select * from productos;
