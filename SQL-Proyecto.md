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

***La tienda Baldwin Bikes el 68% del total de ingresos, con mayor volumen de clientes***
