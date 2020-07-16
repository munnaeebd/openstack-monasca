# To install Brilliant Certificate
~~~
kubectl -n monasca delete ingress grafana-ingress

kubectl -n monasca create secret tls grafana-mon-ingress --cert brilliant.crt --key brilliant.key
kubectl create -f grafana-mon-ingress.yaml
