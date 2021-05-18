
# Rancher Traefik v2.2.x Kubernetes K8s - Single Node

Ubuntu 21.04 LTS

Docker 20.10.6

Kubernetes 1.21.1

Rancher 2.5.x

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

$ sudo su
$ curl https://releases.rancher.com/install-docker/20.10.sh | sh
$ usermod -aG docker ubuntu
```
# Instalar rodar docker-compose

### Fazer build das imagens

Para construir suas imagens dos containers, colocar elas para rodar em conjunto com o docker-compose. 

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
$ docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher  -p 80:80 -p 443:443 --privileged rancher/rancher:stable
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
$ docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.8 --server https://rancher.dev-ops-ninja.com --token 8xf5r2ttrvvqcxdhwsbx9cvb7s9wgwdmgfbmzr4mt7smjbg4jgj292 --ca-checksum 61ac25d1c389b26c5c9acd98a1c167dbfb394c6c1c3019d855901704d8bae282 --node-name k8s-1 --etcd --controlplane --worker
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
git clone https://github.com/efcunha/Traefik-v2.2.git
cd Traefik-v2.2/

- Alterar no arquivo daemon-set.yaml

  - Linha 69: Colocar seu e-mail

  - Habilitar a linha abaixo no arquivo daemon-set.yaml somente quando for gerar certificado para produção.

   "- --certificatesresolvers.default.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"

- Alterar no arquivo ingress.yaml

  - Linha 12: Colocar o endereço e dominio para o Traefik

- Alterar no arquivo app-teste.yaml  
  
  - Linha 55: Colocar o endereço e dominio para o App-teste
```
```ssh
$ kubectl apply -k .

$ kubectl apply -f daemon-set.yaml

$ kubectl --namespace=kube-system get pods
```
Este repositório é baseado no:

https://blog.tomarrell.com/post/traefik_v2_on_kubernetes

https://traefik.io/blog/traefik-2-2-ingress/


### Volumes

Fazer o deployment do Longhorn.

## Agradecimentos:

Este material é baseado no curso:

DevOps Ninja: Docker, Kubernetes e Rancher

https://www.udemy.com/course/devops-mao-na-massa-docker-kubernetes-rancher

https://github.com/jonathanbaraldi

Digo de passagem um ótimo curso, recomendo que se tiver oportunidade faça, pois a parte dos extras é SHOW de bola.

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
