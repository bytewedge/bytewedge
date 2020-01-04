# Deploy Community Edition

Note: the instruction depends on Helm v3.

1. Clone this github repo and timescaledb kubernetes repo
```
git clone https://github.com/bytewedge/bytewedge.git
git clone https://github.com/timescale/timescaledb-kubernetes.git
```
2. cd to charts directory
```
cd bytewedge/charts
```
3. Create namespace
```
kubectl create namespace bwdemo
```
4. download bytewedge chart dependencies
```
helm dep up bytewedge
```
5. install ByteWedge cluster
```
helm install bw bytewedge --set ingress.domain=bytewedge.local -n bwdemo
```
6. verify installation
```
❯❯❯ kubectl get ingress -n bwdemo
NAME                    HOSTS                      ADDRESS         PORTS   AGE
bw-bytewedge-injector   injector.bytewedge.local   192.168.64.28   80      12h
bw-bytewedge-web        ui.bytewedge.local         192.168.64.28   80      12h
```