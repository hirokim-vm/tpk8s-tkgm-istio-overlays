Tanzu Platform for K8s with Tanzu Kubernetes Grid 2.5.2

The observability capability package for Tanzu Kubernetes for K8s uses Istio technology to intercept traffic and enhance the metadata of metrics.
As of 2024/10, this customization method is not officially supported, but we will discuss here how to customize it to support various uses of Istio.
The following is a sample procedure for customizing MeshConfig, which controls settings that affect the entire service mesh.

Prerequisites
　- tkgm must be attached to a cluster group where the Application Engine of Tanzu Kubernetes for K8s is enabled.
　- The following minimum capabilities are installed here to see the customizability of Istio. * This does not take into account deploying applications in Space.
Certificate Manager
Service Mesh Observability * Ingress capability is also installed

As tracing is not enabled by default in Service Mesh Observability, we will add a setting to configmap/istio.

```
# Default state
kubectl get cm istio -n istio-system -oyaml
apiVersion: v1
data:
  mesh: |-
    defaultConfig:
      discoveryAddress: istiod.istio-system.svc:15012
      proxyMetadata:
        ISTIO_META_DNS_AUTO_ALLOCATE: "true"
      tracing:
        zipkin:
          address: zipkin.istio-system:9411
...
```

Create an overlay secret that contains the customization settings.
See overlays/tcs-overlay-secret.yaml. Include the settings that overlay data.mesh.enableTracing: true, which is required to enable tracing.

Next, target your new clustergroup and apply the overlay secrets to the Tanzu Platform cluster group:

```
tanzu project use <project>
tanzu operations clustergroup use <cg>

tanzu deploy --only overlays

kubectl annotate pkgi tcs.tanzu.vmware.com 'ext.packaging.carvel.dev/ytt-paths-from-secret-name.0=tcs-overlay' -n tanzu-cluster-group-system
 * For the “Continue?” prompt, enter “y”.
```

Check that the settings have been applied.

```
# After applying overlay secret
kubectl get cm istio -n istio-system -oyaml
apiVersion: v1
data:
  mesh: |-
    enableTracing: true  <- 追加された
    defaultConfig:
      discoveryAddress: istiod.istio-system.svc:15012
      proxyMetadata:
        ISTIO_META_DNS_AUTO_ALLOCATE: "true"
      tracing:
        zipkin:
          address: zipkin.istio-system:9411
...
```

In order to actually send traces to the Observability tool, settings such as ExtensionProvider are required, but the ability to customize MeshConfig allows for flexible settings.
