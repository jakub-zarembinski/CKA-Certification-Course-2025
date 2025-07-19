```
API_SERVER_URL=$(kubectl config view --minify -o jsonpath='{.clusters[].cluster.server}') \
&& echo $API_SERVER_URL
```
```
TOKEN=$(kubectl get secret ansible-sa-secret -n ansible -o jsonpath="{.data.token}" | base64 -d) \
&& echo $TOKEN \
&& curl -X GET "$API_SERVER/api/v1/namespaces/ansible/pods/" -H "Authorization: Bearer $TOKEN" -k
```
```
TOKEN=$(kubectl create token ansible-sa -n ansible) \
&& echo $TOKEN \
&& curl -X GET "$API_SERVER/api/v1/namespaces/ansible/pods/" -H "Authorization: Bearer $TOKEN" -k
```
```
TOKEN=$(curl -X POST "$API_SERVER_URL/api/v1/namespaces/ansible/serviceaccounts/ansible-sa/token" \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{
    "apiVersion": "authentication.k8s.io/v1",
    "kind": "TokenRequest",
    "spec": {
        "audiences": ["https://kubernetes.default.svc.cluster.local"],
        "expirationSeconds": 3600
    }
}' \
-k -s \
| jq '.status.token' -r) \
&& echo $TOKEN \
&& curl -X GET "$API_SERVER/api/v1/namespaces/ansible/pods/" -H "Authorization: Bearer $TOKEN" -k
```