# Protección contra amenazas (+ coste).

Incendios

 * Detectores y alarmas de humo.
 
 * Sistema de gases inertes que tiene dos formas de actuar para suprimir dos tercios del triángulo de fuego (combustible, comburente y energía de activación):
 
    * Ante una detección de subida de temperatura, que se considere peligrosa, libera halocarbonos que contribuyen a disipar el calor afectando a la energía de activación, en este caso, que es una alta temperatura.

 
    * Cuando los detectores de humo se activen, este sistema libera un gas inerte como el Árgon y este desplaza el oxígeno elimando el comburente de dicho triángulo.
    ![Apagafuegos](https://www.missioncriticalmagazine.com/ext/resources/MC/2020/07-08_July-August/Fire-Suppression-Fig2-900x550.jpg)

    * Un ejemplo de la distribución de este sistema:
    ![Sistema](https://www.siex2001.com/sites/default/files/imagecache/foto-info-sistemas/sistemas/imagenes/salainertes.jpg)

 * Activación de scripts de resguardo de información. 

 * Revisiones periódicas de toda la instalación eléctrica, así como el sistema de extinción y detección de incendios.

 * Control de todas las obras y trabajos que puedan realizarse en el entorno de
la empresa para evitar posibles accidentes por manejo de materiales inflamables o uso de herramientas que puedan generar chispas.


Accesos no autorizados al CPD

 * Un sistema de alarma que sirva tanto para los momentos de ausencia del personal, así como una activa por si intentan acceder posibles intrusos y den una respuesta rápida a las autoridades. A tener en cuenta el refuerzo de potenciales puntos de acceso como puertas y ventanas.

 * Puerta de acceso a la sala de servidores con blindaje y que incluya un sistema biómetrico, un modelo bastante interesante a este tipo de acceso es el de doble verificación (huella y un pin); además incluye una pequeña cámara que realiza una foto al ser usado, aportando una seguridad extra.

 ![Imagen de ejmplo](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse2.mm.bing.net%2Fth%3Fid%3DOIP.yxvwoprNGes-TtYsAp8OawHaHa%26pid%3DApi&f=1)

 * Cámaras de seguridad interiores (cpd) en combinación con algunas perimetrales para tener controlado el recinto de la empresa. Importante que las cámaras tengan capacidad de captar imágenes en entornos de baja luminosidad.


Interrupciones suministro eléctrico

 * SAIS, que permitan mantener los servidores encendidos el tiempo suficiente para activar la generación de copias de respaldo, alertas a los administradores y llegado a un periodo duradero de no poder recuperar el suministro, organizar un apagado controlado de los servidores.
 
 * Generador de gasolina, es un complemento para los SAIS en caso de momentos extremos y que su principal ventaja es que alarga más el tiempo de margen para recueperar la energía principal.


Averías en la electrónica

 * RAID en combinación a discos duros de repuesto, contribuirán a que no haya pérdida de datos para los clientes.

 * Otros componentes de repuesto como fuentes de alimentación, memoria RAM, cableado, switches y routers de repuesto.
 
 * Para casos más extremos de averías, la alta disponibilidad y el servicio estaría garantizado entre local y la conmutación con AWS.


Ransomware

 * Copias de respaldo de manera periódica.
 
 * Uso de escáneres de vulnerabilidades sobre los equipos y los servidores para reforzar posibles puntos débiles.
 
 * Una política de mantener el software lo más actualizado posible, dentro de lo estable, y estar al tanto de los posibles arreglos de seguridad que vayan publicando los canales oficiales.
 
 * Formación y concienciación de los trabajadores en posibles fallos que se pueden cometer y los potenciales peligros a los que se puede someter una empresa de esta índole.


Robo de dispositivos móviles

- Estén encriptados
- Se guarden la menor información crítica posible
- Apps de rastreo y bloqueo
- Backups


Phishing
- Formar a trabajadores
- Firewall con Lista negra con páginas comunes de phishing



Compromiso de contraseñas
- Formar trabajadores
- Cambiar contraseñas regularmente
- Usar algoritmos de encriptación robustos
- Acceso con claves SSH y no con contraseña
- Herramientas de fortaleza de contraseñas



Exploits de vulnerabilidades
- Aplicar parches de seguridad
- Mantener los sistemas, paquetes y programas actualizados
- Desinstalar plugins del navegador que no son necesarios
- Tener solamente los puertos necesarios abiertos (y controlados)
- Review created/homemade/specialty applications – Depending on your business, you may use in-house self-created applications OR have a vendor create ap
- Instalar firewalls o aplicaciones de protección en puntos de frontera (SNORT con reglas que identifiquen)
- Escáneres de vulnerabilidades





Ataques de denegación de servicio
- Balanceador de carga
- ACLs (filtrado de tráfico entre subredes)
- Usar redes CDN (cloudflare)
- Mantener actualizado todo
- Aumentar el ancho de banda
- 3. Implement server-level DDoS protection
