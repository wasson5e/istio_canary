# Istio Canary Deployments
From Istio "Sidecar or Ambient"
>**Sidecar mode**
>
>Istio has been built on the sidecar pattern from its first release in 2017. Sidecar mode is well understood and thoroughly battle-tested, but comes with a resource cost and operational overhead.
>
>Each application you deploy has an Envoy proxy injected as a sidecar
>All proxies can process both Layer 4 and Layer 7

>**Ambient mode**
>
>Launched in 2022, ambient mode was built to address the shortcomings reported by users of sidecar mode. As of Istio 1.22, it is production-ready for single cluster use cases.
>
>All traffic is proxied through a Layer 4-only node proxy
>Applications can opt in to routing through an Envoy proxy to get Layer 7 features
>
[Ambient Mode](./Istio%20Ambient/README.md)

[Sidecar Mode](./Istio%20Sidecar/README.md)