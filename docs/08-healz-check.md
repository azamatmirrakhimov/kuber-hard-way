# You can set up a basic nginx proxy for the healthz endpoint by first installing nginx":
~~~
sudo apt-get install -y nginx-full
~~~
# Create an nginx configuration for the health check proxy:
~~~
cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
    proxy_pass                    https://127.0.0.1:6443/healthz;
    proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
~~~
# Set up the proxy configuration so that it is loaded by nginx:
~~~
sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
~~~
~~~
sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
~~~
~~~
sudo systemctl restart nginx
~~~
~~~
sudo systemctl enable nginx
~~~
# You can verify that everything is working like so:
~~~
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
~~~
### You should receive a 200 OK response.

# RBAC for Kubelet Authorization

# Create a role with the necessary permissions:
~~~
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
~~~
# Bind the role to the kubernetes user:
~~~
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
~~~

