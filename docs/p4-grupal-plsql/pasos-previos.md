# Pasos previos

Descargo el script de generación de tablas y datos:

```shell
wget https://gist.githubusercontent.com/adriasir123/0896abce2b81714557d3c1e2a038c3cb/raw/312c27d4e56b292cfb4fb1d235eacb781b98a4e4/gran-hipodromo-oracle.sql
```

Accedo como `sys`:

```shell
sqlplus / as sysdba
```

Creo el usuario y le doy privilegios:

```sql
create user hipodromo identified by 1234;
grant all privileges to hipodromo;
```

Cambio a este usuario:

```sql
connect hipodromo/1234;
```

Ejecuto el script:

```sql
@gran-hipodromo-oracle.sql
```
