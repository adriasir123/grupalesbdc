# Parte C

c) Pepe quiere crear una tabla Prueba que ocupe inicialmente 256K en el tablespace Ventas.

El usuario Pepe se conecta con su usuario y crea la tabla Prueba con el el storage inicial a 256K y en el tablespace ts_venta, creada anteriormente.

```sql
CREATE TABLE Prueba (
  codigo varchar2(20),
  producto varchar2(20),
  constraint pk_codigo PRIMARY KEY(codigo))
  STORAGE (INITIAL 256K)
  TABLESPACE ts_venta;
```

**Captura de la creación:**

![Prueba](/img/capturas-arantxa/95.png)