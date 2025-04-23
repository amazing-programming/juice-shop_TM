# Análisis de Seguridad Detallado - OWASP Juice Shop

**Fecha:** 23 de abril de 2025  
**Analista:** GitHub Copilot  
**Versión analizada:** 17.3.0  

## Resumen Ejecutivo

OWASP Juice Shop es una aplicación web intencionalmente vulnerable diseñada como plataforma de entrenamiento en seguridad que incluye todas las vulnerabilidades del OWASP Top Ten. Este análisis detalla las vulnerabilidades presentes, evaluando su impacto y proporcionando recomendaciones específicas para su mitigación.

## Tecnologías y Componentes Identificados

- **Backend**: Node.js con Express.js, Sequelize ORM para base de datos
- **Frontend**: Angular 19 con Material UI
- **Base de datos**: SQLite (según dependencias en package.json)
- **Autenticación**: JWT (jsonwebtoken, express-jwt)
- **Testing**: Jasmine, Karma, Mocha, Cypress

## Patrones Arquitectónicos

- Arquitectura cliente-servidor con API REST
- Modelo MVC (Model-View-Controller)
- Separación entre frontend (Angular) y backend (Express)
- Autenticación basada en tokens JWT
- Sistema de roles y permisos (customer, deluxe, accounting, admin)

## 1. Arquitectura y Diseño

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿La aplicación sigue un patrón arquitectónico seguro? | Parcialmente | La aplicación implementa una arquitectura cliente-servidor con separación clara entre frontend y backend, pero intencionalmente incorpora prácticas inseguras como parte de su diseño educativo. La estructura del proyecto es coherente, pero existen numerosas vulnerabilidades deliberadas. |
| ¿Existe una adecuada separación de responsabilidades? | Sí | El código está organizado en directorios separados para modelos, rutas, vistas y lógica de negocio, siguiendo buenas prácticas de estructuración de código. |
| ¿Se implementa el principio de mínimo privilegio? | No | Como se observa en `lib/insecurity.ts`, el manejo de roles (customer, deluxe, accounting, admin) existe, pero hay múltiples formas de escalar privilegios debido a vulnerabilidades intencionales. |
| ¿Se utilizan mecanismos para prevenir ataques comunes? | Parcialmente | Existen intentos de sanitización de entrada (como `sanitizeHtml` en insecurity.ts), pero con implementaciones deliberadamente deficientes para permitir prácticas educativas de explotación. |
| ¿Los flujos de datos críticos están protegidos? | No | El código en `routes/login.ts` muestra una vulnerabilidad de SQL Injection deliberada en la consulta de autenticación. |

## 2. Autenticación y Autorización

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿El sistema de autenticación es robusto? | No | El sistema utiliza JWT, pero con implementaciones inseguras. En `lib/insecurity.ts` se observa que la clave privada está hardcodeada en el código fuente. |
| ¿Se implementa correctamente el manejo de sesiones? | No | El manejo de JWT en `lib/insecurity.ts` muestra debilidades deliberadas, como verificaciones inadecuadas de tokens. |
| ¿Existen controles para prevenir ataques de fuerza bruta? | No | No se observan mecanismos de rate limiting o bloqueo de cuentas después de múltiples intentos fallidos. |
| ¿Los secrets y claves se gestionan adecuadamente? | No | Las claves privadas están hardcodeadas en el código fuente (insecurity.ts), exponiendo material criptográfico sensible. |
| ¿Hay controles para la gestión segura de contraseñas? | No | En `routes/login.ts` se observa que las contraseñas se procesan con un simple hash MD5 (función `hash` en insecurity.ts), un algoritmo obsoleto y vulnerable. |
| ¿El sistema implementa protección contra ataques de timing para autenticación? | No | No hay evidencia de protecciones contra ataques de timing en el proceso de autenticación. |
| ¿Los tokens de sesión se transmiten de manera segura? | Parcialmente | Se utilizan tokens JWT, pero sin asegurar que se transmitan solo a través de HTTPS. |
| ¿El proceso de restablecimiento de contraseñas es seguro? | No | En `server.ts` se identifica una vulnerabilidad en la ruta `/rest/user/reset-password` con un rate limiting insuficiente que usa solo la IP de origen. |

