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

## 12. Crear procedimiento - 1 loop


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

## 13. Crear procedimiento - 2 loop

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

			--set @v_nth = (select count(*) from ##FactFecha where DATEPART(YEAR, Fecha) = DATEPART(year, @Fecha) and DATEPART(MONTH, Fecha) = DATEPART(MONTH, @Fecha) and DATEPART(WEEKDAY, Fecha) = DATEPART(WEEKDAY, @Fecha) and Fecha < @Fecha) + 1;

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

			--set @v_nth = (
			--	with TablaGeneral as (

			--	select Fecha, Dia, Mes, Anio, DiaSemana, NombreDiaSemana, NthOccurrence, count(*) Total
			--	from ##FactFecha
			--	group by Fecha, Dia, Mes, Anio, DiaSemana, NombreDiaSemana, NthOccurrence

			--	)
			--	select count(*)
			--	from TablaGeneral
			--	where DATEPART(YEAR, Fecha) = DATEPART(year, convert(date, @Fecha)) 
			--	and DATEPART(MONTH, Fecha) = DATEPART(MONTH, @Fecha) 
			--	and DATEPART(WEEKDAY, Fecha) = DATEPART(WEEKDAY, @Fecha) 
			--	and Fecha < @Fecha
			--) + 1;

			insert into ##FactFecha(Fecha, Hora, Dia, Mes, Anio, DiaSemana, NombreDiaSemana, SemanaMes, NthOccurrence)
			values(@Fecha, @Hora, DATEPART(DAY, @Fecha), DATEPART(MONTH, @Fecha), DATEPART(YEAR, @Fecha), DATEPART(WEEKDAY, @Fecha) + 1, datename(WEEKDAY, @Fecha), FLOOR((datepart(day, @Fecha)-1)/7)+1, @v_nth);

			set @Hora = @Hora + 1
			


			end
	set @Fecha = DATEADD(day, 1, @Fecha);
	end

end
```
