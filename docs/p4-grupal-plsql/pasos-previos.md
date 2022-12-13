# Pasos previos

Descargo el script de generación de tablas y datos:

```shell
wget https://gist.githubusercontent.com/adriasir123/0896abce2b81714557d3c1e2a038c3cb/raw/d874402ccc93aee7bd323c23b8a035cf7d992f1a/gran-hipodromo-oracle.sql
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


```sql
insert into carrerasProfesionales values (1, to_date('08-01-2017 09:30', 'DD-MM-YYYY HH24:MI'),  8750, 650, to_date('01-01-2010', 'DD-MM-YYYY'), to_date('31-12-2012', 'DD-MM-YYYY'));
```







