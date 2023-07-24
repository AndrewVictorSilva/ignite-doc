# Instalação de Configuração com Ansible

Nesta etapa, faremos as instalações usando Ansible, que executa os comandos de Helm e faz algumas configurações
  como a do ***ingress controller***

Para executar as playbooks é preciso ***"apontar"*** os comandos para o cluster EKS usando a AWS CLI:

#### Comando AWS CLI
- `aws eks --region` **sua_regiao** `update-kubeconfig --name` seu_cluster - Atualiza o contexto do kube config.

### Wordpress Helm Playbook

``` yaml title="wordpress.yaml"
---
- name: Install and config wordpress
  hosts: localhost

  tasks:

    - name: Add bitnami repo
      shell: "helm repo add bitnami https://charts.bitnami.com/bitnami"

    - name: Update repo
      shell: "helm repo update"

    - name: Deploy latest version of wordpress chart inside wordpress namespace (and create it)
      kubernetes.core.helm:
        name: wordpress
        chart_ref: oci://registry-1.docker.io/bitnamicharts/wordpress
        release_namespace: wordpress
        create_namespace: true
      environment:
        KUBECONFIG: /root/.kube/config

```

### Ingress Helm Playbook

``` yaml title="ingress.yaml"
---
- name: Install and config wordpress
  hosts: localhost

  tasks:

    - name: Add nginx repo
      shell: "helm repo add nginx https://helm.nginx.com/stable"

    - name: Update repo
      shell: "helm repo update"

    - name: Deploy latest version of nginx-ingress chart inside ingress namespace (and create it)
      shell: "helm upgrade --install eks-ingress nginx/nginx-ingress --version 0.17.0 -n ingress-nginx --create-namespace -n ingress"
      environment:
        KUBECONFIG: /root/.kube/config

```

### Ingress Configuration Playbook

``` yaml title="ingress.yaml"
---
- name: Download and Update ingress-app.yaml
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Get Current Working Directory
      command: pwd
      register: pwd_output

    - name: Download ingress-app.yaml
      get_url:
        url: "github_plubic_file.yaml"
        dest: "{{ pwd_output.stdout }}/ingress-app.yaml"

    - name: Read ingress-app.yaml template
      slurp:
        path: "{{ pwd_output.stdout }}/ingress-app.yaml"
      register: ingress_app_template

    - name: Retrieve Load Balancer Address and Perform nslookup
      command: kubectl get svc -n wordpress wordpress --output json
      register: svc_output
      #environment:
      #  KUBECONFIG: /root/.kube/config

    - name: Store Load Balancer Address
      set_fact:
        load_balancer_address: "{{ svc_output.stdout | from_json | json_query('status.loadBalancer.ingress[0].hostname') }}"

    - name: Wait for Load Balancer to be Online
      wait_for:
        timeout: 300 # Adjust the timeout value as needed
        delay: 10 # Adjust the delay between checks as needed
        host: "{{ load_balancer_address }}"
        port: 80
        state: started

    - name: Perform nslookup
      command: nslookup "{{ load_balancer_address }}"
      register: nslookup_output

    - name: Extract IP Addresses
      set_fact:
        ip_addresses: "{{ nslookup_output.stdout_lines | select('regex', '^Address: ') | map('regex_replace', '^Address: (.+)', '\\1') | list }}"

    - name: Store IP Addresses in Variables
      set_fact:
        first_ip: "{{ ip_addresses[0] }}"
        second_ip: "{{ ip_addresses[1] }}"

    - name: Substitute IP addresses in ingress-app.yaml
      set_fact:
        ingress_app_content: "{{ ingress_app_template.content | b64decode | replace('$first_ip', first_ip) | replace('$second_ip', second_ip) }}"

    - name: Write updated ingress-app.yaml
      copy:
        content: "{{ ingress_app_content }}"
        dest: "{{ pwd_output.stdout }}/ingress-app.yaml"
        force: yes

    - name: Apply ingress rules with kubectl (on wordpress app namespace)
      command: kubectl apply -f {{ pwd_output.stdout }}/ingress-app.yaml -n wordpress

    - name: Print Load Balancer Address and IP Addresses
      debug:
        msg: "Load Balancer Address: {{ load_balancer_address }}\nIP Addresses: {{ ip_addresses }}"

```


Some example text