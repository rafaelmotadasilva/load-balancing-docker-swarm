# Configuração do Cluster Docker Swarm com Load Balancer NGINX

O Docker Swarm, junto com um balanceador de carga NGINX, oferece uma solução poderosa para gerenciar aplicativos em contêineres em ambientes distribuídos. Este guia apresenta as etapas para configurar e implantar um cluster Docker Swarm, além de configurar um balanceador de carga NGINX para distribuir o tráfego entre os nós do cluster.

## Visão Geral

Este guia aborda o processo de configuração de um cluster Docker Swarm com um balanceador de carga NGINX. Essa configuração permite distribuir o tráfego entre os nós do cluster, melhorando a disponibilidade e a escalabilidade de aplicativos hospedados no Docker Swarm.

## Requisitos

"Antes de iniciar, é fundamental ter acesso a três nós: um para atuar como nó mestre (master) e dois para servir como nós de trabalho (workers). Certifique-se de que esses nós estejam configurados corretamente e tenham o Docker instalado."

## Instruções

1. [Configurando o Cluster Docker Swarm](#configurando-o-cluster-docker-swarm)
2. [Adicionar o Node Worker ao cluster Docker Swarm](#adicionar-o-node-worker-ao-cluster-docker-swarm)
3. [Implantar o Serviço Nginx ao Cluster Docker Swarm](#implantar-o-serviço-nginx-ao-cluster-docker-swarm)
4. [Configurar o Load Balancer](#configurar-o-load-balancer)
5. [Conclusão](#conclusão)

## Configurando o Cluster Docker Swarm

Para iniciar, é necessário configurar o cluster Swarm em dois dos três nós. Execute o comando a seguir para iniciar o cluster Swarm:

```
sudo docker swarm init --advertise-addr ip-master
```

Isso resultará em uma saída informando como adicionar nós ao cluster.

## Adicionar o Node Worker ao cluster Docker Swarm

Para adicionar um Node Worker ao cluster Swarm, execute o seguinte comando:

```
sudo docker swarm join --token SWMTKN-1-1bu8smhaerewus9mn3m821zxqvtnd8mq82m9ppnv83wxijvhnk-0yuk9yx31ch7nhll6s1z6khyo ip-master:2377
```

Após adicionar o Node Worker ao cluster Swarm, você pode verificá-lo com o seguinte comando no Node Master:

```
sudo docker node ls
```

Você deve obter algo assim:

```
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
4uozaria1motuuuz66n9jeeys *   master     Ready     Active         Leader           25.0.3
vjlhqtrcc814gspukhzcsas8k     node01     Ready     Active                          25.0.3
```

## Implantar o Serviço Nginx ao Cluster Docker Swarm

Em seguida, implante o serviço Nginx no Node Master e escale-o entre os dois nós. Execute o seguinte comando para criar um serviço Nginx:

```
sudo docker service create --name backend --replicas 2 --publish 8080:80 nginx
```

Você deverá ver a seguinte saída:

```b9mvrug6d7qz3q3r1tl585adz
overall progress: 2 out of 2 tasks 
1/2: running   [==================================================>] 
2/2: running   [==================================================>] 
verify: Service converged
```

Em seguida, verifique seu serviço Nginx usando o seguinte comando:

```
sudo docker service ls
```

Você deve obter algo assim:

```
ID             NAME      MODE         REPLICAS   IMAGE          PORTS
b9mvrug6d7qz   backend   replicated   2/2        nginx:latest   *:8080->80/tcp
```

## Configurar o Load Balancer

Para configurar o Load Balancer, inicialize um novo cluster Swarm no Node do Load Balancer com o seguinte comando:

```
sudo docker swarm init --advertise-addr ip-load-balancer
```
Em seguida, crie o diretório para o Load Balancer com o seguinte comando:

```
sudo mkdir -p /data/loadbalancer
```

Crie um arquivo de configuração com o seguinte comando:

```
vim /data/loadbalancer/default.conf
```

Adicione as seguintes configurações:

```
server {
   listen 80;
   location / {
      proxy_pass http://backend;
   }
}
upstream backend {
   server ip-master:8080;
   server ip-node01:8080;
}
```

Salve e feche o arquivo, crie o contêiner do Load Balancer e publique-o na porta 80.

```
sudo docker service create --name loadbalancer --mount type=bind,source=/data/loadbalancer,target=/etc/nginx/conf.d --publish 80:80 nginx
```

Você deverá ver a seguinte saída:

```
n73lsh74v6bc7d4zub0czl5mq
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```

Este comando criará um contêiner Nginx e permitirá conexões com os serviços web hospedados pelo seu Docker Swarm.

Agora, abra seu navegador e verifique o Load Balancing utilizando a URL http://ip-loadbalancer. Você deverá visualizar a página do Nginx.

## Conclusão

Parabéns! Você montou seu cluster Docker Swarm com um balanceador de carga NGINX. Agora, seu sistema está pronto para lidar com o tráfego de forma eficiente entre os nós do cluster.

## Contribuição

Se você tiver sugestões de melhorias ou correções para este guia, sinta-se à vontade para enviar uma pull request.

## Referências

- [Cloud Infrastructure Services - How to setup Docker Swarm load balancing using NGINX on Ubuntu 20.04](https://cloudinfrastructureservices.co.uk/how-to-setup-docker-swarm-load-balancing-using-nginx-on-ubuntu-20-04/)
- [UpCloud - Load balancing Docker Swarm mode](https://upcloud.com/resources/tutorials/load-balancing-docker-swarm-mode)

## Licença

Este projeto está licenciado sob a [Licença MIT](LICENSE).
