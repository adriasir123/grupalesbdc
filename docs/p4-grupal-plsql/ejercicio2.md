# Ejercicio 2

> Realizar un procedimiento llamado MostrarInformes que recibirá tres parámetros. El primero determinará el tipo de informe que queremos obtener y los otros dos dependerán del tipo de informe.

## Código

- Mostrar Informes:

```sql
CREATE OR REPLACE PROCEDURE mostrarinformes(
    p_tipoinforme VARCHAR2,
    p_segundo carrerasprofesionales.fechahora%type,
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
BEGIN
    IF p_tipoinforme = 'tipo1' THEN
        informetipo1(p_segundo);
    ELSIF p_tipoinforme = 'tipo2' THEN
        informetipo2(p_tercero);
    ELSIF p_tipoinforme = 'tipo3' THEN
        informe_tipo3(p_tercero);
    END IF;
END mostrarinformes;
/
```

- Informe tipo1:

```sql
create or replace procedure informetipo1 (p_segundo carrerasprofesionales.FECHAHORA%type)
is
begin
    dbms_output.put_line('Fecha: ' || p_segundo );
    mostrar_horas(p_segundo);
end informetipo1;
/

create or replace procedure mostrar_horas(p_segundo carrerasprofesionales.FECHAHORA%type)
is
    cursor c_horas is select to_char(FECHAHORA,'HH24:MI') as hora, IMPORTEPREMIO from CARRERASPROFESIONALES where to_char(FECHAHORA,'DD-MM-YYYY')=p_segundo;
    v_posicion PARTICIPACIONES.POSICIONFINAL%type;
    v_ncaballo CABALLOS.nombre%type;
    v_njockey JOCKEYS.nombre%type;
begin
    for i in c_horas loop
        select POSICIONFINAL into v_posicion from PARTICIPACIONES where CODIGOCARRERA in (select CODIGOCARRERA from CARRERASPROFESIONALES where to_char(FECHAHORA,'DD-MM-YYYY')=p_segundo and to_char(FECHAHORA,'HH24:MI') = i.hora);
        select nombre into v_ncaballo from CABALLOS where CODIGOCABALLO in (select CODIGOCABALLO from PARTICIPACIONES where POSICIONFINAL=v_posicion and CODIGOCARRERA in (select CODIGOCARRERA from CARRERASPROFESIONALES where to_char(FECHAHORA,'DD-MM-YYYY')=p_segundo and to_char(FECHAHORA,'HH24:MI') = i.hora));
        select nombre into v_njockey from JOCKEYS where DNI in (select DNIJOCKEY from PARTICIPACIONES where POSICIONFINAL=v_posicion and CODIGOCARRERA in (select CODIGOCARRERA from CARRERASPROFESIONALES where to_char(FECHAHORA,'DD-MM-YYYY')=p_segundo and to_char(FECHAHORA,'HH24:MI') = i.hora));
        dbms_output.put_line('Hora: ' || i.hora || ' Importe Premio: ' || i.importepremio );
        dbms_output.put_line( 'Clasificación: ' );
        dbms_output.put_line( v_posicion || ' ' || v_ncaballo || ' ' || v_njockey );
    end loop;
end mostrar_horas;
/
```

- Informe tipo2:

