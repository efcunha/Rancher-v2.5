# Traefik-v2.4.x

# Traefik v2.4.x HA Rancher K8s

Ubuntu 21.04 LTS

Docker 20.10.6

Kubernetes 1.20.5

Rancher 2.5.7

Traefik 2.4.x


```sh
$ cd traefik-v2.4
# Alterar o ingress, colocar o host do seu endereço
# Alerar o email e o comentário de staging no deamon-set
$ kubectl apply -k .

# Acessar o dashboard

# trocar o ingress da Aplicação
$ kubectl apply -f app-teste.yml

```

## Agradecimentos:

Este repositório é baseado no:

https://blog.tomarrell.com/post/traefik_v2_on_kubernetes

https://traefik.io/blog/traefik-2-2-ingress/

E tambem no curso:

https://github.com/jonathanbaraldi/devops/tree/master/exercicios/traefik22

https://github.com/jonathanbaraldi

Todo méritos a eles que fizeram e atualizaram este repositório. 

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
