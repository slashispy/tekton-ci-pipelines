# context.md — tekton-ci-pipelines

Contexto para quien (persona o agente) retome este repo.

- **Propósito**: manifiestos de la plataforma de CI (Tekton) y, en fases siguientes, los Tasks,
  Pipelines, Triggers y poller del circuito QuickTask GitOps.
- **Spec**: `quicktask-tekton-ci` (requirements/design/tasks). Fase actual: B.
- **Quién lo consume**: Argo CD, vía `gitops-argocd-apps` (`platform-tekton` → `platform/tekton`,
  `platform-ci` → `ci`). No se aplica a mano.
- **Reglas**: versiones de Tekton fijadas; ningún secreto en Git (los crea
  `kind-gitops-utility-belt/scripts/05-create-ci-secrets.sh`); un PipelineRun a la vez.
- **Validar cambios**: `kubectl kustomize platform/tekton` y `kubectl kustomize ci` deben renderizar sin errores.
