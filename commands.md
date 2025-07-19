docker exec dataphora-control-plane printenv
docker exec dataphora-control-plane cat /etc/kubernetes/manifests/kube-apiserver.yaml
docker exec -it dataphora-control-plane bash

---

kubeadm certs check-expiration
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep validity -i -A2

---

alias k=kubectl
dry="--dry-run=client -o yaml"

---

curl https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml -L -O
curl https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml -L -o metrics-server.yaml

---

k config get-contexts
k config use-context xxx
k config set-context --current --namespace=gsbn
k config view --minify

---

k create deploy nginx --image=nigix $dry
k expose deploy nginx --port=8000 --target-port=80 $dry

---

k create service clusterip myservice --tcp=8000:80 $dry
k create service nodeport myservice --tcp=8000:80 $dry

---

k run bb --image=busybox -it --rm
# nc -zv hlag.com 443
# exit
pod "bb" deleted

---

k run busybox --image=busybox -- sleep 1000
k get pod busybox
k exec busybox -- nc -zv kubernetes 443
k exec busybox -- nslookup kubernetes.io
k exec busybox -- sh -c "while true; do echo hello; sleep 1; done"
k delete pod busybox

---

k run nginx --image=nginx
k get pod nginx
k exec nginx -- curl -L https://kubernetes.io
k exec nginx -it -- bash
k delete pod nginx

---

k config set-context --current --namespace=argocd

---

k run ubuntu --image=ubuntu --rm -it -- bash
k run bitnami --image=bitnami/git --rm -it -- bash
k run alpine --image=alpine/curl --rm -it -- sh

---

k run alpine --image=alpine/curl --rm -it -- sh
nc -zv kubernetes 443
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) && echo $TOKEN
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

---

k get pod -o wide
k get pod --show-labels

---

k get pod -o jsonpath="{.items[0].metadata.uid}"
k get pod -o jsonpath="{range .items[*]}{.metadata.uid}{'\t'}{.metadata.name}{'\n'}{end}"
k get pod -o custom-columns=POD_NAME:.metadata.name,POD_IP:.status.podIP,CREATED_AT:.status.startTime

---

k proxy --port=8090 &
curl http://localhost:8090/api/

---

k port-forward -n mongodb svc/mongodb-headless 27017:27017
mongosh admin --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD

---

k logs accsvc-7f9fbf886f-zxlpx -n gsbn --tail 10
k logs --selector app=nginx

---

echo 'xxx' | base64 -d > seema.crt
TOKEN=$(echo 'xxx' | base64 -d | tr -d "\n")

---

service kubelet status
service kubelet reload
service kubelet restart

---

k rollout history deploy/my-deploy
k rollout undo deploy/my-deploy
k rollout status deploy/my-deploy

---

k scale deploy my-deploy --replicas=2
k autoscale deploy my-deploy --cpu-percent=80 --min=1 --max=5

---