## 3. Manejo de Entrada y Sanitización

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿Se validan adecuadamente las entradas del usuario? | No | La función de login en `routes/login.ts` muestra una vulnerabilidad de SQL Injection, permitiendo la entrada directa de parámetros en consultas SQL. |
| ¿Existe protección contra XSS? | Parcialmente | Se implementan funciones de sanitización HTML en `lib/insecurity.ts`, pero con fallos deliberados para permitir ataques XSS. |
| ¿Hay protección contra CSRF? | No | No se observan mecanismos efectivos de protección contra CSRF en los archivos analizados. |
| ¿Se controlan adecuadamente los redirects? | No | La función `isRedirectAllowed` en `lib/insecurity.ts` muestra una implementación vulnerable que permite open redirects. |
| ¿Se gestiona correctamente el upload de archivos? | No | En `routes/fileUpload.ts` se procesan archivos ZIP, XML y YAML con validaciones insuficientes, permitiendo path traversal, XXE y ataques de deserialización. |
| ¿La aplicación implementa validación adecuada para subidas de archivos? | No | En `routes/fileUpload.ts` se procesan archivos ZIP, XML y YAML con validaciones insuficientes, permitiendo path traversal, XXE y ataques de deserialización. |
| ¿Se implementan controles para prevenir la ejecución de código malicioso? | Parcialmente | Hay intentos de aislamiento utilizando sandbox con `vm.createContext` en el procesamiento de XML y YAML, pero con opciones inseguras que permiten exploits. |
| ¿Existen medidas para prevenir ataques DoS relacionados con el tamaño o contenido de archivos? | Parcialmente | En `checkUploadSize` se verifica el tamaño de archivo, pero no hay mitigaciones efectivas contra bombas XML o YAML que puedan causar DoS. |

## 4. Dependencias y Librerías

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿Las dependencias están actualizadas? | No | En `package.json` se identifican dependencias obsoletas y versiones con vulnerabilidades conocidas, como jsonwebtoken 0.4.0 y express-jwt 0.1.3. |
| ¿Se utilizan librerías de seguridad adecuadas? | No | Las librerías de seguridad como sanitize-html están en versiones desactualizadas (1.4.2) y vulnerables. |
| ¿Existe un proceso para actualizar dependencias? | Sí | Existen configuraciones para dependabot y CI/CD según los archivos de configuración, pero manteniendo intencionalmente versiones vulnerables. |
| ¿Se verifica la integridad de dependencias? | Parcialmente | Hay scripts para generar SBOM (Software Bill of Materials), pero no hay verificación de integridad rigurosa. |
| ¿Hay dependencias innecesarias o riesgosas? | Sí | Hay dependencias obsoletas y vulnerables que aumentan la superficie de ataque, como parte del diseño educativo de la aplicación. |
| ¿Se han implementado contramedidas para ataques de NodeJS específicos? | No | No hay protecciones específicas contra ataques de prototype pollution o vulnerabilidades de dependencias. |
| ¿La aplicación maneja adecuadamente errores sin revelar información sensible? | No | En funciones como `handleXmlUpload`, los mensajes de error incluyen partes del contenido procesado, lo que puede revelar información sensible. |

## 5. Gestión de Configuración y Secretos

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿Hay información sensible hardcodeada en el código? | Sí | En `lib/insecurity.ts` se encuentra hardcodeada la clave privada RSA para JWT y otros secretos como la sal para hashing. |
| ¿Se utilizan variables de entorno para configuraciones sensibles? | Parcialmente | Aunque hay uso de `config` para algunas configuraciones, elementos críticos como claves criptográficas están hardcodeados. |
| ¿Existe separación adecuada entre entornos de desarrollo y producción? | No | No hay mecanismos claros para separar configuraciones entre entornos. |
| ¿Se utiliza HTTPS para todas las comunicaciones? | No | No hay evidencia de redirección forzada a HTTPS ni configuración de HSTS. |
| ¿Se rotan periódicamente los secretos y credenciales? | No | Las claves están hardcodeadas, lo que sugiere que no hay rotación periódica. |

## 6. Registro y Monitoreo

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿Se implementa un sistema adecuado de logging para eventos de seguridad? | Parcialmente | Hay logs de acceso mediante Morgan, pero faltan logs específicos para eventos de seguridad. |
| ¿Los logs contienen información potencialmente sensible? | Sí | No hay sanitización de datos sensibles en los logs, y los logs son accesibles vía `/support/logs`. |
| ¿Existe monitoreo para detección de actividades sospechosas? | Parcialmente | Hay métricas Prometheus, pero sin alertas configuradas para detectar actividad maliciosa. |
| ¿Se almacenan y protegen adecuadamente los logs? | No | Los logs son accesibles públicamente a través de `/support/logs` sin controles de acceso adecuados. |
| ¿Hay mecanismos de alerta para eventos críticos de seguridad? | No | No hay evidencia de sistema de alertas configurado. |

## 7. Protección contra Ataques Comunes Web

| Pregunta | Respuesta | Argumentación Detallada |
|----------|-----------|-------------------------|
| ¿Existe protección efectiva contra CSRF? | No | No se observan tokens CSRF ni otras protecciones.  |
| ¿La aplicación es resistente a ataques de clickjacking? | Parcialmente | Se implementa `helmet.frameguard()`, pero puede estar mal configurado. |
| ¿Se implementan encabezados de seguridad HTTP adecuados? | Parcialmente | Se usa Helmet, pero con opciones limitadas, y el filtro XSS está explícitamente desactivado. |
| ¿Las cookies tienen configuraciones de seguridad apropiadas? | No | No hay evidencia de flags como HttpOnly, Secure o SameSite en las cookies. |
| ¿Se protege adecuadamente contra ataques de enumeración de usuarios? | No | No hay protección evidente contra enumeración de usuarios en endpoints de autenticación. |

