# overlays/oomkilled — Causa alternativa: OOMKilled

Segunda narrativa de incidente para variar entre pasadas de la demo.

- La app está en **v1** (funciona), así que la causa NO es la variable de entorno.
- El `resources.limits.memory` está en **8Mi**, insuficiente para el runtime de
  Node → el kernel mata el contenedor: **OOMKilled** → CrashLoopBackOff.

## Recorrido del agente en este caso

1. `kubectl describe pod` → `Last State: Terminated / Reason: OOMKilled`.
2. `kubectl logs` (si alcanzó a loguear) o `kubectl top pod` (metrics-server).
3. `git diff` → anoche bajaron `limits.memory` a 8Mi.
4. Causa: límite de memoria demasiado bajo.
5. Arreglo (PR): subir `limits.memory` a 64Mi.

## Cómo usarla

Apuntá la Application de Argo CD a `overlays/oomkilled` en lugar de
`overlays/demo` (variable Terraform `gitops_path`, o editá la Application), o
usá `scripts/break.sh --oom`.
