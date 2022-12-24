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
    CREATE OR REPLACE PROCEDURE informetipo1 (
        p_segundo carrerasprofesionales.fechahora%type
    ) IS
    BEGIN
        dbms_output.put_line('Fecha: ' || p_segundo );
        mostrar_horas(p_segundo);
    END informetipo1;
    /

    CREATE OR REPLACE PROCEDURE mostrar_horas(
        p_segundo carrerasprofesionales.fechahora%type
    ) IS
        CURSOR c_horas IS
            SELECT to_char(fechahora, 'HH24:MI') AS hora, importepremio
            FROM carrerasprofesionales
            WHERE to_char(fechahora, 'DD-MM-YYYY')=p_segundo;

        v_posicion participaciones.posicionfinal%type;
        v_ncaballo caballos.nombre%type;
        v_njockey  jockeys.nombre%type;
    BEGIN
        FOR i IN c_horas LOOP
            SELECT posicionfinal INTO v_posicion
            FROM participaciones
            WHERE codigocarrera IN (
                    SELECT codigocarrera
                    FROM carrerasprofesionales
                    WHERE to_char(fechahora, 'DD-MM-YYYY')=p_segundo AND to_char(fechahora, 'HH24:MI') = i.hora
                );

            SELECT nombre INTO v_ncaballo
            FROM caballos
            WHERE codigocaballo IN (
                    SELECT codigocaballo
                    FROM participaciones
                    WHERE posicionfinal=v_posicion AND codigocarrera IN (
                            SELECT codigocarrera
                            FROM carrerasprofesionales
                            WHERE to_char(fechahora, 'DD-MM-YYYY')=p_segundo AND to_char(fechahora, 'HH24:MI') = i.hora
                        )
                );

            SELECT nombre INTO v_njockey
            FROM jockeys
            WHERE dni IN (
                    SELECT dnijockey
                    FROM participaciones
                    WHERE posicionfinal=v_posicion AND codigocarrera IN (
                            SELECT codigocarrera
                            FROM carrerasprofesionales
                            WHERE to_char(fechahora, 'DD-MM-YYYY')=p_segundo AND to_char(fechahora, 'HH24:MI') = i.hora
                        )
                );
                
            dbms_output.put_line('Hora: ' || i.hora || ' Importe Premio: ' || i.importepremio );
            dbms_output.put_line( 'Clasificación: ' );
            dbms_output.put_line( v_posicion || ' ' || v_ncaballo || ' ' || v_njockey );
        END LOOP;
    END mostrar_horas;
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

    CREATE OR REPLACE PROCEDURE mostrar_clasificacion(
        p_tercero carrerasprofesionales.codigocarrera%type
    ) IS
        CURSOR c_posiciones IS
            SELECT posicionfinal
            FROM participaciones
            WHERE codigocarrera=p_tercero;

        v_ncaballo caballos.nombre%type;
        v_njockey  jockeys.nombre%type;
    BEGIN
        FOR i IN c_posiciones LOOP
            SELECT nombre INTO v_ncaballo
            FROM caballos
            WHERE codigocaballo IN (
                    SELECT codigocaballo
                    FROM participaciones
                    WHERE posicionfinal=i.posicionfinal AND codigocarrera IN (
                            SELECT codigocarrera
                            FROM carrerasprofesionales
                            WHERE codigocarrera=p_tercero
                        )
                );

            SELECT nombre INTO v_njockey
            FROM jockeys
            WHERE dni IN (
                    SELECT dnijockey
                    FROM participaciones
                    WHERE posicionfinal=i.posicionfinal AND codigocarrera IN (
                            SELECT codigocarrera
                            FROM carrerasprofesionales
                            WHERE codigocarrera=p_tercero
                        )
                );
                
            dbms_output.put_line( i.posicionfinal || ' º ' || v_ncaballo || ' ' || v_njockey );
        END LOOP;
    END mostrar_clasificacion;
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

    CREATE OR REPLACE PROCEDURE caballos_participantes(
        p_tercero carrerasprofesionales.codigocarrera%type
    ) IS
        CURSOR c_caballos IS
            SELECT nombre
            FROM caballos
            WHERE codigocaballo IN (
                    SELECT codigocaballo
                    FROM participaciones
                    WHERE codigocarrera IN (
                            SELECT codigocarrera
                            FROM carrerasprofesionales
                            WHERE codigocarrera=p_tercero
                        )
                );

        v_jockey      jockeys.nombre%type;
        v_tantoauno   apuestas.tantoauno%type;
        n_victorias   NUMBER;
        n_fechaultima carrerasprofesionales.fechahora%type;
        n_posicion    participaciones.posicionfinal%type;
    BEGIN
        dbms_output.put_line( 'Caballos Participantes: ' );

        FOR i IN c_caballos LOOP
            SELECT nombre INTO v_jockey
            FROM jockeys
            WHERE dni IN (
                    SELECT dnijockey
                    FROM participaciones
                    WHERE codigocaballo IN (
                            SELECT codigocaballo
                            FROM caballos
                            WHERE nombre=i.nombre
                        )
                );

            SELECT COUNT(posicionfinal) INTO n_victorias
            FROM participaciones
            WHERE posicionfinal=1 AND codigocaballo IN (
                    SELECT codigocaballo
                    FROM caballos
                    WHERE nombre=i.nombre
                );

            SELECT MAX(fechahora) INTO n_fechaultima
            FROM carrerasprofesionales
            WHERE codigocarrera IN (
                    SELECT codigocarrera
                    FROM participaciones
                    WHERE codigocaballo IN (
                            SELECT codigocaballo
                            FROM caballos
                            WHERE nombre=i.nombre
                        )
                );

            SELECT tantoauno
            FROM apuestas
            WHERE codigocarrera IN (
                    SELECT codigocarrera
                    FROM carrerasprofesionales
                    WHERE codigocarrera=p_tercero AND codigocaballo IN (
                            SELECT codigocaballo
                            FROM caballos
                            WHERE nombre=i.nombre
                        )
                );

            SELECT posicionfinal INTO n_posicion
            FROM participaciones
            WHERE codigocarrera IN (
                    SELECT codigocarrera
                    FROM carrerasprofesionales
                    WHERE fechahora=n_fechaultima AND codigocaballo IN (
                            SELECT codigocaballo
                            FROM caballos
                            WHERE nombre=i.nombre
                        )
                );
                
            dbms_output.put_line( 'Nombre: ' || i.nombre || ' Jockey: ' || v_jockey || ' Tanto a uno: ' || v_tantoauno || ' NumVictorias: ' || n_victorias || ' Ultima Participacion: ' || n_fechaultima || ' Posicion: ' || n_posicion );
        END LOOP;
    END caballos_participantes;
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
