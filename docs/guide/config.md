# Instalação e Configuração com Ansible

Nesta etapa, faremos as instalações usando Ansible, que executa os comandos de Helm e faz algumas configurações
  como a do ***ingress controller***. Os charts são instalados com o tipo do serviço como NodePort
  para que o Load Balancer Controller crie o Ingress e o Application Load Balancer.


Para executar as playbooks é preciso ***"apontar"*** os comandos para o cluster EKS usando a AWS CLI:

#### Comando AWS CLI
- `aws eks --region` **sua_regiao** `update-kubeconfig --name` seu_cluster - Atualiza o contexto do kube config.

### Wordpress
Nessa playbook o Ansible executa os comandos de Helm que instalam o "pacote" do Wordpress e MariaDB já configurados.

``` yaml title="wordpress.yaml"
---
- name: Install and config wordpress
  hosts: localhost

  tasks:

    - name: Add bitnami repo
      shell: "helm repo add bitnami https://charts.bitnami.com/bitnami"

    - name: Update repo
      shell: "helm repo update"

    - name: Deploy latest version of wordpress chart inside ingress namespace (and create it)
      shell: "helm upgrade --install wordpress oci://registry-1.docker.io/bitnamicharts/wordpress -n wordpress --set service.type=NodePort --create-namespace"
      # environment:
      #   KUBECONFIG: /root/.kube/config

    - name: Apply ingress rules with kubectl (on wordpress app namespace)
      command: kubectl apply -f alb-wordpress.yaml

```
### ALB 🔒

``` yaml title="alb-wordpress.yaml"
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wordpress-ingress
  namespace: wordpress
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm::certificate # - your certificate ARN
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  ingressClassName: alb
  rules:
    - host: wp.ignitehome.online
      http:
        paths:
          - path: /admin
            pathType: Exact
            backend:
              service:
                name: wordpress
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: wordpress
                port:
                  number: 80
```

### Prometheus

``` yaml title="prometheus.yaml"
---
- name: Install and config wordpress
  hosts: localhost

  tasks:
    - name: Add prometheus repo
      shell: "helm repo add prometheus-community https://prometheus-community.github.io/helm-charts"

    - name: Update repo
      shell: "helm repo update"

    - name: Deploy latest version of prometheus chart inside ingress namespace (and create it)
      shell: "helm upgrade --install prometheus prometheus-community/prometheus -n prometheus --create-namespace"
       environment:
         KUBECONFIG: /root/.kube/config

```

### Grafana

``` yaml title="grafana.yaml"
---
- name: Install and config wordpress
  hosts: localhost

  tasks:
    - name: Add grafana repo
      shell: "helm repo add grafana https://grafana.github.io/helm-charts"

    - name: Update repo
      shell: "helm repo update"

    - name: Deploy latest version of grafana chart inside ingress namespace (and create it)
      shell: "helm upgrade --install grafana grafana/grafana  --create-namespace -n grafana --set persistence.enabled=true --set adminPassword='EKS123' --values https://raw.githubusercontent.com/AndrewVictorSilva/apoioprojetofinal/main/grafana-data-source.yaml  --set service.type=NodePort"
      environment:
        KUBECONFIG: /root/.kube/config
    - name: Apply ingress rules with kubectl (on wordpress app namespace)
      command: kubectl apply -f alb-grafana.yaml

```
### Grafana ALB 🔒

``` yaml title="alb-grafana.yaml"
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: grafana
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm::certificate # - your certificate ARN
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  ingressClassName: alb
  rules:
    - host: gf.ignitehome.online
      http:
        paths:
          - path: /login
            pathType: Exact
            backend:
              service:
                name: grafana
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: grafana
                port:
                  number: 80

```