# Deploy Community Edition

Note: the instruction depends on Helm v3.

1. Clone this github repo
```
git clone https://github.com/bytewedge/bytewedge.git
```
2. cd to charts directory
```
cd bytewedge/charts
```
3. Create namespace
```
kubectl create namespace bwdemo
```
4. download Helm chart dependencies
```
helm dep up setup
```
5. install dependencies 
```
helm install svc setup -n bwdemo
```
6. prepare ByteWedge meta header db and blob store
```
helm install schema jobs -n bwdemo
```
7. install ByteWedge cluster
```
helm install bw bytewedge --set ingress.domain=bwdemo.bytewedge.local -n bwdemo
```
8. verify installation

verify ingress
```
kubectl get ingress -n bwdemo
```
it should show both ui and injector

verify all pods
```
kubectl get all -n bwdemo
```
