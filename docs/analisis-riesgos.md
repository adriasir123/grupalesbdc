# Análisis de riesgos

## Incendio

### Identificación

Fuego grande/mediano/pequeño que destruye lo que no debería quemarse.

### Causas
- Fuegos latentes, originados por condensadores o fuentes de alimentación averiadas
- Sobrecalentamiendo del cableado o de los componentes
- Cortocircuito
- Intencionado

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        | <---------- |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       | <---------- |
| BAJA        |             |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN |
| -----------   | -----------   |
| Crítico       | Perder todo el CPD, cierra la empresa         |
| Alto          | Perder 5 servidores de clientes importantes   |
| Medio         | Perder sala de trabajo                        |
| Bajo          | Perder cables, switches, tomas de red en sala de trabajadores |

## Accesos no autorizados al CPD

### Identificación

Acceso indebido, sin autorización o contra derecho, con el fin de obtener una satisfacción de carácter intelectual o monetaria.

### Causas

- Coacción
- Fuerza bruta
- Fallo en el sistema de acceso
- Infiltración (robo de huella + averiguar el pin)

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        | <---------- |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN |
| -----------   | -----------   |
| Crítico       | Acceso directo a la sala del CPD *(robo de datos, instalación de software malicioso de propósito específico, destrozo de material, robo de servidores)*         |
| Bajo          | Alguien quedándose atrapado en el área intermedia (anti-passback) al intentar acceder |

## Interrupciones suministro eléctrico

### Identificación

Evento durante el cual el voltaje cae a cero y no retorna a sus valores normales automáticamente.  
Más de 3 minutos se considera larga interrupción.

### Causas
- Mantenimiento programado de la red eléctrica
- Apagón
- Sabotaje de la red eléctrica malintenciado
- Catástrofe natural

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      | <---------- |
| ALTA        |             |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN |
| -----------   | -----------   |
| Crítico       | Pico de tensión y sistemas de protección insuficientes |
| Alto          | Perder un porcentaje de los servidores             |
| Medio         | No dar servicio durante un corto periodo de tiempo |
| Bajo          | Que haya apagón y se usen los SAIs/generadores |

## Averías en la Electrónica

### Identificación

Son producidas por un suministro incorrecto de energía, fluctuación de esta, o exceso de temperatura.

### Causas
- Desgaste por fin de vida útil *(incluso obsolescencia programada)*
- Subidas de tensión
- Corrosión por humedad
- Defectos de fábrica
- Pulsos electromagnéticos

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      | <---------- |
| ALTA        |             |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       | <---------- |
| BAJA        |             |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Crítico       | Averías en masa de CPUs              |
| Alto          | Averías de routers/switches          |
| Medio         | Averías de fuentes de alimentación   |
| Bajo          | Averías de Discos duros/memorias ram |

## Ransomware

### Identificación

Tipo de malware que impide a los usuarios acceder a su sistema o a sus archivos personales y que exige el pago de un rescate para poder acceder de nuevo a ellos.

### Causas
- Abrir adjuntos de correos desconocidos
- Sitios web maliciosos
- Spam malicioso

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       | <---------- |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Crítico       | Todos los discos duros/copias de seguridad encriptados |
| Alto          | Todo encriptado, pero poder restaurar desde copias de seguridad |
| Bajo          | Pocos datos encriptados, y posibilidad de recuperar desde copias de seguridad rápidamente |

## Robo de dispositivos móviles

### Identificación

Sustracción en contra de la voluntad, debido a la presencia de violencia física, violencia verbal o fuerza sobre las cosas.

### Causas
- Pérdida
- Robo

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       | <---------- |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Crítico       | Conseguir credenciales de administradores y robo de datos sensible  |
| Medio         | Destrucción de dispositivos                         |
| Bajo          | Robo de dispositivos, pero tener posibilidad de bloqueo remoto |

## Phishing

### Identificación

Estafa que tiene como objetivo obtener a través de internet datos privados de los usuarios, especialmente para acceder a sus cuentas o datos bancarios.

### Causas
- Personal no formado
- Fallo de seguridad
- Man in the middle

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        | <---------- |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Crítico       | Conseguir credenciales de administradores, y robo de datos sensible      |
| Alto          | Ejecución de código/aplicaciones no deseado |
| Medio         | Conseguir credenciales de usuarios con permisos especiales |
| Bajo          | Conseguir credenciales con permisos mínimos / que expiren pronto |

## Compromiso de contraseñas

### Identificación

Obtención de contraseñas de manera no legítima.

### Causas
- Fallo humano
- Algoritmos de encriptación débiles/desactualizados
- Filtración

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       | <---------- |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Crítico       | Conseguir credenciales de administradores, y robo de datos sensible |
| Alto          | Ejecución de código/aplicaciones no deseado |
| Medio         | Conseguir credenciales de usuarios con permisos especiales |
| Bajo          | Conseguir credenciales con permisos mínimos / que expiren pronto |

## Exploit de vulnerabilidades

### Identificación

Un exploit es cualquier ataque que aprovecha las vulnerabilidades de las aplicaciones, las redes, los sistemas operativos o el hardware. Por lo general, los exploits toman la forma de un programa de software o una secuencia de código previsto para hacerse con el control de los ordenadores o robar datos de red.

### Causas
- Fallos de seguridad *(puertos abiertos innecesarios, cortafuegos mal configurado)*
- Sistemas/programas desactualizados

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        | <---------- |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Crítico       | Conseguir control total del sistema  |
| Alto          | Robo de información sensible         |
| Medio         | Conseguir acceso con un usuario normal *(pocos permisos)* |
| Bajo          | Explotar la vulnerabilidad de protocolos inseguros |

## Ataques de denegación de servicio

### Identificación

Ataque informático que persigue que un sistema de ordenadores, un servicio o recurso, sea inaccesible para los usuarios legítimos.

### Causas
- No tener medidas de protección implementadas
- Ataque con motivos políticos

### Valoración

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      | <---------- |
| ALTA        |             |
| MEDIA       |             |
| BAJA        |             |
| NULA        |             |

*(sin medidas de prevención)*

| PROBABILIDAD| SELECCIÓN   |
| ----------- | ----------- |
| SEGURA      |             |
| ALTA        |             |
| MEDIA       |             |
| BAJA        | <---------- |
| NULA        |             |

*(con medidas de prevención)*

| IMPACTO/COSTE | DESCRIPCIÓN                          |
| -----------   | -----------                          |
| Alto          | Denegación de servicios imporantes |
| Medio         | Ralentización de servicios |
| Bajo          | Generación de ruido |
