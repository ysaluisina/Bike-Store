# ANÁLISIS DE VENTAS DE BICICLETAS 
## MÉTRICAS GENERALES DEL NEGOCIO

**1) Mostrar totales de productos, clientes, órdenes y tiendas**

```sql
SELECT
	(SELECT COUNT(*) FROM products) AS productos_totales,
	(SELECT ROUND(SUM(oi.quantity*oi.list_price),2) FROM order_items oi)AS facturación_total,
	(SELECT COUNT(*) FROM customers) AS clientes_totales,
	(SELECT COUNT(*) FROM orders) AS total_ordenes,
	(SELECT COUNT(*) FROM stores) AS tiendas_totales;
```
<img width="485" height="45" alt="image" src="https://github.com/user-attachments/assets/cccccd52-b74d-4143-a7e3-f54ed9e21661" />


**2) Mostrar precio de lista promedio**

```sql
SELECT
	ROUND(AVG(list_price),2) as precio_promedio
FROM products;
```
<img width="129" height="41" alt="image" src="https://github.com/user-attachments/assets/6a7b0230-1e8a-4b36-9ca6-21b05ff3bbb7" />


***El negocio de venta de bicicletas cuenta con 3 tiendas, un total de 1445 clientes, 321 productos disponibles y 1615 órdenes registradas.
El ingreso total facturado es de $809.704.662, y el precio promedio por producto es de $145.255,17.***


## DESGLOCE POR TIENDAS

**3) Mostrar como se distribuye el negocio en las 3 tiendas**

```sql
SELECT 
	sto.store_name,
	COUNT(DISTINCT o.customer_id)AS clientes_totales,
	COUNT(DISTINCT o.order_id) AS cantidad_ordenes,
	ROUND(SUM(oi.quantity*oi.list_price),2)AS ingreso_total,
	ROUND(SUM(oi.quantity*oi.list_price)*100.0/SUM(SUM(oi.quantity*oi.list_price)) OVER(),0)AS porcentaje_del_total
FROM stores sto
JOIN orders o
	ON sto.store_id=o.store_id
JOIN order_items oi
	ON o.order_id=oi.order_id
GROUP BY sto.store_name
ORDER BY ingreso_total DESC;
```
<img width="509" height="76" alt="image" src="https://github.com/user-attachments/assets/458cfeab-d93a-4c82-b14c-6698ad9ca7be" />

***La tienda Baldwin Bikes el 68% del total de ingresos, con mayor volumen de clientes.**

## ANÁLISIS DE PRODUCTOS Y CATEGORÍAS

**4) ¿Cuáles son las categorias con mayor facturación?** 

```sql
SELECT 
	c.category_name,
	SUM(oi.quantity*oi.list_price) AS total_facturado,
	ROUND(SUM(oi.quantity*oi.list_price)*100.0/SUM(SUM(oi.quantity*oi.list_price)) OVER(),1) AS porcentaje
FROM dbo.categories c
JOIN products p
	ON c.category_id=p.category_id
JOIN order_items oi
	ON p.product_id=oi.product_id
GROUP BY c.category_name
ORDER BY total_facturado DESC;
```
<img width="296" height="154" alt="image" src="https://github.com/user-attachments/assets/6ee9aee6-4180-4acf-b5b5-61432580282c" />

***Las categoría 'Mountain Bikes' (36,8%) y 'Road Bikes' (22,5%) son las que concentran casi el 60% de la facturación.***

**5) El producto mas vendido fue 'Surly Ice Cream Truck Frameset', con 110 unidades.**

```sql
SELECT TOP 1
	p.product_id,
	p.product_name,
	COUNT(oi.product_id) AS mas_vendido
FROM products p
JOIN order_items oi
	ON p.product_id=oi.product_id
GROUP BY p.product_id, p.product_name
ORDER BY mas_vendido DESC;
```
<img width="381" height="37" alt="image" src="https://github.com/user-attachments/assets/97e28aeb-50b0-4d0d-a228-bb0d1d24fc48" />

**6) TOP 5 de productos por CANTIDAD de unidades vendidas**

```sql
SELECT TOP 5
    p.product_id,
    p.product_name,
    SUM(oi.quantity)AS cantidad_total,
	SUM(oi.quantity*oi.list_price) AS monto_total
FROM products p
JOIN order_items oi
    ON p.product_id=oi.product_id
GROUP BY p.product_id,p.product_name
ORDER BY cantidad_total DESC;
```
<img width="480" height="120" alt="image" src="https://github.com/user-attachments/assets/17e121b6-ab24-49f0-a789-66235d9a8b3b" />

