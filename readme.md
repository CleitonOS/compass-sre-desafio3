
<p align="center">
  <a href="" rel="noopener">
 <img max-width=400px height=100px src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/45/Logo_CompassoUOL_Positivo.png/1200px-Logo_CompassoUOL_Positivo.png" alt="Project logo"></a>
</p>

<h1 align="center">Fazendo o deploy do Kibana em um cluster (minikube)</h1> 
<p align="center"><i>Criando um Cluster ECK com minikube e fazendo o deploy do Kibana e Elasticsearch </i></p>

## 📑 Requisitos

- Minikube instalado - [Documentação Minikube - Instalação](https://minikube.sigs.k8s.io/docs/start/)
- Docker instalado - [Documentação Docker - Instalação para CentOS](https://docs.docker.com/engine/install/centos/)

## 📝 Tabela de conteúdos
- [Fazendo a instalação do Ansible (Passo 1)](#step1)
- [Criando e exceutando um ansible-playbook (Passo 2)](#step2)
- [Referências](#documentation)

## ⚙️ Iniciando o cluster do minikube (Passo 1)<a name = "step2"></a>

- Após instalar e configurar os requisitos necessários, vamos criar o cluster do minikube:

    ```
    minikube start --cpus 4 --memory 6144
    ```

    - Precisaremos de bastante memória para evitar engasgos durante as cargas de trabalho.

## ⚙️🔽 Instalando Kibana e o Elasticsearch (Passo 2)<a name = "step3"></a>

- Começando instalando "custom resource definitions":

    ```
    kubectl create -f https://download.elastic.co/downloads/eck/2.10.0/crds.yaml
    ```

- Instalar o operador com suas regras RBAC

    ```
    kubectl apply -f https://download.elastic.co/downloads/eck/2.10.0/operator.yaml
    ```

- Vamos usar o operador oficial do Elasticsearch para Kubernetes para facilitar a implantação do Elastisearch:
- Observação: o operador Elasticsearch, pode ser usado para implantar e gerenciar instâncias do Elasticsearch e Kibana.

1. Implante o cluster do Elasticsearch (com um node elasticsearch):

- Vamos aplicar uma especificação simples de cluster elasticsearch

    ```
    cat <<EOF | kubectl apply -f -
    apiVersion: elasticsearch.k8s.elastic.co/v1
    kind: Elasticsearch
    metadata:
      name: quickstart
    spec:
      version: 8.11.1
      nodeSets:
      - name: default
        count: 1
        config:
          node.store.allow_mmap: false
    EOF
    ```

- Verifique o estado do cluster criado:

    ```
    kubectl get elasticsearch
    ```

    - Observação: O cluster pode demorar uns 5 minutos para ser iniciado e ficar no estado "green" de saúde. 

    ```
    NAME          HEALTH    NODES     VERSION   PHASE         AGE
    quickstart    green     1         8.11.1     Ready         1m
    ```

- Você também pode checar se o pod do elasticsearch está rodando com:

    ```
    kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'
    ```

2. Implantando uma instância do Kibana:

- Especificando uma instância do Kibana e associando ao cluster do Elasticsearch:

    ```
    cat <<EOF | kubectl apply -f -
    apiVersion: kibana.k8s.elastic.co/v1
    kind: Kibana
    metadata:
      name: quickstart
    spec:
      version: 8.11.1
      count: 1
      elasticsearchRef:
        name: quickstart
    EOF
    ```

- Verifique os recursos criados:

    ```
    kubectl get kibana
    ```

    ```
    kubectl get pod --selector='kibana.k8s.elastic.co/name=quickstart'
    ```

- Cheque se o serviço http do kibana está sendo executado:

    ```
    kubectl get service quickstart-kb-http

    ```


3. Faça o encaminhamento de porta diretamenta para o Pod do Kibana

    ```
    kubectl port-forward service/quickstart-kb-http 5601
    ```

- Abra no seu browser "https://localhost:5601"

- Para logar como usuário "elastic" e obter a senha necessária, use esse comando:

    ```
    kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
    ```

## Referências utilizadas:<a name="documentation"></a>

- [K8S Deploy ECK](https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-deploy-eck.html#k8s-deploy-eck)

- [Elasticsearch cluster - quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-deploy-elasticsearch.html) 

- [Kibana instance - quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-deploy-kibana.html)
