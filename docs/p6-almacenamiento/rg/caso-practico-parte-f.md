# Parte F

> Lidia y Jaime dejan la empresa. Borrar los usuarios

```sql
DROP USER lidia CASCADE;
DROP USER jaime CASCADE;
```

> Borrar los tablespaces creados en el punto b (producción y ventas) sin dejar rastro de los tablespaces

```sql
DROP TABLESPACE ts_produccion INCLUDING CONTENTS AND DATAFILES;
DROP TABLESPACE ts_venta INCLUDING CONTENTS AND DATAFILES;
```
