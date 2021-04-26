
# Rancher Traefik v2.4.x HA K8s

Ubuntu 21.04 LTS

Docker 20.10.6

Kubernetes 1.20.5

Rancher 2.5.7

Traefik 2.4.x

# Ambiente

![arquitetura](https://user-images.githubusercontent.com/52961166/116088850-3fb05200-a670-11eb-8984-22dd6b6bfd68.png)


# Prerequisitos
	
	Faremos a instalação do Docker, e também iremos revisar a arquitetura do ambiente.

	É preciso entrar em todas as máquinas e instalar o Docker.

```ssh

RancherSerber - HOST A
k8s-1         - HOST B
k8s-2         - HOST C
k8s-3         - HOST D

$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
$ sudo apt update
$ apt-cache policy docker-ce
$ sudo apt install docker-ce
$ sudo systemctl status docker
$ sudo usermod -aG docker ubuntu
```
# Construindo sua aplicação

### Fazer build das imagens, rodar docker-compose.

Nesse exercício iremos construir as imagens dos containers que iremos usar, colocar elas para rodar em conjunto com o docker-compose. 

Sempre que aparecer <dockerhub-user>, você precisa substituir pelo seu usuário no DockerHub.

Entrar no host A, e instalar os pacotes abaixo, que incluem Git, Python, Pip e o Docker-compose.
```sh

$ sudo su
$ apt-get install git -y
$ curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### Instalar Rancher - Single Node

Nesse exercício iremos instalar o Rancher 2.5.7 versão single node. Isso significa que o Rancher e todos seus componentes estão em um container. 

Entrar no host A, que será usado para hospedar o Rancher Server. Iremos verficar se não tem nenhum container rodando ou parado, e depois iremos instalar o Rancher.
```sh
$ docker ps -a
$ docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher  -p 80:80 -p 443:443 --privileged rancher/rancher:latest
```
Com o Rancher já rodando, irei adicionar a entrada de cada DNS para o IP de cada máquina.

```sh
$ rancher.<dominio> = IP do host A
```

### Criar cluster Kubernetes

Nesse exercício iremos criar um cluster Kubernetes. Após criar o cluster, iremos instalar o kubectl no host A, e iremos usar para interagir com o cluster.

Seguir as instruções na aula para fazer o deployment do cluster.
Após fazer a configuração, o Rancher irá exibir um comando de docker run, para adicionar os host's.

Adicionar o host B e host C. 

Pegar o seu comando no seu rancher.
```sh
$ docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.4.3 --server https://rancher.dev-ops-ninja.com --token 8xf5r2ttrvvqcxdhwsbx9cvb7s9wgwdmgfbmzr4mt7smjbg4jgj292 --ca-checksum 61ac25d1c389b26c5c9acd98a1c167dbfb394c6c1c3019d855901704d8bae282 --node-name k8s-1 --etcd --controlplane --worker
```
Será um cluster com 3 nós.
Navegar pelo Rancher e ver os painéis e funcionalidades.

### Instalar kubectl no host A

Agora iremos instalar o kubectl, que é a CLI do kubernetes. Através do kubectl é que iremos interagir com o cluster.
```sh

$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
```

Com o kubectl instalado, pegar as credenciais de acesso no Rancher e configurar o kubectl.

```sh
$ vi ~/.kube/config
$ kubectl get nodes
```

### Traefik 

O Traefik é a aplicação que iremos usar como ingress. Ele irá ficar escutando pelas entradas de DNS que o cluster deve responder. Ele possui um dashboard de  monitoramento e com um resumo de todas as entradas que estão no cluster.
```ssh
git clone https://github.com/efcunha/Traefik-v2.4.git
cd Traefik-v2.4/

- Alterar no arquivo daemon-set.yaml
  - Linha 45: Colocar seu e-mail
- Alterar no arquivo ingress.yaml
  - Linha 13: Colocar o endereço e dominio para o Traefik
- Alterar no arquivo app-teste.yaml  
  - Linha 55: Colocar o endereço e dominio para o App-teste

$ kubectl apply -k .

$ kubectl --namespace=kube-system get pods
```
Este repositório é baseado no:

https://blog.tomarrell.com/post/traefik_v2_on_kubernetes

https://traefik.io/blog/traefik-2-2-ingress/


### Volumes

Fazer o deployment do Longhorn.

### Graylog - LOG

O Graylog é a aplicação que iremos usar como agregador de logs do cluster. Os logs dos containers podem ser vistos pelo Rancher, é um dos níveis de visualização. Pelo Graylog temos outros funcionalidades, e também é possível salvar para posterior pesquisa, e muitas outras funcionalidades.

Para instalar o Graylog, iremos aplicar o template dele, que está em graylog.yml. Para isso, é preciso que sejam editados 2 pontos no arquivo.

- Linha 239 - value: http://graylog.<dominino>/api
- Linha 312 - host: graylog.<dominio>

Substituir o {user}, pelo nome do aluno. Após substituir, aplicar e entrar no Graylog para configurar.
```sh
$ kubectl apply -f graylog.yml
```
Seguir os passos do instrutor para configuração do Graylog.

### Grafana - MONITORAMENTO

O Grafana/Prometheus é a stack que iremos usar para monitoramento. O Deployment dela será feito pelo Catálogo de Apps.
Iremos configurar seguindo as informações do instrutor, e fazer o deployment.

Será preciso altear os DNS das aplicações para que elas fiquem acessíveis.

Após o deploymnet, entrar no Grafana e Prometheus e mostrar seu funcionamento.


### Liveness

Iremos testar como fazer para dizer ao kubernetes, quando recuperar a nossa aplicação, caso alguma coisa aconteça a ela.

```js
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) { 
	duration := time.Now().Sub(started) 
	if duration.Seconds() > 10 { 
		w.WriteHeader(500) 
		w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds()))) 
	} else { 
		w.WriteHeader(200) 
		w.Write([]byte("ok")) 
	} 
})
```
O código acima, está dentro do container que iremos rodar. Nesse código, vocês podem perceber que tem um IF, que irá fazer que de vez em quando a aplicação responda dando erro. 

Como a aplicação irá retornar um erro, o serviço de liveness que iremos usar no Kubernetes, ficará verificando se a nossa aplicação está bem, e como ela irá falhar de tempos em tempos, o kubernetes irá reiniciar o nosso serviço.

```sh
$ kubectl apply -f liveness.yml 
	Depois de 10 segundos, verificamos que o container reiniciou. 
