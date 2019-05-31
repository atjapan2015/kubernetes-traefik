# kubernetes-traefik

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: helm-user-cluster-admin-role
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: default
  namespace: kube-system
EOF
```

```
helm install stable/traefik \
  --name traefik-operator \
  --namespace traefik \
  --values traefik/values.yaml  \
  --set "kubernetes.namespaces={traefik}" \
  --wait
```


```
helm upgrade \
  --reuse-values \
  --set "kubernetes.namespaces={traefik,default}" \
  --wait \
  traefik-operator \
  stable/traefik
```

```
cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: sample-path-routing
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
    ingress.kubernetes.io/rewrite-target: /sample
    ingress.kubernetes.io/rule-type: PathPrefix
spec:
  rules:
  - host:
    http:
      paths:
      - path: /sample
        backend:
          serviceName: sample-service
          servicePort: 80
EOF
```
