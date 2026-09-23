# gitops-manifests — Manifiestos GitOps del checkout (nerdearla-eks)

Repo de manifiestos que **Argo CD** sincroniza contra el cluster. El cluster se
arregla **mergeando un Pull Request** acá, nunca con `kubectl apply` manual.

> En el repo final de la demo esto vive como un repositorio git separado
> (`nerdearla-eks-gitops`). Acá está incluido como subcarpeta para que todo el
> material viaje junto. Al preparar la demo, copiá `gitops-manifests/` a su
> propio repo y apuntá `gitops_repo_url` (Terraform) a esa URL.

## Estructura

```
base/                      # Manifiestos base (namespace, ConfigMap, Deployment, Service)
overlays/demo/             # Overlay que sincroniza Argo CD (ESTADO ROTO)
argocd/application.yaml    # Application de Argo CD (referencia)
```

## Enfoque "sin registry" (autónomo)

Para que la demo no dependa de buildear/pushear imágenes, el `Deployment` corre
`server.js` (montado desde el ConfigMap `checkout-app-src`) usando la imagen
pública `node:20-alpine`. El comportamiento por versión lo da la env
`APP_VERSION`. Si preferís imágenes propias en ECR, usá `scripts/build-push.sh`
y descomentá el bloque `images:` en `overlays/demo/kustomization.yaml`.

## Estado ROTO (punto de partida de la demo)

`overlays/demo/patch-deployment-v2.yaml` deja el checkout en **v2 sin
`PAYMENTS_API_URL`**. Al sincronizar, el proceso termina al arrancar con:

```
FATAL: falta la variable de entorno PAYMENTS_API_URL; el checkout no puede arrancar
```

→ el pod queda en **CrashLoopBackOff**.

## El ARREGLO (PR que se aprueba en vivo)

El PR agrega `PAYMENTS_API_URL`. Hay una rama preparada de respaldo:
**`nerdearla-eks/fix-payments-url`**. El cambio:

1. Agrega el ConfigMap `checkout-config` (ver
   `overlays/demo/configmap-checkout.yaml.fix-example`).
2. Referencia `PAYMENTS_API_URL` en el Deployment vía `envFrom`/`valueFrom`.

Tras el merge, Argo CD (auto-sync + self-heal) reconcilia y el pod vuelve a
**Running** en menos de un minuto.

## Causa alternativa: OOMKilled (para variar entre pasadas)

`overlays/oomkilled/` deja el checkout en v1 (que funciona) pero con un
`resources.limits.memory` absurdamente bajo, forzando **OOMKilled**. Es una
segunda narrativa de incidente sin tocar la variable de entorno. Ver
`overlays/oomkilled/README.md`.
