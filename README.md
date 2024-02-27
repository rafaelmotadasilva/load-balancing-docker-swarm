# Cluster Swarm Load Balancer com NGINX

## Configurar o cluster Docker Swarm

Primeiro, é necessário configurar o cluster Swarm para dois dos três nós. Para fazer isso, execute o seguinte comando para iniciar o cluster Swarm:

```
sudo docker swarm init --advertise-addr ip-master
```
Você obterá a seguinte saída:

```
Swarm initialized: current node (4uozaria1motuuuz66n9jeeys) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1bu8smhaerewus9mn3m821zxqvtnd8mq82m9ppnv83wxijvhnk-0yuk9yx31ch7nhll6s1z6khyo ip-master:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

## Adicionar o Node Worker ao cluster Docker Swarm

Em seguida, é necessário adicionar o Node Worker ao cluster Swarm. Execute o seguinte comando:

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

Em seguida, implante o serviço Nginx no Node Master e escale-o entre os dois nós. Vá para o Node Master e execute o seguinte comando para criar um serviço Nginx:

```
sudo docker service create --name backend --replicas 2 --publish 8080:80 nginx
```
Você deverá ver a seguinte saída:

```
b9mvrug6d7qz3q3r1tl585adz
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

Este comando criará um contêiner Nginx e permitirá conexões com os serviços web hospedados pelo seu Docker Swarm. Em seguida, abra seu navegador e verifique o Load Balancing usando a URL http://ip-load-balancer. Você deverá ver a página do Nginx.
