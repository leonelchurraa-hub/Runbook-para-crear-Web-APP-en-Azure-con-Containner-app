# Runbook técnico: Implementación de arquitectura Azure Container Apps + Application Gateway + ACR
## Versión sanitizada para GitHub

## 1. Objetivo

Implementar una arquitectura web en Azure basada en:

- Docker local para construir la imagen
- Azure Container Registry (ACR) para almacenar la imagen
- Azure Container Apps para ejecutar la aplicación
- Application Gateway v2 como reverse proxy de entrada
- WAF como capa de protección web
- Managed Identity para buenas prácticas de seguridad
- enfoque alineado a Zero Trust

Este runbook está sanitizado y usa ejemplos ficticios para evitar exponer configuraciones reales.

---

## 2. Arquitectura objetivo

```text
Internet
  -> Application Gateway v2 + WAF
  -> Azure Container Apps
  -> Azure Container Registry (ACR)
```

Servicios complementarios recomendados:

- Managed Identity
- Log Analytics
- Azure Monitor
- Key Vault (si luego se usan secretos)

---

## 3. Prerrequisitos

Antes de empezar:

- Tener Docker Desktop instalado y funcionando en Windows
- Tener Azure CLI instalado
- Tener acceso al portal de Azure
- Tener una suscripción activa de Azure
- Tener VS Code para editar el proyecto
- Tener la aplicación local en una carpeta, por ejemplo:

```text
C:\Users\Usuario\Desktop\mi-app-react
```

---

## 4. Preparar la aplicación localmente

Abrir el proyecto en VS Code.

### 4.1 Crear el archivo `Dockerfile`

```dockerfile
FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80
```

### 4.2 Crear el archivo `.dockerignore`

```text
node_modules
dist
.git
.gitignore
npm-debug.log
```

---

## 5. Construir la imagen Docker

Desde la terminal, dentro de la carpeta del proyecto:

```bash
docker build -t mi-app-web .
```

### 5.1 Ejecutar la imagen localmente

```bash
docker run -d -p 8080:80 --name mi-app-web mi-app-web
```

### 5.2 Probar en navegador

```text
http://localhost:8080
```

Si la página abre, la imagen quedó bien construida.

---

## 6. Errores frecuentes en Docker y corrección

### Error: `docker: command not found`

**Causa:** Docker no estaba instalado o no estaba levantado.

**Solución:** instalar Docker Desktop y verificar:

```bash
docker --version
docker ps
```

### Error: typo en `Dockerfile`

Ejemplo incorrecto:

```dockerfile
ROM node:20-alpine AS build
```

Debe ser:

```dockerfile
FROM node:20-alpine AS build
```

### Error: pensar que faltan dependencias

No hace falta instalar dependencias aparte dentro del contenedor si el `Dockerfile` ya tiene:

```dockerfile
RUN npm install
```

---

## 7. Crear Azure Container Registry (ACR)

Desde el portal de Azure:

1. Ir a **Container Registries**
2. Crear un registro nuevo
3. Elegir:
   - Subscription
   - Resource Group
   - Nombre único del ACR
   - Región
   - SKU: **Basic**

### 7.1 Networking del ACR

Si el firewall del ACR bloquea el acceso, permitir la IP pública del operador o habilitar acceso público temporalmente.

### 7.2 Admin user

Para laboratorio se puede habilitar temporalmente.

**Buena práctica:** luego deshabilitarlo y usar RBAC + Managed Identity.

---

## 8. Login en Azure y ACR

```bash
az login
az acr login --name miAcr
```

Para obtener el login server exacto:

```bash
az acr show --name miAcr --query loginServer -o tsv
```

---

## 9. Subir la imagen al ACR

Supongamos que el login server es:

```text
miacr.azurecr.io
```

### 9.1 Etiquetar la imagen

```bash
docker tag mi-app-web:latest miacr.azurecr.io/mi-app-web:v1
```

### 9.2 Subirla

```bash
docker push miacr.azurecr.io/mi-app-web:v1
```

### 9.3 Verificar en Azure

```bash
az acr repository list --name miAcr --output table
az acr repository show-tags --name miAcr --repository mi-app-web --output table
```

---

## 10. Errores frecuentes al subir al ACR y corrección

### Error: `tag does not exist`

**Causa:** se hizo `push` sin hacer antes `docker tag`.

**Solución:** etiquetar primero y después subir.

### Error: `invalid reference format`

**Causa:** variable con caracteres ocultos como `\r`.

**Solución:** limpiar así:

```bash
LOGIN_SERVER=$(az acr show --name miAcr --query loginServer -o tsv | tr -d '\r')
```

### Error: `no such host`

**Causa:** login server mal escrito.

**Solución:** usar el valor exacto de `az acr show`.

### Error: acceso denegado por firewall del ACR

**Causa:** IP pública no permitida.

**Solución:** agregar la IP del operador o abrir acceso temporalmente.

---

## 11. Crear Azure Container App

Desde el portal:

1. Ir a **Container Apps**
2. Crear nueva app
3. Configurar:
   - Resource Group
   - Nombre de la app
   - Región
   - Environment nuevo o existente

### 11.1 Imagen

- Image source: **Azure Container Registry**
- Registry: `miAcr`
- Image: `mi-app-web`
- Tag: `v1`

### 11.2 Ingress

- Ingress: **Enabled**
- Traffic: **Accepting traffic from anywhere**
- Transport: **Auto / HTTP**
- Target port: **80**

---

## 12. Error frecuente en Container Apps

### Error: la página no abre

**Causa:** tráfico entrante no habilitado.

**Solución:** habilitar ingress y permitir tráfico externo.