**7) Productos con mayor ticket promedio de ventas (con un mínimo de 5 ventas)**
```sql
SELECT 
    p.product_id,
    p.product_name,
    c.category_name,
    COUNT(oi.item_id) AS items,
	ROUND(AVG ( oi.quantity * oi.list_price),1)	AS prom_valor_transaccion
FROM dbo.products p
JOIN dbo.order_items oi
    ON p.product_id = oi.product_id
JOIN dbo.categories c
    ON p.category_id = c.category_id
GROUP BY p.product_id, p.product_name, c.category_name
HAVING COUNT(oi.item_id) >= 5
ORDER BY prom_valor_transaccion DESC;
```
<img width="560" height="380" alt="image" src="https://github.com/user-attachments/assets/770d2b6a-e98e-4768-a4fc-0b2a8d7693a4" />

***Se muestran los productos con mayor ticket promedio de ventas***


**8) Analisis de compra: Valor promedio de pedido e items promedio de cada orden por categoría (con Expresión de Tabla Común - CTE)**

```sql
WITH item_por_pedido AS
	(
    SELECT order_id, COUNT(*) AS item_por_pedido
    FROM dbo.order_items
    GROUP BY order_id
    )
SELECT
c.category_name,
COUNT(DISTINCT o.order_id) AS ordenes_totales,
AVG (oi.list_price * oi.quantity) AS valor_promedio,
SUM (oi.list_price * oi.quantity) AS Total_categoria,
AVG(ipp.item_por_pedido) as item_promedio
FROM dbo.categories c
JOIN dbo.products p
    ON c.category_id = p.category_id
JOIN dbo.order_items oi
    ON oi.product_id=p.product_id
JOIN dbo.orders o
    ON o.order_id=oi.order_id
JOIN item_por_pedido ipp
    ON ipp.order_id=o.order_id
    GROUP BY c.category_name
    ORDER BY Total_categoria DESC;
```
<img width="499" height="153" alt="image" src="https://github.com/user-attachments/assets/ad3d2cc8-53fc-4f84-b0b7-b3d9225dc9ed" />

## Análisis y tendencias temporales

**9) Mostrar las tendencias de ventas mensuales: Ventas totales por mes (cantidad y monto), y tendencia de ventas temporales**

```sql
SELECT
	YEAR(o.order_date)AS anio,
	MONTH(o.order_date) AS mes,
	SUM(oi.quantity*oi.list_price) AS total_mensual
FROM orders o
JOIN order_items oi
	ON o.order_id=oi.order_id
GROUP BY YEAR(o.order_date),MONTH(o.order_date)
ORDER BY anio;
```

<img width="192" height="305" alt="image" src="https://github.com/user-attachments/assets/a4f10e93-7044-4018-af58-5b94cc80244a" />

**10) Mostrar los meses del año 2016 y la facturación total por mes de forma descendente ¿Cual fué el mes que tuvo mayor facturación?**

```sql
SELECT
    month (o.order_date) as mes,
    sum(oi.list_price * oi.quantity) as total_mes
FROM dbo.orders o
JOIN dbo.order_items oi
    on o.order_id = oi.order_id
WHERE YEAR(o.order_date) = '2016'
group by month(o.order_date)
order by total_mes desc;
```
<img width="134" height="248" alt="image" src="https://github.com/user-attachments/assets/b9a16430-ab4a-4a53-8f05-8b016356dbda" />

***El mes de septiembre fue el de mayor facturación durante el año 2016***

**11) Calcular el porcentaje sobre el promedio mensual**
```sql
WITH ventas_mensuales AS(
SELECT 
	MONTH(o.order_date) AS mes,
	SUM(oi.quantity*oi.list_price) AS venta_mensual
FROM orders o
JOIN order_items oi
	ON o.order_id=oi.order_id
GROUP BY MONTH(o.order_date))
SELECT 
	mes,
	venta_mensual,
	ROUND(venta_mensual*100.0/AVG(venta_mensual) OVER(),0) AS porcentaje_sobre_promedio
FROM ventas_mensuales
ORDER BY mes;
```
<img width="300" height="242" alt="image" src="https://github.com/user-attachments/assets/1b85550e-be25-4ba3-94b9-eb83b5fdd112" />

