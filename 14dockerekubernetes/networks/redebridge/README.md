# Rede bridge

- É o tipo de network padrão. 

- Cria um link entre os containers permitindo a comunicação encaminhando o tráfego entre os segmentos de rede. Os containers ficam isolados de outros containers e podem se comunicar com o host através do gateway da rede. Todos os containers recebem um IP interno da rede. Eles também podem se comunicar internamente através dos seus IPs.

- docker network create my-network

- docker network rm my-network

Habilitar um container em uma rede:

- docker run --rm -e MYSQL_ROOT_PASSWORD=root --name mysql1 --network=my-network mysql:8.0.30-debian

- docker inspect mysql1

- docker inspect mysql1 | grep IPAddress

```
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.21.0.2",

```

Rodar uma aplicação node.js:

- docker run --init -it --rm --name node1 --network=my-network node:20-slim bash

- docker inspect node1

- docker inspect node1 | grep IPAddress

```
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.21.0.3",

```

- apt update && apt install iputils-ping

- ping mysql1

Está conseguindo pingar (se comunicar com) o mysql.

Cada container é como se fosse uma máquina. Então, localhost não sai do container. Para acessar de fora do container, deve-se fazer a chamada pelo nome do container.

No docker-compose.yaml, define-se external para uma rede que foi criada de forma manual, ou seja, não está sendo controlada diretamente pelo docker-compose.

Um container pode fazer parte de várias redes.

Quando estamos trabalhando com uma rede, habilitando no container e todos os containers precisam se comunicar, então, todos os containers precisarão estar na mesma rede.

Não é necessário definir dentro do docker-compose uma rede bridge, pois já é criado uma rede por padrão pelo docker-compose que já é uma rede que, por padrão, já fica compartilhada entre os containers, ou seja os containers já ficam na mesma rede: então, eles vão conseguir se comunicar.

Essa rede é destruída sempre que for executado o comando `docker compose down`.

```
networks:
  my-network:
    external: true
```

Qual a vantagem de criar uma rede bridge de forma manual?

- Se estamos rodando uma outra leva de containers com outro docker-compose, tem uma outra rede bridge isolada dessa que está rodando no momento. Então, esses containers não conseguem se comunicar.

- Para que se consiga fazer essa comunicação, aí é necessário criar uma rede entre eles. Então, poderia usar uma rede externa para fazer essa comunicação ou uma rede já criada automaticamente com o docker-compose e jogar no outro docker-compose.yaml como external:

```
networks:
  redebridge_default:
    external: true
```

- Assim, é possível fazer a comunicação entre docker-compose's diferentes.


