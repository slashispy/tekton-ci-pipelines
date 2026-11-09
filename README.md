# tekton-ci-pipelines

CI **dentro del cluster KIND** del circuito [QuickTask GitOps](https://slashispy.github.io/kind-gitops-utility-belt/),
construido con **Tekton**. Responde a: _¿cómo se ejecutan los pipelines y qué ejecutan?_

> Spec de diseño: `quicktask-tekton-ci`. Estado actual: **Fase B** (plataforma Tekton
> instalada por Argo CD). Tasks, Pipelines, Triggers y SonarQube llegan en las fases siguientes.

## Estructura

```
platform/tekton/     # lo que instala Argo CD (Application `platform-tekton`)
  kustomization.yaml         # releases oficiales con versión fijada
  config-defaults-patch.yaml # timeout por defecto de los runs
  resource-limits.yaml       # límites de memoria/CPU de cada componente
  remove-hpa.yaml            # KIND no tiene metrics-server (HPA quedaría Degraded)
ci/                  # namespace `tekton-ci` (Application `platform-ci`)
  namespace.yaml · limits.yaml · rbac/serviceaccounts.yaml
docs/                # decisiones y políticas
```

## Versiones fijadas

| Componente | Versión | Origen |
|---|---|---|
| Tekton Pipelines | `v1.18.0` | `infra.tekton.dev/tekton-releases/pipeline/previous/v1.18.0/release.yaml` |
| Tekton Triggers (+ interceptors) | `v0.37.2` | `.../triggers/previous/v0.37.2/{release,interceptors}.yaml` |
| Tekton Dashboard | `v0.74.0` | `.../dashboard/previous/v0.74.0/release.yaml` (arranca con `--read-only=true`) |

Para actualizar: cambiar la versión en `platform/tekton/kustomization.yaml`, revisar con
`kubectl kustomize platform/tekton` y dejar que Argo CD sincronice.

## Decisiones

- **Plataforma por GitOps**: Argo CD instala Tekton; lo único imperativo es el bootstrap.
- **Dos Applications con sync waves**: `platform-tekton` (CRDs y controllers, wave 0) antes que
  `platform-ci` (recursos que usan esos CRDs, wave 1).
- **Namespaces separados**: `tekton-pipelines` (plataforma) y `tekton-ci` (ejecuciones).
- **Límites explícitos**: los manifiestos oficiales casi no traen límites (y el de resolvers trae
  4 Gi); se parchean para caber en el presupuesto de 12 GB. Valores iniciales, se miden en la fase G.
- **Mínimo privilegio**: ServiceAccounts separados (`pipeline-build`, `pipeline-pr`, `pipeline-smoke`),
  sin `cluster-admin`; Dashboard en solo lectura y sin Ingress.

## Uso

```bash
# Dashboard (solo lectura)
kubectl -n tekton-pipelines port-forward svc/tekton-dashboard 9097:9097   # http://localhost:9097

# Estado de la plataforma
kubectl -n argocd get applications platform-tekton platform-ci
kubectl -n tekton-pipelines get pods
```

## Parte del circuito QuickTask GitOps

Ver la tabla completa en [`kind-gitops-utility-belt`](https://github.com/slashispy/kind-gitops-utility-belt).
Este repo lo consume [`gitops-argocd-apps`](https://github.com/slashispy/gitops-argocd-apps)
(`apps/platform-tekton.yaml`, `apps/platform-ci.yaml`).