$ kubectl describe pod liveness-http 
$ kubectl get pod liveness-http 
```

### Autoscaling

Iremos executar o tutorial oficial para autoscaling.

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#before-you-begin

Para isso iremos rodar e expor o php-apache server

Desabilitar o monitoramento com prometheus e Grafana para o Autoscaling poder funcionar.


```sh
$ kubectl apply -f php-apache.yml
```

Agora iremos fazer a criação do Pod Autoscaler

```sh
$ kubectl apply -f hpa.yml
```

Iremos pegar o HPA

```sh
$ kubectl get hpa
```

### Autoscaling - Aumentar a carga

Agora iremos aumentar a carga no pod contendo o apache em php.

```sh
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
# Hit enter for command prompt
$ while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```
Agora iremos em outro terminal, com o kubectl, verificar como está o HPA, e também no painel do Rancher. 

```sh 
$ kubectl get hpa
$ kubectl get deployment php-apache
```

### HELM


```sh 
$ curl -O https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
$ helm version

$ helm repo add stable https://charts.helm.sh/stable

$ helm search hub redis

$ helm repo update

$ helm install stable/redis
```

As aplicações no catálogo do Rancher são feitas pelo Helm.

## Agradecimentos:

Este material é baseado no curso:

DevOps Ninja: Docker, Kubernetes e Rancher

https://www.udemy.com/course/devops-mao-na-massa-docker-kubernetes-rancher

https://github.com/jonathanbaraldi

# License

Copyright (c) 2014-2018 [Rancher Labs, Inc.](http://rancher.com)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
