# Traefik-v2.4

# Traefik v2.4 HA K8s

https://blog.tomarrell.com/post/traefik_v2_on_kubernetes

https://traefik.io/blog/traefik-2-2-ingress/

https://github.com/jonathanbaraldi/devops/tree/master/exercicios/traefik22

Ubuntu 21.04 LTS

Docker 20.10.6

Kubernetes 1.20.5

Rancher 2.5.7

Traefik 2.4


```sh
$ cd traefik-v2.42
# Alterar o ingress, colocar o host do seu endereço
# Alerar o email e o comentário de staging no deamon-set
$ kubectl apply -k .

# Acessar o dashboard

# trocar o ingress da Aplicação
$ kubectl apply -f app-teste.yml

```
