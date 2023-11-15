# 10 Reasons to Choose Red Hat Advanced Cluster Management (RHACM) for GitOps with ArgoCD

## 1. UI-Support: ApplicationSets, Single Pane of Glass

RHACM offers a user-friendly interface with support for ApplicationSets, allowing you to define and manage multiple applications across clusters. The Single Pane of Glass view provides a consolidated overview of your entire multi-cluster environment, simplifying the monitoring and management of applications.

## 2. Integration with MultiCluster Search

Efficiently locate and manage resources across multiple clusters with RHACM's integrated MultiCluster Search. This feature empowers administrators to quickly identify and act upon resources, enhancing the overall observability and control of the Kubernetes landscape.

## 3. Pull-Model Option

RHACM supports a pull-model option, allowing you to synchronize configurations from Git repositories using ArgoCD. This flexibility enables you to choose the workflow that best fits your organizational needs, whether it's a pull or push model. Enjoy the benefits of the Pull-Model option, such as enhanced performance, managing GitOps installations with policies, and comprehensive AppSet summary reports.

**Advantages of the Pull-Model:**
The advantage of the pull model is decentralized control, where each cluster has its own copy of the configuration and is responsible for pulling updates independently. The hub-managed architecture using Argo CD and the pull model can reduce the need for a centralized system to manage the configurations of all target clusters, making the system more scalable and easier to manage. However, note that the hub cluster itself still represents a potential single point of failure, which you should address through redundancy or other means.

Additionally, the pull model provides more flexibility, allowing clusters to pull updates on their schedule and reducing the risk of conflicts or disruptions.

## 4. Integration with Cluster Lifecycle

Streamline the cluster lifecycle management with RHACM's integrated tools. When you integrate with the GitOps operator for every managed cluster that is bound to the GitOps namespace through the placement and ManagedClusterSetBinding custom resources, a secret with a token to access the ManagedCluster is created in the namespace. This is required for the GitOps controller to sync resources to the managed cluster. When a user is given administrator access to a GitOps namespace to perform application lifecycle operations, the user also gains access to this secret and admin level to the managed cluster.

If this is not desired, instead of binding the user to the namespace-scoped admin role, use a more restrictive custom role with permissions required to work with application resources that can be created and used to bound the user. See the following ClusterRole example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: application-set-admin
rules:
- apiGroups:
  - argoproj.io
  resources:
  - applicationsets
  verbs:
  - get
  - list
  - watch
  - update
  - delete
  - deletecollection
  - patch

## 5. Strong Built-in RBAC Support with Centralized Push Model

Before the new feature, end-users use a cluster admin SA to deploy applications when using ArgoCD push model in ACM. With the new feature, end-users can deploy applications using a customized service account with specific permissions.

**Leveraging Managed-Service-Account:**
Managed Service Account is an OCM addon enabling a hub cluster admin to manage service accounts across multiple clusters with ease. By controlling the creation and removal of the service account, the addon agent will monitor and rotate the corresponding token back to the hub cluster. To grant permissions to the new service account, we leverage the new cluster permission resource.

```yaml
apiVersion: rbac.open-cluster-management.io/v1alpha1
kind: ClusterPermission
metadata:
  name: clusterpermission-msa-subject-sample
  namespace: cluster1
spec:
  roles:
  - namespace: mortgage
    rules:
    - apiGroups: ["apps"]
      resources: ["deployments"]
      verbs: ["get", "list", "create", "update", "delete", "patch"]
    - apiGroups: [""]
      resources: ["configmaps", "secrets", "pods", "services", "namespace"]
      verbs: ["get", "update", "list", "create", "delete", "patch"]
    roleBindings:
  - namespace: mortgage
    roleRef:
      kind: Role
    subject:
      apiGroup: authentication.open-cluster-management.io
      kind: ManagedServiceAccount
      name: managed-sa-sample
  clusterRoleBinding:
    subject:
      apiGroup: authentication.open-cluster-management.io
      kind: ManagedServiceAccount
      name: managed-sa-sample

## 6. Integration with Placement for Advanced MultiCluster Scheduling

Take advantage of advanced scheduling capabilities with RHACM's integration with Placement. Efficiently distribute workloads across clusters based on defined policies, optimizing resource utilization and improving overall cluster performance.

## 7. MultiCluster Optimized Dashboards

Visualize and analyze the health and performance of your multi-cluster environment through RHACM's optimized dashboards. Gain insights into the status of applications, clusters, and resources, facilitating informed decision-making.

**Key Metrics to Monitor:**

Here are some key metrics you might want to include in your Grafana dashboards for monitoring ApplicationSets:

- **ApplicationSet Sync Status:**
  ```promql
  argocd_app_set_sync_status{namespace="argocd", applicationset="your-applicationset-name"}


object-templates-raw: |
  {{ range $placedec := (lookup "cluster.open-cluster-management.io/v1beta1" "PlacementDecision" "openshift-gitops" "" "cluster.open-cluster-management.io/placement=aws-app-placement").items }}
  {{ range $clustdec := $placedec.status.decisions }}
  - complianceType: musthave
    objectDefinition:
      apiVersion: authentication.open-cluster-management.io/v1alpha1
      kind: ManagedServiceAccount
      metadata:
        name: managed-sa-sample
        namespace: {{ $clustdec.clusterName }}
      spec:
        rotation: {}
  - complianceType: musthave
    objectDefinition:
      apiVersion: rbac.open-cluster-management.io/v1alpha1
      kind: ClusterPermission
      metadata:
        name: clusterpermission-msa-subject-sample
        namespace: {{ $clustdec.clusterName }}
  {{ end }}
  {{ end }}

## 10. Advanced Disaster Recovery Capabilities with ODF Integration

Integrate RHACM with Open Data Foundation (ODF) for advanced Disaster Recovery (DR) capabilities. Ensure the resilience of your applications and data across clusters, minimizing downtime and providing a reliable solution for business continuity.

## Summary

Red Hat Advanced Cluster Management for Kubernetes (RHACM) stands out as a powerful solution for GitOps with ArgoCD. Whether you are looking for centralized control with a push model or decentralized flexibility with a pull model, RHACM has you covered. From UI-Support and MultiCluster Search to strong RBAC support and advanced disaster recovery capabilities with ODF integration, RHACM provides a feature-rich experience.

Integrations with Managed Service Account, Governance, Placement, Gatekeeper, and ODF enhance the scalability, security, and ease of management of your Kubernetes deployments. Choose RHACM for a robust, user-friendly, and comprehensive solution to manage your multi-cluster environments effectively.
