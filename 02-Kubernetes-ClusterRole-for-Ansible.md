```
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ansible-cluster-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "endpoints", "serviceaccounts"]
    verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]
  - apiGroups: [""]
    resources: ["serviceaccounts/token"]
    verbs: ["create"]
EOF
```
```
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ansible-cluster-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ansible-cluster-role
subjects:
  - apiGroup: ""
    kind: ServiceAccount
    namespace: ansible
    name: ansible-sa
EOF
```
```
kubectl auth can-i create serviceaccounts/token --as system:serviceaccount:ansible:ansible-sa
```