# Primeiro contato com o docker

## Instalação nos sistemas operacionais

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/01-Instala%C3%A7%C3%A3o%20nos%20sistemas%20operacionais.md

## Integração com vscode e outras ide's

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/02-Integra%C3%A7%C3%A3o%20com%20VSCode%20e%20outras%20IDE.md

## Rodando primeiro container com docker

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/03-Rodando%20primeiro%20container%20com%20Docker.md

Comandos:

- docker container ls: mostrar apenas os containers ativos

- docker ps: mostrar apenas os containers ativos

- docker container ls -a: mostrar todos os containers, inclusive os não ativos

- docker ps -a: mostrar todos os containers, inclusive os não ativos

Por que aparecem vários containers? Porque, cada vez que está executando um docker run, um novo container é criado, o container executa, morre e ele fica registrado.

Fazer uma limitação de recursos (memória e cpu): 50% de CPU e 512 de memória:

docker run --cpus 0.5 --memory 512m nginx:1.19.10-alpine

## Container versus images

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/04-Containers%20vs%20imagens.md

Vamos criar um container a partir da imagem node:20-slim, que é uma imagem do Node.js com o sistema operacional Debian. Para criar o container, execute o comando abaixo:

``
docker run -it node1 node:20-slim bash
``

Se passar o bash diretamente, não vai dar certo, porque é necessário informar que o container vai se comportar como bash e ele tem que conectar com o meu bash atual.

Então, para poder ativar o bash, é necessário passar, antes, o -it, que é o modo iterativo.

Cada container é um processo independente, se fizer um echo 'zzz' > /tmp/outro-file.txt, tem o arquivo nesse container, mas, no outro container, esse arquivo não aparece, pois os containers estão em namespaces distintos.

Criamos um ponto de montagem com diretórios Linux para os dois processos: eles estão em namespaces distintos.

Imagem é só para leitura: ela é imutável. Uma vez que o node:20-slim está na máquina, nós temos a userland com os binários e algumas outras coisas e é o que tem ali. Então, todo container que for criado, vai ser criado baseando-se nesse molde. Se tivermos outro container que fez outra modificação, a modificação vai ficar somente nesse containe. Não existe nenhum compartilhamento entre eles.

Podemos comparar uma imagem com uma classe e um container com um objeto, que é uma instância daquela classe.

## Administrando imagens e containers

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/05-Administrando%20imagens%20e%20containers.md


- Opção -d: detached: permite desbloquear o terminal, para poder continuar executando os comandos. Retorna o id do container que está executando, para ser possível administrar a partir desse id.

- Exemplo: docker run -d nginx:1.19.10-alpine

Como parar um container?

- docker stop, passando o id do container ou o nome do container. Não é necessário passar o id inteiro, pode passar os 3 primeiros caracteres

- Exemplo: docker stop 5b9

Container é algo efêmero: deve ser possível excluir a qualquer momento. Se trabalhar com persistência de dados, é necessário utilizar volumes para armazenar esses dados.

Como remover os containers?

- docker rm, passando o id, o nome ou os 3 primeiros caracteres do container.

Exemplo:

- docker ps -a

- docker rm 5b9

- docker ps -q vai retornar a lista de ids dos containers.

- Parar todos os containers: docker stop $(docker ps -q)

Pode-se remover todos os containers seguindo o mesmo princípio: forçar a remoção, passando a listagem de ids de containers ativos e inativos:

- docker rm -f $(docker ps -q -a)

- docker container prune: para limpar todos os containers. É um comando mais eficiente. Pode-se utilizar um filtro, baseado no número de horas: docker container prune --filter "until=200h"

Remover imagem:

- docker rmi, passando o id da imagem ou o seu nome.

Remover imagens:

- docker rmi -f $(docker images -q -a)

Pode-se utilizar o prune, mas ele não remove todas as imagens: remove as dangling images, que são layers intermediárias para poder construir a imagem.

- docker image prune

Para fazer uma limpeza geral do docker, pode-se utilizar o system prune: vai excluir networks, volumes e outras coisas:

- docker system prune

Ficar executando container e excluindo é muito ruim, então, se fizer docker run --rm [imagem], significa que, quando sair ou o container terminar de executar, o container já vai ser excluído automaticamente:

- docker run --rm hello-world

- docker run --rm -d nginx:1.19.10-alpine
- 4f31a254a312fb2e531ee5a3c8c2eeb852669da18a7b0c8e22ce9d7275a7718f

- docker stop 4f

- docker ps -a

Nomear o container:

- docker run --rm -d --name nginx1 nginx:1.19.10-alpine

- docker rm nginx1

