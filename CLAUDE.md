# sample-api-config

Part of the platform portfolio. Workspace rules live in `../platform/CLAUDE.md`; read them first, then `../platform/docs/PLAN.md` for this repo's phase.

## This repo's job
Kustomize manifests for sample-api. base/ holds the environment-agnostic Deployment, Service, HPA, PDB, NetworkPolicy, ServiceAccount with Workload Identity annotations, and ExternalSecret. overlays/dev, stg and prod patch only replicas, resources, ingress host and the image tag. The sample-api pipeline opens PRs here to bump the dev image tag; humans open PRs to promote to stg and prod. ArgoCD syncs this repo. Phase 5 in the plan.

## Conventions specific to this repo
- base is env-agnostic. Overlays only patch replicas, resources, hosts, image tags and env-specific labels.
- Image tag is set through `images:` in the overlay kustomization.yaml so a pipeline can edit it with `kustomize edit set image`.
- Every workload passes the Kyverno policies from platform-argocd: requests and limits, probes, non-root, read-only rootfs, seccomp, no latest tag.
- One namespace per env: sample-api-dev, sample-api-stg, sample-api-prod.
- Branch policy: dev PRs from the pipeline auto-merge; stg and prod PRs need a human approval.

## Commands
```
pre-commit run --all-files
kustomize build overlays/dev
kustomize build overlays/dev | kubeconform -strict -ignore-missing-schemas
kustomize build overlays/dev | kyverno apply ../platform-argocd/addons/kyverno/policies -r -
cd overlays/dev && kustomize edit set image sample-api=acrplatdev.azurecr.io/sample-api:<sha>
```

## Do not
- Do not put anything env-specific in base.
- Do not commit Secret objects. Use ExternalSecret.
- Do not kubectl apply from this repo. ArgoCD is the only writer to the cluster.
