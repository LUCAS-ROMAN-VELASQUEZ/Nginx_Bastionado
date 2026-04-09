Despliegue y bastionado de un servicio web con Docker (Nginx)
1. Despliegue del servicio

Para realizar la práctica, se ha desplegado un contenedor web utilizando la imagen oficial de Nginx mediante Docker con el siguiente comando:

docker run -d -p 8080:80 --name mi_nginx nginx

Esto levanta un servidor web en segundo plano, exponiendo el puerto 80 del contenedor en el puerto 8080 de la máquina host. Posteriormente, se ha comprobado el acceso al servicio a través del navegador mediante http://localhost:8080, verificando que el servidor responde correctamente.

2. Exposición del servicio

El servicio está expuesto en el puerto 8080 del host y, según la configuración observada, se encuentra asociado a la dirección 0.0.0.0, lo que implica que es accesible desde cualquier interfaz de red.

Esto supone que cualquier equipo con conectividad hacia la máquina podría acceder al servicio, lo cual en un entorno real representa un riesgo si no se aplican controles adicionales. Además, el servicio funciona sobre HTTP, por lo que toda la comunicación se realiza en texto plano.

3. Inspección del contenedor

Se ha utilizado el comando docker inspect para analizar la configuración interna del contenedor, así como docker logs y acceso interactivo con docker exec.

Entre los aspectos más relevantes detectados:

El contenedor se ejecuta como usuario root, lo que incrementa el impacto en caso de compromiso.
El puerto 8080 está expuesto en todas las interfaces (0.0.0.0).
No existe política de reinicio configurada.
No se han definido límites de CPU ni memoria.
El sistema de archivos no es de solo lectura.
Los logs muestran información detallada como la versión de Nginx y del sistema operativo.
Se ejecutan scripts automáticos al arranque del contenedor.
Se ha comprobado acceso directo al contenedor con privilegios elevados.

Todo esto indica que la configuración es la estándar y no está endurecida para un entorno de producción.

4. Riesgos de seguridad

A partir del análisis realizado, se identifican los siguientes riesgos principales:

Exposición pública del servicio: accesible desde cualquier red sin restricciones.
Ejecución como root: aumenta el impacto de una posible vulnerabilidad.
Configuración por defecto: el servidor muestra información como la versión, útil para ataques dirigidos.
Falta de cifrado (HTTPS): la información viaja en texto plano.
Ausencia de control de acceso: cualquier usuario puede acceder al servicio.
Filtración de información en logs: se exponen detalles del sistema y errores internos.

Esto supone un riesgo porque un atacante podría obtener información del sistema, explotar vulnerabilidades conocidas o acceder sin ningún tipo de restricción.

5. Propuesta de bastionado

Para mejorar la seguridad del servicio, se plantean las siguientes medidas:

🔒 Red y puertos
Limitar la exposición del servicio a localhost (127.0.0.1) o a IPs concretas.
Aplicar reglas de firewall para restringir el acceso.
👤 Usuario seguro
Ejecutar el contenedor con un usuario no privilegiado en lugar de root.
⚙️ Configuración de Nginx
Ocultar la versión del servidor (server_tokens off).
Limitar métodos HTTP permitidos.
Gestionar correctamente los errores para evitar filtración de información.
📊 Logging
Centralizar logs y evitar mostrar información sensible.
Implementar monitorización de accesos.
🔐 HTTPS
Implementar cifrado mediante TLS, por ejemplo usando un reverse proxy.
⚙️ Recursos y sistema
Definir límites de CPU y memoria.
Usar sistema de archivos en modo solo lectura.
Configurar política de reinicio automática.
6. Automatización

Para facilitar un despliegue más seguro y repetible, se propone el uso de automatización:

Docker Compose
version: '3'
services:
  web:
    image: nginx
    ports:
      - "127.0.0.1:8080:80"
    restart: always

Esto permite controlar mejor la exposición del servicio y su comportamiento.

Script de comprobación
#!/bin/bash

PORT=8080

if netstat -tuln | grep $PORT; then
  echo "Puerto $PORT abierto ⚠️"
else
  echo "Puerto cerrado ✅"
fi

Este tipo de scripts permite verificar de forma automatizada el estado del servicio y detectar posibles exposiciones.

7. Conclusión

Aunque el despliegue inicial del contenedor es funcional, se ha comprobado que presenta múltiples debilidades de seguridad debido a su configuración por defecto.

En un entorno real, sería necesario aplicar medidas de bastionado y automatización para reducir la superficie de ataque, limitar accesos y asegurar el servicio. Este ejercicio permite entender la importancia de no confiar en configuraciones por defecto y de aplicar buenas prácticas desde el inicio del despliegue.