- docker stop nginx1

Como está rodando com detached, não sabemos o que está rodando por detrás no container. Existe um comando para vermos os logs de o que está acontecendo com o container, passando o nome ou o id:

- docker logs nginx1

## Manipulando terminais dos containers

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/06-Manipulando%20terminais%20dos%20containers.md

- docker run --rm -d --name nginx2 nginx:1.19.10-alpine

Como entrar no terminal desse container?

O parâmetro -it executa o container em modo interativo e abre o terminal do container.

- docker run -it --rm --name nginx3 nginx:1.19.10-alpine sh

- ps aux

ps aux exibe uma visão geral de todos os processos que estão em execução.

Acessar os logs:

- docker logs nginx3

Cenário onde quer executar o nginx, deixar ele rodando lá, mas entrar no container e executar comandos:

- docker run -it -d --rm --name nginx4 nginx:1.19.10-alpine

- docker exec -it nginx4 sh

Qual o usuário que está sendo utilizado dentro do container?

- whoami

- root

Estando como root, pode-se rodar o apk, que é o apt do alpine.

- apk update

- apk add git

Com o run, acabamos desvirtuando algum processo inicial que seria executado ali, mas, às vezes, queremos entrar no container para fazer alguma coisa. Mas, normalmente, é mais executado o docker exec. Devemos saber ambas as formas.

Vamos executar o node:

- docker run -it -d --rm --name node1 node:20-slim bash

- docker exec -it node1 bash

Entramos no node. Qual é o usuário que está sendo utilizado no momento?

- whoami

É o usuário root.

Habilitar algumas coisas no container para poder trabalhar:

- apt update

- apt install curl

Se eu sou root, não tem sudo. Isso não é comum, é muito raro um container ter sudo. 

Mas vai acontecer situações em que o usuário corrente do seu container não é o root.

Então, você não conseguiria rodar um apt, porque você não tem sudo ali, você teria que logar como root.

Vamos fazer uma mudança de usuário.

Primeiro, vamos entrar dentro do container:

- docker exec -it node1 bash

Ver os usuários disponíveis no container.

- cat /etc/passwd

Há um usuário 1000 no container (node) que equivale ao meu usuário 1000 aqui (user). O id do meu usuário é 1000:

- id -u

Então, se quiser logar no terminal como esse usuário (de id 1000):

- docker exec -it -u node node1 bash

- whoami

- node

Então, se tentarmos fazer um instal de git, ele vai acusar um permission deny.

Com docker run, você desvirtua o comando principal do container, ou exec depois que o container já está rodando.

## Publicando portas

https://github.com/devfullcycle/mba-docker/blob/main/cap%2003%20-%20Primeiro%20contato%20com%20Docker/07-Publicando%20portas.md

Para rodar a aplicação nginx, é necessário fazer a publicação de portas.

- docker rm node1 -f

- docker run -it -d --rm name nginx3 nginx:1.19.10-alpine

- docker exec -it nginx3 sh

- docker stop nginx3

- docker run -it -d --rm name -p 8000:80 nginx3 nginx:1.19.10-alpine

Quando acessa na porta 8000, nós temos um binding de interface de rede para dentro do namespace.

- docker run -it -d --rm name -p 8001:80 nginx3 nginx:1.19.10-alpine

A porta 80 já não está sendo usada?

Lembre-se: cada container está rodando em um namespace, ele está em uma jaula.

Então, ele tem todas as portas disponíveis.

Vantagem:

- Quando que seria possível rodar 2 servidores nginx em uma mesma máquina local? (Com essa mesma facilidade que os containers nos oferecem?)

- 8000:80: à esquerda dos dois pontos (:) é a nossa máquina, que a gente chama de host e, do lado direito, o container.

Vamos entrar no nginx:

- docker exec -it nginx sh

- ls /usr/share/nginx/html

É aqui que está o index.html.

Se quiser fazer uma modificação nele:

- apk update

- apk add vim

- vim /usr/share/nginx/html/index.html

- A imagem é imutável; o container é mutável.

Deve-se entender o conceito de namespaces: saber que nós temos um submundo com essas portas lá. Também, da mesma forma, dentro do mesmo namespace, não é possível ter dois processos rodando na mesma porta. Se tentar rodar nginx novamente, vai falar que o endereço já está sendo utilizado:

- nginx

    - Address in use

    - still could not bind

Essa publicação de portas é muito comum, podemos rodar um banco de dados:

- docker run --rm --name mysql1 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:8.0.30-debian

### Referência
MBA ARQUITETURA FULL CYCLE. Docker & Kubernetes. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 05 dez. 2024.