---

## 13. Managed Identity en Container Apps

### 13.1 Activar identidad administrada

En la Container App:

- **Identity**
- **System assigned** = On

### 13.2 Dar permisos de pull al ACR

En el ACR:

- **IAM / Access control**
- agregar rol **AcrPull**
- asignarlo a la identidad de la Container App

### 13.3 Buena práctica

- operador humano: **AcrPush**
- runtime de la app: **AcrPull**

---

## 14. Crear Application Gateway

Como capa frontal regional más segura que exponer la Container App directamente.

### 14.1 Tipo recomendado

- **Application Gateway v2**
- **WAF habilitable**

### 14.2 No usar

- **Application Gateway for Containers**

porque está orientado a AKS/Kubernetes, no a Container Apps.

---

## 15. Configurar el backend hacia Container Apps

### 15.1 Backend pool

Tipo de destino:

- **Dirección IP o FQDN**

Usar el FQDN de la Container App, por ejemplo:

```text
miapp.entorno.region.azurecontainerapps.io
```

### 15.2 Listener

Para pruebas:

- protocolo: **HTTP**
- puerto: **80**
- IP pública del Application Gateway

### 15.3 Backend setting

Crear configuración así:

- protocolo: **HTTP**
- puerto: **80**
- reemplazar host por nombre específico
- host = FQDN de la Container App

Ejemplo:

```text
miapp.entorno.region.azurecontainerapps.io
```

---

## 16. Error frecuente en Application Gateway: 502 Bad Gateway

### Síntoma

Al abrir la IP pública del Application Gateway aparece:

```text
502 Bad Gateway
```

### Revisión recomendada

Ir a:

- **Backend health**

### Caso real frecuente

El backend responde, pero con:

```text
404
```

Eso suele significar que el **host header** está mal.

### Solución

En **Backend settings**:

- habilitar reemplazo de nombre de host
- usar el FQDN exacto de la Container App

Cuando eso se corrige, el backend pasa a saludable y la página carga correctamente.

---

## 17. Validación funcional final

### 17.1 Probar Container App directo

Abrir la URL pública de la Container App.

### 17.2 Probar Application Gateway

Abrir en navegador:

```text
http://IP_PUBLICA_DEL_APP_GATEWAY
```

Si la página carga, la arquitectura quedó operativa.

---

## 18. Activar WAF

Una vez que la arquitectura ya funciona:

1. Ir al Application Gateway
2. Activar **WAF**
3. Empezar en modo:
   - **Detection**
4. Más adelante pasar a:
   - **Prevention**

---

## 19. Dominio y HTTPS

Cuando ya exista un dominio:

### 19.1 Dominio

En el proveedor DNS:

- crear registro apuntando a la IP pública del Application Gateway

### 19.2 HTTPS

Configurar en Application Gateway:

- listener HTTPS 443
- cargar certificado PFX del dominio
- mantener backend hacia Container App por HTTP 80

### 19.3 Redirección

Crear redirección:

- HTTP 80 -> HTTPS 443

### Nota importante

No vale la pena poner HTTPS solo por IP pública si no hay dominio, porque el certificado no va a coincidir con la IP y el navegador mostrará advertencia.

---

## 20. Administración segura desde VS Code

### Flujo recomendado

```bash
az login
az acr login --name miAcr
docker build -t mi-app-web .
docker tag mi-app-web:latest miacr.azurecr.io/mi-app-web:v2
docker push miacr.azurecr.io/mi-app-web:v2
```

Después, actualizar la Container App al nuevo tag.

### Buenas prácticas

- usar Azure CLI con identidad de Entra
- no guardar contraseñas del ACR en el proyecto
- deshabilitar `admin user` del ACR una vez estabilizado el entorno

---

## 21. Costos: consideraciones

Aunque la Container App pueda escalar a cero, los recursos como:

- Application Gateway
- ACR

pueden seguir generando costo mientras existan.

### Recomendación de laboratorio

Si se quiere minimizar costo:

- borrar Application Gateway al terminar
- dejar Container App en mínimo o eliminarla
- conservar ACR solo si hace falta guardar la imagen

---

## 22. Resumen del enfoque Zero Trust aplicado

### Identidad

- Managed Identity en la Container App
- RBAC mínimo necesario
- sin credenciales hardcodeadas

### Red

- Application Gateway como única entrada pública
- Container App detrás del gateway
- posibilidad de agregar IP restrictions

### Aplicación

- imagen inmutable en ACR
- versionado por tags
- despliegue controlado

### Seguridad adicional recomendada

- WAF en Application Gateway
- Key Vault para secretos
- Log Analytics y Monitor para observabilidad

---

## 23. Checklist final

- [ ] Proyecto probado localmente en Docker
- [ ] Imagen subida a ACR
- [ ] Container App creada
- [ ] Ingress habilitado correctamente
- [ ] Managed Identity activa
- [ ] Rol AcrPull asignado al runtime
- [ ] Application Gateway creado
- [ ] Backend pool configurado con FQDN
- [ ] Backend setting con host correcto
- [ ] Aplicación accesible por IP pública del App Gateway
- [ ] WAF activado en modo Detection
- [ ] Dominio pendiente o configurado
- [ ] HTTPS pendiente o configurado con certificado válido

---

## 24. Resultado esperado

Al finalizar este runbook, se obtiene una arquitectura funcional y endurecida basada en:

- Docker local
- Azure Container Registry
- Azure Container Apps
- Application Gateway
- WAF
- Managed Identity

lista para evolucionar posteriormente hacia una implementación todavía más cerrada con dominio, HTTPS, restricciones de acceso y servicios adicionales de seguridad.
