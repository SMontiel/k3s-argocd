# k3s-argocd

´´´ bash
k3d cluster create <cluster-name>
# or create it disabling Traefik
k3d cluster create local --k3s-arg "--disable=traefik@server:0"

k3d cluster create local --k3s-arg "--disable=traefik@server:0" -p 8080:80@loadbalancer -p 8443:443@loadbalancer

k3d kubeconfig print <cluster-name> > ~/.kube/local-cluster-config
export KUBE_CONFIG_PATH=~/.kube/local-cluster-config

# install k8s-dashboard
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
sudo k3s kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml

# nginx-ingress controller
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

# install cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.9.1 \
  --set installCRDs=true
# create a ClusterIssuer with name selfsigned-cluster-issuer
kubectl apply -f selfsigned-cluster-issuer.yaml
kubectl apply -f bootstrap-ca.yaml
kubectl apply -f example-com.cert.yaml

# echo-server
helm upgrade -i echo-server ealenn/echo-server --namespace testing --create-namespace --force
´´´