```sql
CREATE OR REPLACE PROCEDURE informetipo2 (
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
BEGIN
    dbms_output.put_line('Código Carrera: ' || p_tercero );
    mostrar_fecha(p_tercero);
    mostrar_horas2(p_tercero);
    importe_premio(p_tercero);
    caballos_nacidos(p_tercero);
    mostrar_clasificacion(p_tercero);
END informetipo2;
/

CREATE OR REPLACE PROCEDURE mostrar_fecha (
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
    v_fecha carrerasprofesionales.fechahora%type;
BEGIN
    SELECT fechahora INTO v_fecha
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;
    
    dbms_output.put_line('Fecha: ' || v_fecha );
END mostrar_fecha;
/

CREATE OR REPLACE PROCEDURE mostrar_horas2(
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
    v_hora caballos.nombre%type;
BEGIN
    SELECT to_char(fechahora, 'HH24:MM') INTO v_hora
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;
    
    dbms_output.put_line('Hora: ' || v_hora );
END mostrar_horas2;
/

CREATE OR REPLACE PROCEDURE importe_premio(
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
    v_importe carrerasprofesionales.importepremio%type;
    v_limite  carrerasprofesionales.limiteapuesta%type;
BEGIN
    SELECT importepremio INTO v_importe
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;

    SELECT limiteapuesta INTO v_limite
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;
    
    dbms_output.put_line('Importe Premio: ' || v_importe || '       ' || 'Límite apuesta: ' || v_limite);
END importe_premio;
/

CREATE OR REPLACE PROCEDURE caballos_nacidos(
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
    v_min carrerasprofesionales.fechanacmin%type;
    v_mac carrerasprofesionales.fechanacmac%type;
BEGIN
    SELECT fechanacmin INTO v_min
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;

    SELECT fechanacmac INTO v_mac
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;
    
    dbms_output.put_line('Caballos nacidos de : ' || v_min || ' a ' || v_mac);
END caballos_nacidos;
/

create or replace procedure mostrar_clasificacion(p_tercero carrerasprofesionales.CODIGOCARRERA%type)
is
    cursor c_posiciones is select POSICIONFINAL from PARTICIPACIONES where CODIGOCARRERA=p_tercero;
    v_ncaballo CABALLOS.nombre%type;
    v_njockey JOCKEYS.nombre%type;
begin
    for i in c_posiciones loop
        select nombre into v_ncaballo from CABALLOS where CODIGOCABALLO in (select codigocaballo from participaciones where POSICIONFINAL=i.posicionFinal and codigocarrera in (select codigocarrera from carrerasprofesionales where codigocarrera=p_tercero));
        select nombre into v_njockey from JOCKEYS where DNI in (select DNIJOCKEY from participaciones where posicionfinal=i.posicionFinal and codigocarrera in (select codigocarrera from carrerasprofesionales where codigocarrera=p_tercero));
        dbms_output.put_line( i.posicionFinal || ' º ' || v_ncaballo || ' ' || v_njockey );
    end loop;
end mostrar_clasificacion;
/
```

- Informe tipo3:

