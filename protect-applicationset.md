To protect an ApplicationSet from accidental deletion, you can implement the following approaches:

Preserving Resources on Deletion in ApplicationSet YAML:

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: helmtest
  namespace: openshift-gitops
spec:
  syncPolicy:
    preserveResourcesOnDeletion: true
  generators:
    - clusterDecisionResource:
        configMapRef: acm-placement
        labelSelector:
          matchLabels:
            cluster.open-cluster-management.io/placement: helmtest-placement
        requeueAfterSeconds: 180
  template:
    metadata:
      name: helmtest-{{name}}
      labels:
        velero.io/exclude-from-backup: "true"
    spec:
      project: default
      sources:
        - repositoryType: helm
          repoURL: https://redhat-developer.github.io/redhat-helm-charts
          chart: quarkus
          targetRevision: 0.0.3
      destination:
        namespace: remote
        server: "{{server}}"
      syncPolicy:
        automated:
          selfHeal: true
          prune: false
        syncOptions:
          - CreateNamespace=true
          - PruneLast=false
```


Applying Labels to Protect Resources on Kubernetes Cluster:
Apply the following label to protect resources:

```
argocd.argoproj.io/sync-options: Prune, Delete=false
```

Gatekeeper Constraint Template:
Write a Gatekeeper Constraint Template to restrict deletion of ApplicationSets unless the user/group has the role of ApplicationSet-Admin and specify the names of the ApplicationSets to be checked. Here's an example:


```
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: applicationset-deletion-restriction
spec:
  crd:
    spec:
      names:
        kind: ApplicationSetDeletionRestriction
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8s.appsetdeletionrestriction
        violation[{"msg": msg}] {
          input.request.kind.kind == "ApplicationSet"
          not input.request.operation == "CREATE"
          user_groups[user] := input.user.groups[_]
          user_roles := {role | role := input.user.roles[_]}
          not (input.request.object.metadata.name == input.parameters.allowedNames[_])
          not (any(role == input.parameters.allowedRoles[_]))
          msg := sprintf("User %v is not authorized to delete ApplicationSet", [input.user.username])
        }
```
And here's how you would write a Constraint to use this template, specifying the allowed roles and ApplicationSet names:

```
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: ApplicationSetDeletionRestriction
metadata:
  name: applicationset-deletion-restriction
spec:
  match:
    kinds:
      - apiGroups: ["argoproj.io"]
        kinds: ["ApplicationSet"]
  parameters:
    allowedRoles:
      - "ApplicationSet-Admin"
      # Add more allowed roles as needed
    allowedNames:
      - "helmtest"
      - "another-applicationset"
      # Add more ApplicationSet names as needed
```



This best way however is to create a RHACM Policy to generate and monitor the ApplicationSet

1. When you delete the Policy be incident the ApplicationSet remains by default
2. If you delete the ApplicationSet gets deleted, then the PolicyController would recreate it after a short notive





By implementing these measures, you can safeguard your ApplicationSets from accidental deletion and mitigate potential outages.
