# Create 2 AKS clusters with AAD enabled

https://learn.microsoft.com/en-us/azure/aks/enable-authentication-microsoft-entra-id

https://learn.microsoft.com/en-us/azure/aks/azure-ad-rbac?tabs=portal

```
az aks create -g tap-rbac -n tap17-rbac --enable-aad --aad-admin-group-object-ids YYYY --node-vm-size Standard_A4_v2  --location japaneast --disable-local-accounts
az aks create -g tap-rbac -n tap19-rbac --enable-aad --aad-admin-group-object-ids YYYY --node-vm-size Standard_A4_v2  --location japaneast --disable-local-accounts
```

Added to cluster-admin group https://portal.azure.com/#view/Microsoft_AAD_IAM/GroupDetailsMenuBlade/~/Overview/groupId/b1e05aa9-d0fb-478e-b7cc-867b4d591138/menuId/

```
az role assignment create --role "Azure Kubernetes Service Cluster User Role"  --assignee YYYYY --scope /subscriptions/ZZZZZZ/resourcegroups/tap-rbac/providers/Microsoft.ContainerService/managedClusters/tap17-rbac
az role assignment create --role "Azure Kubernetes Service Cluster User Role"  --assignee YYYYY --scope /subscriptions/ZZZZZZ/resourcegroups/tap-rbac/providers/Microsoft.ContainerService/managedClusters/tap19-rbac
```

```
az aks get-credentials --resource-group tap-rbac --name tap17-rbac
az aks get-credentials --resource-group tap-rbac --name tap19-rbac
```

# Install TAP 

```
ceip_policy_disclosed: true
shared:
  image_registry:
    project_path: ghcr.io/mhoshi-vm/tap
    secret:
      name: private-repo
      namespace: tap-install
  ingress_domain: tap17.aks.lespaulstudioplus.info
  ingress_issuer: letsencrypt
tap_gui:
  app_config:
    auth:
      environment: development
      providers:
        microsoft:
          development:
            clientId: XXXXXX
            clientSecret: XXXX
            tenantId: XXXX
    kubernetes:
      clusterLocatorMethods:
        - clusters:
            - authProvider: aks
              name: tap-rbac
              skipMetricsLookup: true
              skipTLSVerify: true
              url: https://tap-rbac-tap-rbac-82c5fd-q31ifc2h.hcp.japaneast.azmk8s.io:443
          type: config
      serviceLocatorMethod:
        type: multiTenant
```

```
ceip_policy_disclosed: true
shared:
  image_registry:
    project_path: ghcr.io/mhoshi-vm/tap
    secret:
      name: private-repo
      namespace: tap-install
  ingress_domain: tap19.aks.lespaulstudioplus.info
  ingress_issuer: letsencrypt
tap_gui:
  app_config:
    auth:
      environment: development
      providers:
        microsoft:
          development:
            clientId: XXXX
            clientSecret: XXX
            tenantId: XXXX
    kubernetes:
      clusterLocatorMethods:
        - clusters:
            - authProvider: aks
              name: tap-rbac
              skipMetricsLookup: true
              skipTLSVerify: true
              url: https://tap-rbac-tap-rbac-82c5fd-q31ifc2h.hcp.japaneast.azmk8s.io:443
          type: config
      serviceLocatorMethod:
        type: multiTenant
```

# Update Web Redirect URI

https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationMenuBlade/~/Overview/appId/f9254a24-03b6-4d54-845f-fd4580ef1a31/isMSAApp~/false

# Install Manifests

```
cd manifests/
kubectl apply -f .
kubectl apply -f .
```

# Create kubeconfig

```
cp ~/.kube/config ./tap19-kubeconfig.yaml
export KUBECONFIG=./tap19-kubeconfig.yaml
TOKEN=`kubectl get secret admin-account -n kube-system -o jsonpath='{.data.token}' | base64 -d`
kubectl config set-credentials app-editor --token=$TOKEN
kubectl config set-context --current --user=app-editor
unset KUBECONFIG
```

```
cp ~/.kube/config ./tap17-kubeconfig.yaml
export KUBECONFIG=./tap17-kubeconfig.yaml
TOKEN=`kubectl get secret admin-account -n kube-system -o jsonpath='{.data.token}' | base64 -d`
kubectl config set-credentials app-editor --token=$TOKEN
kubectl config set-context --current --user=app-editor
unset KUBECONFIG
```