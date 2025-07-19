```
openssl genrsa -out seema.key 2048
openssl req -new -key seema.key -out seema.csr -subj "/CN=seema"
```
```
k apply -f - <<EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: seema
spec:
  request: $(cat seema.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 7776000
  usages:
  - client auth
EOF
```
```
k get csr
k certificate approve seema
k get csr
```
```
k get csr seema -o jsonpath='{.status.certificate}' | base64 -d > seema.crt
openssl x509 -noout -text -in seema.crt
```
```
CLUSTER_NAME=$(k config view --minify -o jsonpath='{.clusters[].name}') && echo $CLUSTER_NAME
API_SERVER_URL=$(k config view --minify -o jsonpath='{.clusters[].cluster.server}') && echo $API_SERVER_URL
```
```
k config set-credentials seema --client-certificate=seema.crt --client-key=seema.key --embed-certs=true
k config set-context seema@$CLUSTER_NAME --cluster=$CLUSTER_NAME --user=seema
```
```
k apply -f - <<EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-viewer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
EOF
```
```
k create rolebinding seema-rolebinding --role=pod-viewer-role --user=seema
k auth can-i --as=seema get pod
k auth can-i --as=seema get deploy
```
```
cat ~/.kube/config | yq '.clusters.[] | select(.name == "'$CLUSTER_NAME'") | .cluster.certificate-authority-data' | base64 -d > ca.crt
openssl x509 -noout -text -in ca.crt
```
```
curl $API_SERVER_URL/api/v1/namespaces/default/pods --cacert ca.crt --cert seema.crt --key seema.key
curl $API_SERVER_URL/api/v1/namespaces/default/deployments --cacert ca.crt --cert seema.crt --key seema.key
```