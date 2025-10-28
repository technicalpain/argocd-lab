# argocd-lab
Este repositorio contiene un laboratorio simple para aprender los fundamentos de Argo CD y GitOps. El objetivo es comprender cómo Argo CD sincroniza el estado del clúster con lo declarado en Git y cómo responde ante cambios, desviaciones (drift) o eliminaciones (prune). También incuye un ejercicio para usar Kustomizr con Argo CD.

## 📘 Requisitos previos

- Kubernetes funcional (minikube, kind, k3d, EKS, etc.).
- kubectl y [argocd cli](https://argo-cd.readthedocs.io/en/stable/cli_installation/) instalados.
- Argo CD  [desplegado](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd) en el namespace argocd.

## 🧪 Ejercicio 1: Manifests-plain 

### 1.1 Despliegue inicial

Crea una aplicación desde la UI de Argo colocando la dirección https de este repo y el path manigests-plan para crear la demo en Argo.

Verifica que los pods se inicien correctamente.

### 1.2  Cambio en Git (GitOps básico)

Establece la politica de sincronización de la aplicación en auto-sync para que argo esté monitoreando los cambios en los manifests del repositorio.

Modifica el número de réplicas o la imagen en deployment.yaml.

Haz commit + push y observa cómo Argo aplica el cambio automáticamente.

### 1.3 Drift + Self-Heal

Escala manualmente el Deployment (kubectl scale ...).

Argo detectará el drift y lo corregirá para coincidir con Git.

### 1.4 Prune

Elimina un manifest del repositorio y haz push.

Argo eliminará el recurso correspondiente del clúster.

## 🧪 Ejercicio 2: Manifests-kustomize
 
 ### 2.1 Despliegue inicial
