# Conectando containers via um endereço único

https://github.com/devfullcycle/mba-docker/tree/main/cap%2007%20-%20Networks/05%20-%20Conectando%20containers%20via%20um%20endere%C3%A7o%20%C3%BAnico

Vai acontecer muito, tanto em desenvolvimento quanto produção, de termos dois docker-composes rodando. Então, nós temos dois containers que estão em redes diferentes, se não estiverem, obviamente, na host; se houverem dez docker-composes e todos estiverem rodando no modo da rede host, está todo mundo conseguindo se comunicar.

Mas, acontece que, muitas vezes, não trabalhamos com o modo host, trabalhamos com o modo bridge (padrão) e, aí, nós temos que criar uma rede entre esses containers de docker-compose's separados para que eles possam fazer a comunicação.

Isso pode-se tornar burocrático para gerenciar.

Então, vamos ver um recurso muito comum, que a comunidade docker utiliza, que consiste em conectar os containers via um endereço único.

Quando temos dois docker-compose's distintos que não trabalham com modo de rede host, será criado uma rede bridge distinta em cada docker-compose e os containers não vão conseguir se comunicar.

Nesse caso, temos 2 opções para trabalhar:

- Criar uma rede externa, habilitando em ambos os docker-compose's:

    - docker network create my-network.

    - Fazendo com que o docker-compose não crie uma rede padrão.

```
network:
  my-network:
    external: true
```

- Permitir um dos docker-compose's criar uma rede padrão (nome_da_pasta_default):

```
network:
  conectandocontainersviaenderecounico_default:
    external: true
```

Desvantagem:

- Pode ficar burocrático, complexo e não produtivo ter que lidar com o gerenciamento de rede, caso tenhamos uma malha maior de docker-compose's para gerenciar.

Vantagem:

Por conta disso, existe uma forma de criar um endereço único para fazer com que a comunicação fique mais simples entre containers de diferentes docker-compose's.

- 1. Vamos criar um endereço, o qual pode ter qualquer nome:

    - host.docker.internal

- 2. Vamos colocar dentro do /etc/hosts de cada container o host-gateway, ou seja, o endereço do docker que, quase sempre, é 172.17.0.1, o qual vai equivaler ao host.docker.internal:

```
172.17.0.1      host.docker.internal
```

- Se o container node.js precisar se comunicar com o mysql, que está em um outro docker-compose, vamos acessar dessa forma: `host.docker.internal:3306`.

- Quando acessar o endereço `host.docker.internal`, vai direcionar para o gateway do docker 172.17.0.1, que vai acabar batendo na nossa máquina. 

- Vamos ter o /etc/hosts em nossa máquina, que vai ter:

```
127.0.0.1      host.docker.internal
```

- Então, quando precisar fazer a comunicação para o mysql, vai sair da rede do docker (172.17.0.1), vai bater na máquina local, redirecionando para o próprio localhost (127.0.0.1), na porta 3306. Ou seja, ele saiu da rede do docker, bateu na máquina local e foi para dentro do outro container que está com a porta exposta.

- Editar o arquivo /etc/hosts, adicionando:

```
127.0.0.1      host.docker.internal
```

- docker compose -f docker-compose-mysql.yaml up

- docker compose -f docker-compose-mysql.yaml exec db bash

- cat /etc/hosts

```
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.1      host.docker.internal
172.22.0.2      4f4fa80f1147
```

- docker compose -f docker-compose-node.yaml exec app bash

- apt update && apt install iputils-ping

- ping host.docker.internal

```
PING host.docker.internal (172.17.0.1) 56(84) bytes of data.
64 bytes from host.docker.internal (172.17.0.1): icmp_seq=1 ttl=64 time=0.140 ms
64 bytes from host.docker.internal (172.17.0.1): icmp_seq=2 ttl=64 time=0.049 ms
64 bytes from host.docker.internal (172.17.0.1): icmp_seq=3 ttl=64 time=0.066 ms
```

Dessa forma, podemos fazer essa comunicação entre containers que estão em redes totalmente diferentes, através desse endereço único:

```
    extra_hosts:
       - "host.docker.internal:host-gateway"
```

E teremos ferramentas, como o RabbitMQ, que está rodando separado, ou Apache Kafka, Redis e, aí, é muito melhor usar esse endereço: host.docker.internal, do que usar uma rede, principalmente, para desenvolver, se torna um pouco burocrático.

