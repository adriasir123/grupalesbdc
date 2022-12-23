# Pasos previos

## Oracle

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

## PostgreSQL

Añado un nuevo usuario de sistema:

```shell
sudo adduser hipodromo_admin
```

Cambio al usuario `postgres`:

```shell
su - postgres
```

Descargo el script de generación de tablas y datos:

```shell
wget https://gist.githubusercontent.com/adriasir123/652ee842c3966696aeb6a07ab22a59b2/raw/715cf2683488b6aa9e8d481749620c282e5872c3/gran-hipodromo-postgres.sql
```

Accedo a la bd:

```shell
psql
```

Creo la bd y me conecto:

```sql
CREATE DATABASE hipodromo;
\connect hipodromo
```

Ejecuto el script:

```sql
\i gran-hipodromo-postgres.sql
```

Creo el usuario y doy privilegios:

```sql
create user hipodromo_admin with encrypted password '1234';
GRANT all privileges ON DATABASE hipodromo TO hipodromo_admin;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO hipodromo_admin;
```

Cambio a este usuario:

```sql
set role hipodromo_admin;
```
