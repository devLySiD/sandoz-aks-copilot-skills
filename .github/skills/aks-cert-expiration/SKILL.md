---
name: aks-cert-expiration-checker
description: Checks and reports the expiration dates of TLS certificates stored in Kubernetes secrets within an AKS cluster.
---

# Verificador de Vencimiento de Certificados en AKS

Cuando un desarrollador pregunte sobre las fechas de vencimiento de los certificados almacenados en secretos de Kubernetes, ejecuta el siguiente flujo de trabajo de manera sistemática utilizando las herramientas de terminal o `kubectl` disponibles.

## Flujo de Ejecución

1. **Identificar Namespaces y Secretos Objetivo**
   - Lista los secretos en todos los namespaces o pregúntale al usuario si desea enfocar la búsqueda en un namespace en particular.
   - Filtra los secretos de tipo `kubernetes.io/tls` o aquellos que contengan claves estándar de certificados (`tls.crt`, `ca.crt`, `cert`).

2. **Extraer e Inspeccionar los Certificados**
   - Para cada secreto coincidente, extrae y decodifica el certificado usando `kubectl`:
     ```bash
     kubectl get secret <nombre-secreto> -n <namespace> -o jsonpath="{.data['tls\.crt']}" | base64 --decode
     ```
   - Si la clave tiene un nombre diferente (por ejemplo, `ca.crt`), adapta la ruta en el `jsonpath` correspondientemente.

3. **Verificar la Fecha de Expiración mediante OpenSSL**
   - Envía el certificado decodificado a `openssl` para consultar su período de validez:
     ```bash
     ... | openssl x509 -enddate -noout
     ```
   - Opcionalmente, extrae también el emisor y el nombre común (CN) para tener un mejor contexto:
     ```bash
     ... | openssl x509 -noout -subject -dates
     ```

4. **Formatear la Respuesta**
   - Presenta los hallazgos en una tabla en formato Markdown limpia que incluya:
     - Namespace
     - Nombre del Secreto
     - Nombre Común (CN) / Sujeto
     - Fecha de Expiración
     - Estado (Activo vs. Por vencer / Vencido)