## Vulnerabilidades Críticas Identificadas

### 1. Inyección SQL en Autenticación
- **Ubicación**: `routes/login.ts`, línea 27
- **Descripción**: La consulta de autenticación usa interpolación directa de cadenas, permitiendo ataques de inyección SQL.
- **Impacto**: Crítico - Permite bypass de autenticación y potencial acceso a datos sensibles de todos los usuarios.
- **Riesgo**: Alto - Fácilmente explotable con conocimientos básicos de SQL.

### 2. Gestión Insegura de Tokens JWT
- **Ubicación**: `lib/insecurity.ts`, líneas 17-18
- **Descripción**: La clave privada para JWT está hardcodeada en el código fuente.
- **Impacto**: Alto - Compromiso de la clave privada permite forjar tokens y elevar privilegios.
- **Riesgo**: Alto - La clave está disponible en el código fuente para cualquiera que tenga acceso.

### 3. XXE en Procesamiento de Archivos XML
- **Ubicación**: `routes/fileUpload.ts`, función `handleXmlUpload`
- **Descripción**: Se procesan archivos XML con opciones inseguras (`noent: true`), lo que permite ataques XXE.
- **Impacto**: Alto - Puede conducir a divulgación de archivos del sistema y ataques DoS.
- **Riesgo**: Medio - Requiere una carga específicamente creada, pero es relativamente fácil de explotar.

### 4. Hashing MD5 para Contraseñas
- **Ubicación**: `lib/insecurity.ts`, línea 49
- **Descripción**: Se utiliza MD5 para el hash de contraseñas, un algoritmo criptográficamente débil.
- **Impacto**: Alto - Permite ataques rápidos de fuerza bruta contra contraseñas.
- **Riesgo**: Alto - MD5 es conocido por ser vulnerable y existen tablas rainbow.

### 5. Path Traversal en Procesamiento de ZIP
- **Ubicación**: `routes/fileUpload.ts`, función `handleZipFileUpload`
- **Descripción**: Validación insuficiente de las rutas de los archivos extraídos del ZIP.
- **Impacto**: Alto - Permite escritura en ubicaciones no autorizadas del sistema de archivos.
- **Riesgo**: Medio - Requiere preparación de un archivo ZIP malicioso específico.

## Top 5 Recomendaciones de Seguridad

1. **Implementar Consultas Parametrizadas para Prevenir Inyección SQL**
   - **Impacto**: Crítico
   - **Recomendación**: Reemplazar todas las consultas SQL que usan interpolación de cadenas con consultas parametrizadas o usar ORM con vinculación de parámetros.
   - **Evidencia**: En `routes/login.ts`, línea 27, se utiliza interpolación de cadenas directa en la consulta SQL de autenticación.

2. **Mejorar la Gestión de Claves y Secretos**
   - **Impacto**: Alto
   - **Recomendación**: Mover todas las claves privadas y secretos a variables de entorno o gestores de secretos como HashiCorp Vault, eliminar hardcoding de secretos.
   - **Evidencia**: En `lib/insecurity.ts`, línea 17-18, se observa una clave privada RSA hardcodeada para JWT.

3. **Actualizar el Mecanismo de Hash de Contraseñas**
   - **Impacto**: Alto
   - **Recomendación**: Reemplazar MD5 con algoritmos diseñados para contraseñas como Argon2id, bcrypt o PBKDF2 con salting único por usuario.
   - **Evidencia**: En `lib/insecurity.ts`, línea 49, se utiliza MD5 para el hash de contraseñas.

4. **Implementar Validación Estricta en Procesamiento de Archivos**
   - **Impacto**: Alto
   - **Recomendación**: Añadir validación estricta de rutas y contenido para prevenir path traversal y ataques XXE/YAML.
   - **Evidencia**: En `routes/fileUpload.ts`, se observan vulnerabilidades en el procesamiento de archivos ZIP, XML y YAML.

5. **Actualizar Dependencias Críticas de Seguridad**
   - **Impacto**: Alto
   - **Recomendación**: Actualizar versiones de dependencias críticas como express-jwt, jsonwebtoken y sanitize-html a versiones seguras recientes.
   - **Evidencia**: En `package.json`, se observan versiones muy antiguas de dependencias críticas como jsonwebtoken 0.4.0 y express-jwt 0.1.3.

## Conclusión

OWASP Juice Shop es una aplicación intencionalmente vulnerable para propósitos educativos, por lo que muchas de las vulnerabilidades identificadas son deliberadas. Este análisis detallado proporciona una visión completa de las vulnerabilidades presentes, su impacto potencial y recomendaciones para su mitigación. Las recomendaciones proporcionadas serían esenciales para asegurar una aplicación similar en un entorno de producción real.

**Nota importante**: Este análisis se ha realizado reconociendo que OWASP Juice Shop es intencionalmente vulnerable como herramienta educativa. Las vulnerabilidades identificadas son parte de su diseño para servir como plataforma de aprendizaje en seguridad web.