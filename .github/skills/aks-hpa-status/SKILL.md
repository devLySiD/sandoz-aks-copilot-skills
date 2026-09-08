---
name: aks-hpa-status-checker
description: Checks and reports the Horizontal Pod Autoscaler (HPA) status, min/max replica limits, and current replica counts for microservices in an AKS cluster.
---

# Verificador de Estado de HPA en AKS

Cuando un desarrollador pregunte por el HPA (Horizontal Pod Autoscaler) o por las réplicas de sus microservicios, ejecuta el siguiente flujo de trabajo sistemáticamente utilizando las herramientas de terminal o `kubectl` disponibles.

## Flujo de Ejecución

1. **Identificar Namespaces y Recursos HPA**
   - Lista los recursos de tipo `hpa` en el clúster (puedes consultar en todos los namespaces o preguntar al usuario si prefiere uno específico):
     ```bash
     kubectl get hpa -A
     ```
   - Si se requiere mayor detalle sobre los deployments asociados, cruza la información obtenida.

2. **Extraer la Información Clave del HPA**
   - Para cada HPA encontrado, extrae los siguientes datos utilizando `jsonpath` o una salida estructurada:
     - Nombre del HPA y Namespace
     - Deployment u objeto objetivo (*Target*)
     - Réplicas Mínimas (`minReplicas`)
     - Réplicas Máximas (`maxReplicas`)
     - Réplicas Actuales (`currentReplicas`)
     - Réplicas Deseadas / En ejecución

     Ejemplo con comandos de extracción:
     ```bash
     kubectl get hpa -n <namespace> -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,TARGET:.spec.scaleTargetRef.name,MIN:.spec.minReplicas,MAX:.spec.maxReplicas,CURRENT:.status.currentReplicas,DESIRED:.status.desiredReplicas
     ```

3. **Formatear la Respuesta**
   - Presenta los resultados al desarrollador en una tabla en formato Markdown clara y ordenada que incluya:
     - Namespace
     - Microservicio / Target
     - Mín. Réplicas
     - Máx. Réplicas
     - Réplicas Actuales
     - Estado general (por ejemplo, si está escalado al mínimo, al máximo, o si hay alguna alerta de métricas).