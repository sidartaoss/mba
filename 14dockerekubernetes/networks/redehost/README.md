# Rede host

- Em vez de receber um IP interno e criar um isolamento, os containers compartilham a rede do host. Esta opção é muito performática, já que não há isolamento de rede, re-encaminhamento de portas ou pontes de redes, o acesso é direto.

- É mais performático, porque não vai criar essa outra camada, que é o bridge, que vai fazer o encaminhamento, tem várias coisas que acontecem, então, acaba tornando a comunicação um pouco mais lenta.

- Então, o host vai utilizar a própria interface de rede da máquina, ele não vai fazer aquele isolamento com o namespace.

- docker run --rm -e MYSQL_ROOT_PASSWORD=root --name mysql1 --network host mysql:8.0.30-debian

Não é necessário publicar portas no docker-compose: a porta do container que está rodando ali é que vai ficar exposta.

Para desenvolvimento, às vezes, em um cenário de testes de stress, alterando do modo bridge para o modo host será mais performático.

Em Produção, também é muito utilizado para poder acelerar a comunicação entre containers.

- docker ps

- docker inspect redehost-nginx-1 | grep IPAddress

```
"SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "",
```

- Não é retornado um IP. Não temos um IP associado com uma rede, porque está utilizando o IP da própria máquina: só está alocando a porta do próprio sistema operacional; não dentro de um namespace. Então, todos os containers são acessíveis pelo localhost ou 127.0.0.1 ou pelo próprio IP da máquina.

- docker inspect redehost-nginx-1

- docker network ls

## Administrando redes

https://github.com/devfullcycle/mba-docker/blob/main/cap%2007%20-%20Networks/04-Administrando%20redes.md

Remover redes

- docker network rm primeiroexemplocomdockercompose_default

Remover redes que não estão sendo utilizadas por nenhum container (redes órfãs):

- docker network prune

- docker compose down: remove a rede padrão do docker-compose.

## Conectando containers via um endereço único