```sql
CREATE OR REPLACE PROCEDURE informe_tipo3(
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
BEGIN
    comprobar_fecha(p_tercero);
    dbms_output.put_line('Código Carrera: ' || p_tercero );
    mostrar_fecha(p_tercero);
    mostrar_horas3(p_tercero);
    caballos_participantes(p_tercero);
END informe_tipo3;
/

CREATE OR REPLACE PROCEDURE mostrar_horas3(
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
    v_hora   date;
    v_limite carrerasprofesionales.limiteapuesta%type;
BEGIN
    SELECT to_char(fechahora, 'HH24:MM') INTO v_hora
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;

    SELECT limiteapuesta INTO v_limite
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;
    
    dbms_output.put_line('Hora: ' || v_hora || ' Limite apuesta: ' || v_limite );
END mostrar_horas3;
/

create or replace procedure caballos_participantes(p_tercero carrerasprofesionales.CODIGOCARRERA%type)
is
    cursor c_caballos is select nombre from caballos where CODIGOCABALLO in (select codigocaballo from participaciones where codigocarrera in (select codigocarrera from carrerasprofesionales where codigocarrera=p_tercero));
    v_jockey JOCKEYS.nombre%type;
    v_tantoauno apuestas.TANTOAUNO%type;
    n_victorias number;
    n_fechaultima CARRERASPROFESIONALES.FECHAHORA%type;
    n_posicion  PARTICIPACIONES.POSICIONFINAL%type;
begin
    dbms_output.put_line( 'Caballos Participantes: ' );
    for i in c_caballos loop
        select nombre into v_jockey from JOCKEYS where DNI in (select DNIJOCKEY from participaciones where CODIGOCABALLO in (select CODIGOCABALLO from CABALLOS where nombre=i.nombre));
        select count(POSICIONFINAL) into n_victorias from participaciones where POSICIONFINAL=1 and CODIGOCABALLO in (select CODIGOCABALLO from caballos where nombre=i.nombre);
        select max(FECHAHORA) into n_fechaultima from CARRERASPROFESIONALES where CODIGOCARRERA in (select CODIGOCARRERA from PARTICIPACIONES where CODIGOCABALLO in (select codigocaballo from CABALLOS where nombre=i.nombre));
        select tantoauno from apuestas where codigocarrera in (select codigocarrera from CARRERASPROFESIONALES where codigocarrera=p_tercero and CODIGOCABALLO in (select CODIGOCABALLO from CABALLOS where nombre=i.nombre)); 
        select posicionFinal into n_posicion from PARTICIPACIONES where CODIGOCARRERA in (select CODIGOCARRERA from CARRERASPROFESIONALES where fechahora=n_fechaultima and CODIGOCABALLO in (select CODIGOCABALLO from caballos where nombre=i.nombre));
        dbms_output.put_line( 'Nombre: ' || i.nombre || ' Jockey: ' || v_jockey || ' Tanto a uno: ' || v_tantoauno || ' NumVictorias: ' || n_victorias || ' Ultima Participacion: ' || n_fechaultima || ' Posicion: ' || n_posicion );
    end loop;
end caballos_participantes;
/
```

- Excepción de errores:

```sql
CREATE OR REPLACE PROCEDURE comprobar_fecha (
    p_tercero carrerasprofesionales.codigocarrera%type
) IS
    v_fecha carrerasprofesionales.fechahora%type;
BEGIN
    SELECT fechahora INTO v_fecha
    FROM carrerasprofesionales
    WHERE codigocarrera=p_tercero;
    
    IF v_fecha < '01-01-2017' THEN
        raise_application_error(-20001, 'La carrera ya se ha celebrado');
    END IF;
END;
/
```

## Prueba de funcionamiento

- Tipo 1:

    ```sql
    SQL> exec MostrarInformes('tipo1',08-01-2017,1);

    Fecha: 08-JAN-17
    Hora: 09:00      Importe Premio: 8750
    Clasificación:  
    1 º Dagoberto N. de Julian
    4 º Mayo J.Gelabert
    7 º Daniela B.Fayos
    ```

- Tipo 2:

    ```sql
    SQL> exec MostrarInformes('tipo2',08-01-2017,1);

    Código Carrera: 1
    Fecha: 08-JAN-17
    Hora: 09:01
    Importe Premio: 8750       Límite apuesta: 650
    Caballos nacidos de : 01-JAN-10 a 31-DEC-12
    1 º Dagoberto N. de Julian
    4 º Mayo J.Gelabert
    7 º Daniela B.Fayos

    PL/SQL procedure successfully completed.
    ```

- Tipo 3:

    ```sql
    SQL> exec MostrarInformes('tipo3',1);

    Código Carrera: 1
    Fecha: 08-JAN-17
    Hora: 09:01 Limite apuesta: 650
    Caballos Participantes:
    Nombre: Dagoberto Jockey: N. de Julian Tanto a uno: 6.8  NumVictorias: 1 Ultima Participacion: 13-NOV-16 Posicion: 8
    Nombre: Mayo Jockey: J.Gelabert Tanto a uno: 4.3  NumVictorias: 1 Ultima Participacion: 13-NOV-16 Posicion: 7
    Nombre: Daniela Jockey: B.Fayos Tanto a uno: 2.1 NumVictorias: 0 Ultima Participacion: 13-NOV-16 Posicion: 5

    PL/SQL procedure successfully completed.
    ```
