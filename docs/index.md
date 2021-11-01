# Welcome to MkLorum

Empresa elegida: comvive-mini

En un polígono industrial

Documento de referencia:
https://dit.gonzalonazareno.org/moodle/pluginfile.php/14687/mod_resource/content/1/48ca15671b800.pdf


En nuestra práctica, estudiaremos los sistemas de una PYME de uno de los sectores que aparecen en https://itinerarios.incibe.es

Os recomiendo ver los vídeos relativos al sector elegido para realizar la práctica más fácilmente.
Además, la página del INCIBE es una referencia y os puede ayudar mucho, por ejemplo, a la hora de desarrollar un plan de contingencias:
https://www.incibe.es/protege-tu-empresa/que-te-interesa/plan-contingencia-continuidad-negocio

o un plan director de seguridad:
https://www.incibe.es/protege-tu-empresa/que-te-interesa/plan-director-seguridad


La práctica tendrá las siguientes fases:

    • Redacción de una política de seguridad en la que se detallen las medidas de protección y se incluya un plan de respuesta a las contingencias, especificando que hacer cuando alguno de los riesgos estudiados se haga realidad.


AL FINAL, SE ENTREGAN 2 SECCIONES PRINCIPALES:
- Plan de seguridad
 - Identificación y valoración de los activos del sistema.
     -  INVENTARIO de HARDWARE
     -  INVENTARIO SOFTWARE
     -  INVENTARIO COMUNICACIONES
     -  DATOS A PROTEGER


  - Identificación de las amenazas para esos activos.

Incendios

Descripción de la amenaza

Las causas de un incendio en un CPD:
- Cortocircuito
- fuegos latentes, originados por condensadores o fuentes de alimentación averiadas
- Intencionado
- Sobrecalentamiendo del cableado o de los equipos


Accesos no autorizados al CPD









Interrupciones suministro eléctrico



Averías en la Electrónica de Red
Ransomware
Robo de dispositivos móviles
Phishing
Compromiso de contraseñas
Exploit de vulnerabilidades
Ataques de denegación de servicio





  • Valoración de los riesgos identificados

  Probabilidad de que pase (a lo largo del tiempo)
  (SIN MEDIDAS, CON medidas de prevención, )
  (si es nula con medidas de preveción, no debería de aparecer pero aparece en el plan de contingencia, decirlo en el plan de contingencia)
  (ORDENADO POR probabilidad aparecen al principio)

  Probabilidad de que pase (a lo largo del tiempo)

  SEGURA
  ALTO
  MEDIO
  BAJO
  NULA (seguro que no va a pasar)

  Impacto (coste)

  CRÍTICO
  ALTO
  MEDIO
  BAJO


  TABLA COMO RESUMEN





  Incendios


Probabilidad de que pase (a lo largo del tiempo)

SEGURA
ALTO
MEDIO X
BAJO
NULA

Impacto (coste)

En función de la gravedad, el impactó podrá ser...

CRÍTICO - Perder todo el CPD (cerrar empresa)
ALTO - Perder 5 servidores
MEDIO - Perder sala de trabajo
BAJO - Perder cables o switch, enchufes de la sala de trabajadores

Apartado con las medidas aplicadas una estimación





  Accesos no autorizados al CPD

  Probabilidad de que pase (a lo largo del tiempo)

  SEGURA
  ALTO
  MEDIO
  BAJO X
  NULA

  Impacto (coste)

  CRÍTICO - Sala del CPD directamente
  ALTO -
  MEDIO -
  BAJO -






  Interrupciones suministro eléctrico
  Averías en la Electrónica de Red
  Ransomware



  Robo de dispositivos móviles
  Phishing
  Compromiso de contraseñas
  Exploit de vulnerabilidades
  Ataques de denegación de servicio




  • Medidas de protección apropiadas para los riesgos (con coste).

Incendios

   * Detectores y alarmas de humos.

   * Dispositivos automáticos de extinción de incendios

   Extinción de oxígeno (https://www.missioncriticalmagazine.com/ext/resources/MC/2020/07-08_July-August/Fire-Suppression-Fig2-900x550.jpg)

   Gas inerte que se combina con el oxígeno en una reacción química, y lo elimina

   Desconexión automática de la corriente

   Revisiones periódicas de toda la instalación eléctrica.

   Revisiones de todas las obras y trabajos que se realicen en el entorno de
  la empresa para evitar posibles accidentes debido al manejo de materiales inflamables.




Accesos no autorizados al CPD

 alarma de la empresa cuando no se
encuentre nadie dentro de la oficina.

- Alarma con sensores en los potenciales puntos de acceso, puertas y vetanas.

- Puerta blindada con acceso restringido a los servidores (acceso biométrico).

- Cámaras de seguridad interiores (cpd).





  Interrupciones suministro eléctrico

  - SAIS
  - Generador de gasolina
  - Backup



  Averías en la Electrónica de Red
  - RAID
  - Componentes de repuesto (discos, RAM)
  - Cableado y switches/routers de repuesto
  - Backups y alta disponibilidad entre local y remoto AWS



  Ransomware

  - Backups
  - Tener un plan de reaccións
  - Escáneres de vulnerabilidades
  - Actualizar software
  - Formar a los trabajadores en buenas prácticas







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









- Plan de contingencias  (soluciones, plan de respuesta)

según daños

Pérdida de datos (BD, configuraciones... )

Todo lo que se pueda perder que haga que se pare el negocio

Si se estropea el router principal

Rompen servidores
