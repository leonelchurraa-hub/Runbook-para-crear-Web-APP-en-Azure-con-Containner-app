# Runbook: Publicación de PDF sanitizado en GitHub

## 1. Objetivo

Publicar en GitHub una versión **sanitizada** de la guía técnica de implementación de arquitectura en Azure, asegurando que:

- no se expongan datos sensibles
- no se publiquen nombres reales de recursos críticos
- no se revelen dominios, IPs, tenant IDs, subscription IDs, registry names reales ni credenciales
- el documento quede disponible como referencia técnica reutilizable

---

## 2. Alcance

Este procedimiento cubre:

- validación del PDF sanitizado
- creación o uso de un repositorio en GitHub
- carga del documento
- organización recomendada del repositorio
- verificación posterior a la publicación

No cubre:

- publicación del PDF con datos reales
- exposición de configuraciones sensibles
- publicación de credenciales, secretos o access keys

---

## 3. Prerrequisitos

Antes de comenzar, verificar:

- tener cuenta activa en GitHub
- tener acceso al repositorio destino
- disponer del archivo PDF sanitizado final
- confirmar que el PDF no contiene:
  - nombres reales de recursos
  - FQDN reales
  - IPs públicas reales
  - credenciales
  - nombres reales de ACR
  - nombres reales de Container Apps
  - nombres reales de Application Gateway
  - identificadores de suscripción o tenant
  - capturas con información sensible

---

## 4. Nombre recomendado del archivo

Usar un nombre claro y profesional, por ejemplo:

`guia_containerapps_appgw_zero_trust_sanitizada.pdf`

Opcionalmente, usar versionado:

`guia_containerapps_appgw_zero_trust_sanitizada_v1.pdf`

---

## 5. Estructura recomendada del repositorio

Se recomienda guardar el PDF en una estructura como esta:

```text
docs/
  arquitectura/
    guia_containerapps_appgw_zero_trust_sanitizada.pdf
README.md
```

Si el repo es exclusivamente documental, también puede usarse:

```text
/
  guia_containerapps_appgw_zero_trust_sanitizada.pdf
  README.md
```

---

## 6. Validación previa de seguridad

Antes de subir el archivo, revisar nuevamente:

### 6.1 Datos que deben estar sanitizados

- nombres de recursos reales
- login server reales
- FQDN reales
- IP públicas reales
- nombres de app reales
- nombres de registry reales
- nombres de listeners, probes o backend settings reales si se consideran sensibles
- credenciales o tokens
- capturas con información privada

### 6.2 Datos permitidos

- nombres genéricos
- ejemplos ficticios
- arquitectura lógica
- flujo técnico
- buenas prácticas
- pasos operativos
- comandos con placeholders

---

## 7. Publicación desde la interfaz web de GitHub

### 7.1 Ingresar al repositorio

1. Abrir GitHub
2. Entrar al repositorio donde se publicará el documento

### 7.2 Crear carpeta de documentación si no existe

1. Hacer clic en **Add file**
2. Elegir **Create new file**
3. Crear una ruta como:

`docs/arquitectura/README.md`

4. Guardar el archivo

### 7.3 Subir el PDF

1. Hacer clic en **Add file**
2. Elegir **Upload files**
3. Arrastrar o seleccionar el archivo:

`guia_containerapps_appgw_zero_trust_sanitizada.pdf`

4. Ubicarlo dentro de la carpeta recomendada:

`docs/arquitectura/`

5. Escribir un mensaje de commit, por ejemplo:

`Agrega guía sanitizada de arquitectura Azure Container Apps + Application Gateway`

6. Confirmar con **Commit changes**

---

## 8. Publicación por Git local

Si se prefiere subirlo desde Git local:

### 8.1 Clonar el repositorio

```bash
git clone https://github.com/TU-USUARIO/TU-REPO.git
cd TU-REPO
```

### 8.2 Crear estructura de carpetas

```bash
mkdir -p docs/arquitectura
```

### 8.3 Copiar el PDF

Copiar el archivo dentro de:

`docs/arquitectura/`

### 8.4 Agregar y commitear

```bash
git add docs/arquitectura/guia_containerapps_appgw_zero_trust_sanitizada.pdf
git commit -m "Agrega guía sanitizada de arquitectura Azure"
```

### 8.5 Subir a GitHub

```bash
git push origin main
```

---

## 9. README recomendado

Se recomienda agregar en el `README.md` una referencia al documento:

```md
## Documentación

- [Guía sanitizada de arquitectura Azure Container Apps + Application Gateway](docs/arquitectura/guia_containerapps_appgw_zero_trust_sanitizada.pdf)
```

---

## 10. Buenas prácticas para publicación

### 10.1 Publicar solo la versión sanitizada

No subir nunca al mismo repo:

- PDF real
- DOCX real
- diagramas con nombres reales
- capturas del portal con datos productivos

### 10.2 Separar repositorios

Si existe documentación interna y pública:

- usar un repo privado para documentación real
- usar un repo público solo para documentación sanitizada

### 10.3 Versionado

Si el documento cambia, usar:

- `v1`
- `v2`
- `v3`

Ejemplo:

`guia_containerapps_appgw_zero_trust_sanitizada_v2.pdf`

### 10.4 Mantener consistencia

Usar siempre placeholders como:

- `mi-acr.azurecr.io`
- `mi-containerapp.region.azurecontainerapps.io`
- `miappgateway`
- `midominio.com`

---

## 11. Verificación posterior

Después de subir el archivo, verificar:

- que el PDF abra correctamente desde GitHub
- que el nombre sea claro
- que el enlace en README funcione
- que no haya quedado información sensible
- que el commit no incluya archivos no deseados

---

## 12. Checklist final

Antes de cerrar, confirmar:

- [ ] El archivo subido es el PDF sanitizado
- [ ] No contiene credenciales ni secretos
- [ ] No contiene nombres reales de recursos
- [ ] Está ubicado en una carpeta ordenada
- [ ] El README lo referencia correctamente
- [ ] El repo no contiene versiones reales del documento

---

## 13. Convención sugerida para repositorio técnico

Si el objetivo es usar GitHub como repositorio profesional de documentación, una buena estructura sería:

```text
docs/
  arquitectura/
  runbooks/
  diagramas/
README.md
```

Ejemplo:

- `docs/arquitectura/guia_containerapps_appgw_zero_trust_sanitizada.pdf`
- `docs/runbooks/RUNBOOK_Subida_PDF_Sanitizado_GitHub.md`

---

## 14. Resultado esperado

Al finalizar este runbook, el repositorio deberá contener:

- el PDF sanitizado publicado
- una estructura ordenada
- una referencia en README
- cero exposición de datos sensibles

---

## 15. Nota de seguridad

Nunca subir a GitHub público:

- access keys
- contraseñas
- connection strings
- tenant IDs sensibles
- subscription IDs
- nombres reales de componentes de producción
- dominios internos
- capturas con datos operativos reales