***Si bien, los datos se muestran ordenados por mes, rapidamente se puede visualizar que ABRIL es el mes en que se vendió 90% encima del porcentaje sobre promedio de ventas***

## ANÁLISIS DE CLIENTES: COMPORTAMIENTOS Y VALOR

**12)¿Cuál fué el monto gastado por cada cliente y que articulos compró?**

```sql
SELECT
	o.customer_id,
	COUNT(DISTINCT o.order_id) AS cantidad_compras,
	SUM(oi.quantity*oi.list_price) AS gasto_total,
CASE 
	WHEN COUNT(DISTINCT o.order_id)>=2
	THEN 'Cliente destacado'
	ELSE 'Cliente Estándar'
END AS 'Tipo de Cliente'
FROM orders o
JOIN order_items oi
	ON o.order_id=oi.order_id
GROUP BY o.customer_id
ORDER BY gasto_total DESC;
```
<img width="374" height="311" alt="image" src="https://github.com/user-attachments/assets/fa0e8e78-cbdb-4478-a525-f23e67f913a5" />


***Se crea un CASE para diferenciar los comportamientos de los clientes, teniendo en cuenta la cantidad de compras***

**13) ¿ Quiénes son los clientes mas valiosos? Mostrar TOP 10 de los clientes de alto valor**

```sql
SELECT TOP 10
	cu.customer_id,
	cu.first_name,
	cu.last_name,
	cu.email,
	COUNT(DISTINCT o.order_id) AS total_ordenes,
	SUM((oi.list_price * oi.quantity) - oi.discount)AS gasto_total,
	ROUND(AVG(oi.list_price),1)AS gasto_promedio
FROM customers cu
JOIN orders o
	ON cu.customer_id=o.customer_id
JOIN order_items oi
	ON o.order_id=oi.order_id
GROUP BY cu.customer_id,cu.first_name,cu.last_name,cu.email
ORDER BY gasto_total DESC;
```
<img width="630" height="212" alt="image" src="https://github.com/user-attachments/assets/4a9d1542-2729-416c-b442-6198bc1712d7" />

**14) Segmentación de clientes por recencia, frecuencia y valor monetario (RFM)**
```sql
WITH CustomerRFM AS(
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    DATEDIFF(DAY,MAX(o.order_date),GETDATE()) AS recency_days,
    COUNT(o.order_id) AS frecuency,
    SUM(oi.list_price * oi.quantity) as monetary_value
FROM customers c 
JOIN orders o 
    on c.customer_id = o.customer_id
JOIN order_items oi
    ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name,c.last_name
)

SELECT
    customer_id,
    first_name,
    last_name,
    recency_days,
    frecuency,
    monetary_value,
CASE
    WHEN recency_days <=30 THEN 'Activo'
    WHEN recency_days <=90 THEN 'Regular'
    ELSE 'Inactivo'
END AS recency_segment,
CASE   
    WHEN frecuency >=10 THEN 'Frecuente'
    WHEN frecuency >=5 THEN 'Frecuencia media'
    ELSE 'No frecuente'
END AS frecuency_segment,
CASE
    WHEN monetary_value >=5000 THEN 'Valor alto'
    WHEN monetary_value >=2000 THEN 'Valor medio'
    ELSE 'Valor bajo'
END AS monetary_segment
FROM CustomerRFM
ORDER BY monetary_value DESC;
```
<img width="778" height="402" alt="image" src="https://github.com/user-attachments/assets/ee8d9830-812c-43aa-9b63-3b85353fc957" />

**15) Creación de vista de resumen general del negocio de venta de bicicletas**
```sql
DROP VIEW IF EXISTS vw_bikesdb;
GO
CREATE VIEW vw_bikesdb AS
SELECT
    (SELECT COUNT(*) FROM customers) AS Clientes_Totales,
    (SELECT COUNT(*) FROM products) AS Total_Productos,
    (SELECT COUNT(*) FROM order_items) AS Total_Ventas,
    (SELECT SUM(list_price*quantity)FROM order_items) AS Importe_total,
    (SELECT AVG(list_price*quantity)FROM order_items) AS Promedio_importe,
    (SELECT COUNT(DISTINCT customer_id) FROM orders
    WHERE order_date>=DATEADD(MONTH,-2,GETDATE())) AS Customer_Active;
GO

SELECT * FROM vw_bikesdb;
```

<img width="593" height="45" alt="image" src="https://github.com/user-attachments/assets/f4cafcbd-6a0f-4934-8880-1493b26c59ad" />
