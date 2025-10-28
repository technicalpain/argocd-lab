# 🧭 Argo CD Lab

Este repositorio contiene un laboratorio simple para aprender los fundamentos de **Argo CD** y **GitOps**.  
El objetivo es comprender cómo Argo CD sincroniza el estado del clúster con lo declarado en Git y cómo responde ante cambios, desviaciones (*drift*) o eliminaciones (*prune*).  

También incluye un segundo ejercicio para usar **Kustomize** con Argo CD, mostrando cómo gestionar múltiples entornos (por ejemplo *dev* y *prod*) a partir de una sola base de manifests.

---

## 📘 Requisitos previos

- Un clúster de **Kubernetes** funcional (`minikube`, `kind`, `k3d`, `EKS`, etc.).
- **kubectl** y [**argocd CLI**](https://argo-cd.readthedocs.io/en/stable/cli_installation/) instalados.
- **Argo CD** [desplegado](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd) en el namespace `argocd`.

---

## 🧪 Ejercicio 1: `manifests-plain`

### 1.1 Despliegue inicial

Crea una aplicación desde la **UI de Argo CD** con los siguientes valores:

- **Repository URL:** `https://github.com/galvarado/argocd-lab.git`
- **Revision:** `main`
- **Path:** `manifests-plain`
- **Namespace:** `demo` (marca *Create Namespace*)
- **Sync Policy:** activa *Auto-Sync*, *Self-Heal* y *Prune*

Una vez creada, Argo desplegará automáticamente el Deployment y el Service de ejemplo (`nginxdemos/hello`).

Verifica el despliegue:

```bash
kubectl get all -n demo
```

Deberías ver un `Deployment`, `Pod` y `Service` activos.

---

### 1.2 Cambio en Git (GitOps básico)

Con *Auto-Sync* activo, Argo CD monitorea los cambios en Git.

1. Edita `manifests-plain/deployment.yaml`
2. Cambia:
   ```yaml
   replicas: 1
   ```
   por:
   ```yaml
   replicas: 3
   ```
3. Haz commit y push:
   ```bash
   git add .
   git commit -m "Aumentar replicas a 3"
   git push
   ```

Argo CD detectará el cambio y escalará automáticamente el deployment.  
Verifica con:
```bash
kubectl get pods -n demo
```

---

### 1.3 Drift + Self-Heal

Escala manualmente desde `kubectl`:

```bash
kubectl scale deploy hello -n demo --replicas=7
```

Argo CD detectará el *drift* (desviación) y lo corregirá automáticamente, devolviendo el estado a lo declarado en Git.

---

### 1.4 Prune

Elimina un manifest del repositorio, por ejemplo:

```bash
git rm manifests-plain/service.yaml
git commit -m "Eliminar service para probar prune"
git push
```

Argo CD eliminará el recurso correspondiente del clúster (`Service` en este caso).  
Luego puedes restaurarlo:

```bash
git checkout main -- manifests-plain/service.yaml
git commit -am "Restaurar service"
git push
```

---

## 🧩 Ejercicio 2: `manifests-kustomize`

En este ejercicio se introduce **Kustomize**, una herramienta nativa de Kubernetes que permite definir una **base común** y **overlays** específicos para cada entorno (por ejemplo `dev` y `prod`).

### Estructura del directorio

```
manifests-kustomize/
├─ base/
│  ├─ deployment.yaml
│  ├─ service.yaml
│  └─ kustomization.yaml
└─ overlays/
   ├─ dev/
   │  ├─ kustomization.yaml
   │  ├─ namespace.yaml
   │  └─ patch-replicas.yaml
   └─ prod/
      ├─ kustomization.yaml
      ├─ namespace.yaml
      └─ patch-replicas.yaml
```

---

### 2.1 Despliegue inicial (con Argo CD)

Crea **dos aplicaciones** nuevas en Argo CD:

#### 🔹 Dev
- Repository URL: `https://github.com/galvarado/argocd-lab.git`
- Revision: `main`
- Path: `manifests-kustomize/overlays/dev`
- Namespace: `demo-dev`
- Auto-Sync, Self-Heal y Prune: ✅

#### 🔹 Prod
- Repository URL: `https://github.com/galvarado/argocd-lab.git`
- Revision: `main`
- Path: `manifests-kustomize/overlays/prod`
- Namespace: `demo-prod`
- Auto-Sync, Self-Heal y Prune: ✅

Cada aplicación desplegará su propio entorno (dev o prod) usando los mismos manifests base.

Verifica:

```bash
kubectl get all -n demo-dev
kubectl get all -n demo-prod
```

---

### 2.2 Personalizaciones por entorno

| Entorno | Réplicas | Imagen | Namespace | Sufijo |
|----------|-----------|---------|------------|---------|
| **Dev**  | 2         | `nginxdemos/hello:latest` | `demo-dev` | `-dev` |
| **Prod** | 5         | `nginxdemos/hello:plain-text` | `demo-prod` | `-prod` |

Kustomize aplica estos cambios automáticamente mediante los archivos `patch-replicas.yaml` y las reglas en `kustomization.yaml`.

---

### 2.3 Pruebas recomendadas

**1️⃣ Escalado:**  
Cambia `replicas` en `overlays/dev/patch-replicas.yaml` → commit + push.  
Argo CD aplicará el cambio solo al entorno `dev`.

**2️⃣ Cambio de imagen:**  
Edita `images.newTag` en `overlays/dev/kustomization.yaml` → commit + push.  
Verás el rollout solo en `demo-dev`.

**3️⃣ Self-Heal:**  
Escala manualmente en el cluster:

```bash
kubectl scale deploy hello-dev -n demo-dev --replicas=9
```

Argo lo revertirá automáticamente al valor en Git.

**4️⃣ Prune:**  
Borra `service.yaml` del `base/` → commit + push.  
Argo eliminará el Service en ambos entornos.

---

## 🚀 Conclusión

- Los manifests de **plain** representan el estado final de una app (deploy directo).
- Los manifests de **kustomize/base** son una **plantilla común** reutilizable.
- Los **overlays** definen lo que cambia según el entorno.
- **Argo CD** entiende Kustomize nativamente, por lo que no necesitas pasos adicionales para renderizarlo.

---

## 📚 Recursos adicionales

- [Documentación oficial de Argo CD](https://argo-cd.readthedocs.io)
- [Guía de Kustomize](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/)
- [Conceptos de GitOps por WeaveWorks](https://www.weave.works/technologies/gitops/)