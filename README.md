# sql-tips

<br />
<br />

## 11. Aplicar ***row_number() over*** a ciertas filas

- Aplicar solo a filas que no son vacias

```sql
select *
  , count(case when NombreCliente is not null then 1 end) over(partition by NombreCliente order by NombreProducto) as ContadorProducto
from DashVentasUnionTab12
order by Compania, Fecha, Hora
```
<br />

<p align="center">
  <img src="/img/img11.jpg" width=60% height=60%>
</p>

<br />
<br />

## 12. Crear procedimiento (Estructura de Fecha) - 1 loop

- Crear tabla temporal

```sql
create table ##FactFecha(
Fecha date not null
--Hora int
--Dia int,
--Mes int,
--Anio nvarchar(20),
--DiaSemana int,
--NombreDiaSemana nvarchar(20),
--SemanaMes int,
--NthOccurrence int
)
```

- Siempre incluir el **begin** después de ***while*** loop.

```sql
create or alter procedure EstructuraFecha
as
begin
	declare @Fecha date;
	declare @Hora int;

	set @Fecha = '2024-01-01';
	set @Hora = 1;

	while (@Fecha <= '2024-01-03')
		begin

		insert into ##FactFecha(Fecha)
		values(@Fecha);

		set @Fecha = DATEADD(day, 1, @Fecha);

		end
end
```

<br />
<br />

## 13. Crear procedimiento (Estructura de Fecha) - 2 loop

- Crear tabla temporal

```sql
create table ##FactFecha(
Fecha date not null,
Hora int,
Dia int,
Mes int,
Anio nvarchar(20),
DiaSemana int,
NombreDiaSemana nvarchar(20),
SemanaMes int,
NthOccurrence int
)
```


- Siempre incluir el **begin** después de ***while*** loop.

```sql
create or alter procedure EstructuraFecha
as
begin
	declare @Fecha date;
	declare @Hora int;
	declare @v_nth int;

	set @Fecha = '2020-01-01'; -- '2024-01-01'
	set @Hora = 0;

	while (@Fecha <= '2027-01-01') -- '2024-03-01'
	begin
	set @Hora = 0;
			while (@Hora <= 23)
			begin

			set @v_nth = (
			
			select count(*)
			from (

			select distinct Fecha, Dia, Mes, Anio, DiaSemana, NombreDiaSemana, NthOccurrence
			from ##FactFecha

			) A
			where DATEPART(YEAR, A.Fecha) = DATEPART(year, convert(date, @Fecha)) 
			and DATEPART(MONTH, A.Fecha) = DATEPART(MONTH, @Fecha) 
			and DATEPART(WEEKDAY, A.Fecha) = DATEPART(WEEKDAY, @Fecha) 
			and A.Fecha < @Fecha
			
			) + 1;

			insert into ##FactFecha(Fecha, Hora, Dia, Mes, Anio, DiaSemana, NombreDiaSemana, SemanaMes, NthOccurrence)
			values(@Fecha, @Hora, DATEPART(DAY, @Fecha), DATEPART(MONTH, @Fecha), DATEPART(YEAR, @Fecha), DATEPART(WEEKDAY, @Fecha) + 1, datename(WEEKDAY, @Fecha), FLOOR((datepart(day, @Fecha)-1)/7)+1, @v_nth);

			set @Hora = @Hora + 1
			


			end
	set @Fecha = DATEADD(day, 1, @Fecha);
	end

end
```

<br />
<br />

## 14. PIVOT - Pivotear tabla 

<br />

- Como estaba antes

<p align="center">
  <img src="/img/pivot11.PNG" width=20% height=20%>
</p>



<br />

- Utilizando CTE y la palabra reservada ***pivot***


<br />

```sql
with TablaGeneral16_2 as (

...

)
, TablaGeneral17 as (

select *
from TablaGeneral16_2
pivot (

sum(TotalCasos)
for TipoEstado in ([Anulado por el Vendedor], [Anulado por el Cliente], [Anulado por la Tienda], [Listo Recoger], [En Cocina], [Anulado por el Proveedor de Delivery]
					, [En Entrega], [Programado], [Entregado], [Anulado por el Sistema], [Completado], [En Preparación], [Pendiente])

) as PivotTable

)
select *
from TablaGeneral17

```

<p align="center">
  <img src="/img/pivot12.PNG" width=60% height=60%>
</p>

<br />

- Otro ejemplo

