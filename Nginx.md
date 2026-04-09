# 🐳 Despliegue y Bastionado de un Servicio Web con Docker (Nginx)

---

## 1. Despliegue del Servicio

Para realizar la práctica, se desplegó un contenedor web utilizando la imagen oficial de **Nginx** mediante Docker:

```bash
docker run -d -p 8080:80 --name mi_nginx nginx
```

Este comando levanta un servidor web en segundo plano, exponiendo el **puerto 80** del contenedor en el **puerto 8080** del host. El acceso se verificó a través del navegador en:

```
http://localhost:8080
```

---

## 2. Exposición del Servicio

El servicio queda expuesto en el puerto `8080` del host, asociado a la dirección `0.0.0.0`, lo que implica que es **accesible desde cualquier interfaz de red**.

> ⚠️ **Riesgo:** Cualquier equipo con conectividad hacia la máquina podría acceder al servicio sin ninguna restricción. Además, al funcionar sobre **HTTP**, toda la comunicación viaja en **texto plano**.

---

## 3. Inspección del Contenedor

Se utilizaron los siguientes comandos para analizar la configuración interna:

```bash
docker inspect mi_nginx
docker logs mi_nginx
docker exec -it mi_nginx bash
```

### Hallazgos principales

| Aspecto | Estado |
|---|---|
| Usuario de ejecución | `root` ⚠️ |
| Puerto expuesto | `0.0.0.0:8080` (todas las interfaces) ⚠️ |
| Política de reinicio | No configurada ⚠️ |
| Límites de CPU/Memoria | No definidos ⚠️ |
| Sistema de archivos | Lectura y escritura (no read-only) ⚠️ |
| Información en logs | Versión de Nginx y SO expuestas ⚠️ |
| Scripts de arranque | Se ejecutan automáticamente ⚠️ |
| Acceso al contenedor | Con privilegios elevados ⚠️ |

> La configuración es completamente estándar y **no está endurecida** para un entorno de producción.

---

## 4. Riesgos de Seguridad

A partir del análisis realizado, se identifican los siguientes riesgos:

### 🌐 Exposición pública
El servicio es accesible desde cualquier red sin restricciones de IP ni firewall.

### 👑 Ejecución como root
Aumenta considerablemente el impacto de una posible vulnerabilidad o compromiso del contenedor.

### 🔍 Configuración por defecto
El servidor expone su versión y detalles del sistema, facilitando ataques dirigidos.

### 🔓 Sin cifrado (HTTP)
La información viaja en texto plano, susceptible de interceptación.

### 🚪 Sin control de acceso
Cualquier usuario puede acceder al servicio sin autenticación ni restricción alguna.

### 📋 Filtración en logs
Los logs exponen detalles internos del sistema y errores que podrían aprovecharse.

---

## 5. Propuesta de Bastionado

### 🔒 Red y Puertos

- Limitar la exposición del servicio a `127.0.0.1` o a IPs concretas.
- Aplicar reglas de **firewall** para restringir el acceso externo.

### 👤 Usuario Seguro

- Ejecutar el contenedor con un **usuario no privilegiado** en lugar de `root`.

### ⚙️ Configuración de Nginx

- Ocultar la versión del servidor:
  ```nginx
  server_tokens off;
  ```
- Limitar los métodos HTTP permitidos.
- Gestionar correctamente los errores para evitar filtración de información sensible.

### 📊 Logging

- Centralizar logs y evitar mostrar información sensible.
- Implementar **monitorización de accesos** para detectar comportamientos anómalos.

### 🔐 HTTPS / TLS

- Implementar cifrado mediante **TLS**, utilizando por ejemplo un reverse proxy (Traefik, Caddy, etc.).

### 🖥️ Recursos y Sistema

- Definir **límites de CPU y memoria** para evitar DoS.
- Usar el sistema de archivos en **modo solo lectura**.
- Configurar una política de **reinicio automático**.

---

## 6. Automatización

### 🐙 Docker Compose

Para un despliegue más seguro y repetible:

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "127.0.0.1:8080:80"
    restart: always
    read_only: true
    mem_limit: 256m
    cpus: "0.5"
```

Esto permite controlar la exposición del servicio, su comportamiento ante fallos y sus recursos disponibles.

### 📜 Script de Comprobación

Para verificar de forma automatizada el estado del puerto:

```bash
#!/bin/bash

PORT=8080

if netstat -tuln | grep -q $PORT; then
  echo "⚠️  Puerto $PORT abierto — revisar exposición"
else
  echo "✅ Puerto $PORT cerrado"
fi
```

Este tipo de scripts permite detectar posibles exposiciones de forma rápida y automatizada.

---

## 7. Conclusión

Aunque el despliegue inicial del contenedor es funcional, la configuración por defecto presenta **múltiples debilidades de seguridad** que lo hacen inadecuado para un entorno de producción.

En un contexto real, sería imprescindible aplicar medidas de **bastionado y automatización** para:

- Reducir la superficie de ataque.
- Limitar los accesos no autorizados.
- Proteger la confidencialidad e integridad del servicio.

> 💡 Este ejercicio pone de manifiesto la importancia de **no confiar en configuraciones por defecto** y de aplicar buenas prácticas de seguridad desde el inicio del despliegue, siguiendo el principio de *secure by design*.

---

*Práctica realizada con Docker + Nginx · Seguridad en Sistemas y Redes*