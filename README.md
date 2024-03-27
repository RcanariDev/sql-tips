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


