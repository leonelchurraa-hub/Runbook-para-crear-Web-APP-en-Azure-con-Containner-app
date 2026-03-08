# Runbook: Azure Container Apps + Application Gateway + ACR
## Implementación segura con enfoque Zero Trust

Este repositorio contiene un **runbook técnico sanitizado** que documenta, de punta a punta, el proceso de implementación de una arquitectura en Azure basada en:

- **Azure Container Registry (ACR)**
- **Azure Container Apps**
- **Azure Application Gateway**
- **Buenas prácticas de seguridad**
- **Enfoque Zero Trust**

El objetivo de este documento es servir como guía práctica para desplegar una aplicación web containerizada en Azure, comenzando desde la construcción local de la imagen Docker hasta su publicación segura detrás de un Application Gateway, incluyendo lineamientos para dominio propio, certificados HTTPS y endurecimiento de la arquitectura.

---

## Qué incluye este runbook

El documento describe:

- preparación del proyecto local
- creación del contenedor Docker
- construcción y prueba de la imagen local
- publicación de la imagen en Azure Container Registry
- despliegue de la imagen en Azure Container Apps
- configuración de ingress
- implementación de Application Gateway como reverse proxy
- corrección de errores frecuentes de backend, probes y host headers
- activación de identidad administrada (**Managed Identity**)
- lineamientos de seguridad y operación
- recomendaciones para incorporar dominio propio y certificados TLS/HTTPS
- lecciones aprendidas y troubleshooting basado en errores reales

---

## Enfoque de seguridad

La guía está orientada a una implementación con principios de **Zero Trust**, contemplando:

- separación entre plano de administración y plano de ejecución
- uso de **Managed Identity**
- uso de **RBAC** en Azure
- desacople entre runtime y registro de imágenes
- Application Gateway como punto de entrada controlado
- minimización de exposición pública innecesaria
- buenas prácticas para proteger configuración, conectividad y despliegue

---

## Público objetivo

Este runbook está pensado para:

- administradores de infraestructura
- analistas o arquitectos cloud
- perfiles DevOps / SecOps
- personas que necesiten desplegar aplicaciones containerizadas en Azure de forma segura
- equipos que quieran documentar un laboratorio o PoC con criterios cercanos a producción

---

## Contenido disponible

- `RUNBOOK_Arquitectura_ContainerApps_AppGateway_ZeroTrust_Sanitizado_v2.md`
- documentación sanitizada para uso técnico y compartible
- ejemplos sin exponer configuraciones sensibles reales

---

## Importante

Este material está **sanitizado**.  
No incluye:

- credenciales
- secretos
- nombres reales de recursos productivos
- FQDN sensibles
- IPs públicas reales
- identificadores internos de suscripciones o tenants

---

## Uso recomendado

Se recomienda utilizar este repositorio como:

- guía de implementación
- base documental para PoC o laboratorio
- referencia para futuras automatizaciones
- material de soporte para documentación interna o transferencia de conocimiento

---

## Próximos pasos sugeridos

Después de implementar esta arquitectura, se recomienda continuar con:

- dominio propio
- listener HTTPS en Application Gateway
- certificado TLS válido
- redirección HTTP → HTTPS
- endurecimiento de reglas WAF
- restricciones adicionales de red
- observabilidad y monitoreo centralizado

---
