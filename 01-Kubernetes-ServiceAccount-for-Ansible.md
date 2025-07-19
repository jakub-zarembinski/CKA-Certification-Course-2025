```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: ansible
EOF
```
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: ansible
  name: ansible-sa
EOF
```
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  namespace: ansible
  name: ansible-sa-secret
  annotations:
    kubernetes.io/service-account.name: ansible-sa
type: kubernetes.io/service-account-token
EOF
```