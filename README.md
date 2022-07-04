# 1. Introduction to Istio Service Mesh

## What's a service Mesh?

A service mesh is a distributed application infrastructure that is responsible for handling network traffic on behalf of the application in a transparent, out-of-process manner. Figure shows how service proxies form the data plane through which all traffic is handled and observed. The data plane is responsible for establishing, securing, and controlling the traffic through the mesh. The data plane behavior is configured by the control plane. The control plane is the brains of the mesh and exposes an API for operators to manipulate network behaviors. Together, the data plane and the control plane provide important capabilities necessary in any cloud-native architecture:

* Service resilience
* Observability signals
* Traffic control capabilities
* Security
* Policy enforcement

## Where Istio fits in distributed architectures

![Image](Istio_fit.png?raw=true)

## Istio Architecture Diagram

![Image](Istio_Architecture.png?raw=true)

## Running K8s and Istio

### Steps to follow to install istio service mesh

* Install aws-cli

```bash
brew install aws-cli
```

* Install aws-cli

```bash
brew install eksclti
```

* Create eks cluster

```bash
 eksctl create cluster --name k8s-istio --region=us-east-2 --nodes 2
```

* Get eks cluster kubeconfig

```bash
 aws eks update-kubeconfig --region us-east-2 --name k8s-istio
```

* Create key pair

```bash
aws ec2 create-key-pair \
    --key-name k8s-istio \
    --region us-east-2 \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > k8s-istio-key-pair.pem

```

* Create public key

```bash
ssh-keygen -y -f k8s-istio-key-pair.pem > k8-istio-public-key.pub

```

* Create node group with 3 nodes

```bash
eksctl create nodegroup \
  --cluster k8s-istio \
  --name k8-istio-nodes \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --ssh-access \
  --managed=false \
  --ssh-public-key k8-istio-public-key.pub \
  --region us-east-2
```

* Download Istio latest version

```bash
curl -L https://istio.io/downloadIstio | sh -
```

* Move to the Istio package directory. For example, if the package is istio-1.14.1:

```bash
cd istio-1.14.1
```

* Add the istioctl client to your path (Linux or macOS):

```bash
export PATH=$PWD/bin:$PATH
```

* For this installation, we use the demo configuration profile. It’s selected to have a good set of defaults for testing, but there are other profiles for production or performance testing.

```bash
istioctl install --set profile=demo -y
```

* Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:

```bash
kubectl label namespace default istio-injection=enabled
```

* Deploy the Bookinfo sample application into default namespace

```bash
kubectl apply -f istio-1.14.1/samples/bookinfo/platform/kube/bookinfo.yaml
```

* The application will start. As each pod becomes ready, the Istio sidecar will be deployed along with it.

```bash
kubectl get services
```

and

```bash
kubectl get pods
```

* Verify everything is working correctly up to this point. Run this command to see if the app is running inside the cluster and serving HTML pages by checking for the page title in the response:

```bash
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

response

```bash
<title>Simple Bookstore App</title>
```

* Associate this application with the Istio gateway:

```bash
kubectl apply -f istio-1.14.1/samples/bookinfo/networking/bookinfo-gateway.yaml
```

* Execute the following command to determine if your Kubernetes cluster is running in an environment that supports external load balancers:

```bash
kubectl get svc istio-ingressgateway -n istio-system
```

* Follow these instructions if you have determined that your environment has an external load balancer.

Set the ingress IP and ports:

```bash
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
```

* Set GATEWAY_URL:

```bash
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

* Ensure an IP address or hostname and port were successfully assigned to the environment variable:

```bash
echo "$GATEWAY_URL"
```

* Run the following command to retrieve the external address of the Bookinfo application.

```bash
echo "http://$GATEWAY_URL/productpage"
```

* Install Kiali and the other addons and wait for them to be deployed.

```bash
kubectl apply -f istio-1.14.1/samples/addons
```

```bash
kubectl rollout status deployment/kiali -n istio-system
```

* Access the Kiali dashboard

```bash
istioctl dashboard kiali
```
