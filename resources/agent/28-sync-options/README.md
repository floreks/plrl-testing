# Sync options

This scenario checks sync option behavior during manifest removal and ServiceDeployment deletion.

## Prepare a disposable source

Use a disposable Git fork or branch. The parent entrypoint reads `services/agent` from `main` by default, and this scenario's `spec.git.ref` also defaults to `main`. Before reconciling, point the parent entrypoint's repository URL and Git ref at the disposable fork and branch. Set this scenario's `repositoryRef` to the repository resource for that fork and its `spec.git.ref` to the same branch.

Commit the ServiceDeployment and all fixture files. Wait for `agent-28-testing-sync-options` to reconcile. Confirm the 12 ConfigMaps exist and appear in the ServiceDeployment inventory:

```sh
kubectl get configmaps -n testing -l scenario=sync-options
kubectl get servicedeployments agent-28-testing-sync-options -n testing \
  -o jsonpath='{range .status.components[*]}{.kind}/{.name}{"\n"}{end}'
```

## Stage B: remove fixture manifests

In the disposable branch, remove the listed files from `resources/agent/28-sync-options`, commit, and wait for the scenario service to reconcile. The inventory is `ServiceDeployment.status.components`; check it alongside live ConfigMaps.

| Remove this file | Expected result after reconcile |
| --- | --- |
| `default-prune.yaml.liquid` | `sync-options-default-prune` is deleted and absent from the inventory. |
| `plural-prune-false.yaml.liquid` | `sync-options-plural-prune-false` stays live and remains in the inventory. Stage C should delete it. |
| `argocd-prune-false.yaml.liquid` | `sync-options-argocd-prune-false` stays live and remains in the inventory. Stage C should delete it. |
| `plural-prune-delete-false.yaml.liquid` | `sync-options-plural-prune-delete-false` stays live and leaves the inventory. |
| `plural-detach.yaml.liquid` | `sync-options-plural-detach` stays live and leaves the inventory. |
| `legacy-lifecycle-detach.yaml.liquid` | `sync-options-legacy-detach` stays live and leaves the inventory. |
| `argocd-detach-negative-control.yaml.liquid` | `sync-options-argocd-detach-negative-control` is deleted; Argo-only `detach` is unsupported. |
| `plural-delete-false-prune-control.yaml.liquid` | `sync-options-plural-delete-false-prune-control` is deleted; `Delete=False` alone does not disable pruning. |
| `plural-precedence-over-argocd-prune.yaml.liquid` | `sync-options-plural-precedence-over-argocd-prune` is deleted. The Plural `Delete=False` annotation takes precedence over, and is not merged with, Argo `Prune=False`. |

Keep `default-service-destroy.yaml.liquid`, `plural-delete-false.yaml.liquid`, and `argocd-delete-false.yaml.liquid` in Git for Stage C. After the first manifest-removal reconcile, wait for one more operator sync/relist and repeat the inventory and live-resource checks. The three ConfigMaps removed from inventory, `sync-options-plural-prune-delete-false`, `sync-options-plural-detach`, and `sync-options-legacy-detach`, must remain live and absent from `.status.components`. Do not use the live `config.k8s.io/owning-inventory` annotation to infer inventory state; prune-time detach can leave that annotation in place.

## Stage C: remove the ServiceDeployment manifest

In the disposable branch, remove `services/agent/28-sync-options.yaml.liquid` and commit. Wait for the parent entrypoint to prune `ServiceDeployment/agent-28-testing-sync-options`. Do not delete the ServiceDeployment directly while its manifest remains in the parent folder, because the parent can recreate it.

Confirm `sync-options-default-service-destroy`, `sync-options-plural-prune-false`, and `sync-options-argocd-prune-false` are gone. `sync-options-plural-delete-false` and `sync-options-argocd-delete-false` must remain live. Inspect those ConfigMaps and confirm `metadata.annotations["config.k8s.io/owning-inventory"]` is absent. The ConfigMaps detached during Stage B, `sync-options-plural-prune-delete-false`, `sync-options-plural-detach`, and `sync-options-legacy-detach`, must remain live; their owning annotation is not part of the prune-time assertion.