```sql
with TablaGeneral11 as (

select CONVERT(date, DATEADD(hour, -5, A.FechaOrden)) as FechaOrden, DATEPART(hour, DATEADD(hour, -5, A.FechaOrden)) as HoraOrden, D.NombreCompania, E.MetodoPago, count(*) as TotalAnuladosCanalVenta
from FactVentas A
left join DimCanalVenta B on A.IdCanalVenta = B.IdCanalVenta
left join DimEstados C on A.IdEstado = C.IdEstado
left join DimCompania D on A.IdCompania = D.IdCompania
left join DimMetodoPago E on A.IdMetodoPago = E.IdMetodoPago
where CONVERT(date, DATEADD(hour, -5, A.FechaOrden)) = CONVERT(date, '2023-03-31')
--where CONVERT(date, DATEADD(hour, -5, A.FechaOrden)) = CONVERT(date, getdate())
--where CONVERT(date, DATEADD(hour, -5, A.FechaOrden)) = CONVERT(date, @VarFecha)
and C.TipoEstado like 'Anulado%'
group by CONVERT(date, DATEADD(hour, -5, A.FechaOrden)), DATEPART(hour, DATEADD(hour, -5, A.FechaOrden)), D.NombreCompania, E.MetodoPago

)
, TablaGeneral12 as (

select FechaOrden, HoraOrden, NombreCompania, [YAPE] as ANULADOSyape, [RAPPI] as ANULADOSrappi, [ONLINE] as ANULADOSonline, [EFECTIVO] as ANULADOSefectivo, [Pago Link] as ANULADOSpagolink, [POS VISA] as ANULADOSposvisa, [POS MASTERCARD] as ANULADOSposmastercards, [PedidosYa] as ANULADOSpedidosya
from TablaGeneral11
pivot(

sum(TotalAnuladosCanalVenta)
for MetodoPago in ([YAPE], [RAPPI], [ONLINE], [EFECTIVO], [Pago Link], [POS VISA], [POS MASTERCARD], [PedidosYa]) 

) as PivotTable

)
select *
from TablaGeneral12
```


<br />
<br />

## 15. Crear funciones

<br />
 
- Primer ejemplo


```sql
create or alter function EtClienteDigital (
@Var1 varchar(50)
)
returns varchar(50)
	begin
		declare @Otro varchar(50)
		set @Otro = case when @Var1 in ('WhatsApp', 'Landing') then 'ClienteDigital'
					else 'Otro' end
	return @Otro
	end
go
```

<br />

- Segundo ejemplo

```sql
create or alter function CambiarEtiqueta (
@Var1 varchar(100)
)
returns varchar(100)
	begin
		declare @Otro varchar(100)
		set @Otro = case when @Var1 like 'Anulado%' then 'Anulado'
						when @Var1 = 'Completado' then 'Completado'
						when @Var1 = 'Listo Recoger' then 'ListoRecoger'
						when @Var1 = 'En Entrega' then 'EnEntrega'
						when @Var1 = 'En Preparación' then 'EnPreparacion'
						when @Var1 = 'En Cocina' then 'EnCocina'
						else @Var1 end
	return @Otro
	end
go
```


<br />
<br />

## 16. Crear secuencia de pasos PERO que no sea PROCEDURE

<br />

- Crear la tabla temporal

```sql
create table #FechaCompania(
Fecha date null,
Compania nvarchar(50) null
)
go
```


<br />

- Crear la secuencia de pasos y ejecutarla

```sql
declare @Var1 date;

set @Var1 = '2021-12-22';

while (@Var1 <= CONVERT(DATE, GETDATE()))
	begin

	insert into #FechaCompania
	values(@Var1, 'DON TITO'),
			(@Var1, 'PRIMOS CHICKEN BAR'),
			(@Var1, 'TORI')

	set @Var1 = DATEADD(day, 1, @Var1);

	end;
go
```

<br />

- Evaluar

```sql
select *
from #FechaCompania
go
```


<br />
<br />

## 17. Insertar data a través del SELECT

<br />

- Hay de 2 formas
  
1. Simple

```sql
insert into #NombreTabla
select *
from Tabla1
```

<br />

2. Uso de WITH

```sql
with Tabla1 as (
)
insert into #NombreTabla
select *
from Tabla1

```


<br />
<br />

## 18. Obtener decimales de una división de enteros

<br />

- Se debe aplicar **CAST()** pero directamente a la variable y NO a la operación en general

```sql
select cast(Var1 as decimal(8, 2)) / cast(Var2 as decimal(8, 2)
from Tabla1
```  

- Otro ejemplo

```sql
select case when C.TotalOrdenes = 0 then cast(0 as decimal(8, 2)) else (cast(A.TotalOrdenes as decimal(8, 2)) - cast(C.TotalOrdenes as decimal(8, 2)))/cast(C.TotalOrdenes as decimal(8, 2))*100 end as PorTotalOrdenesHoyAyer
from Tabla1
```


<br />
<br />


## 19. Unir información con un periodo anterior

<br />

- Se tiene que hacer un **left join** y **dateadd()** pare restarle un periodo de tiempo (puede ser día, mes, etc)

```sql
select *
from Tabla1 A
left join Tabla1 B on DATEADD(day, -1, A.Fecha) = B.Fecha
```

<br />
<br />
