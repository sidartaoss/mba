# As instruções run / copy / workdir / expose / cmd - parte 1

https://github.com/devfullcycle/mba-docker/blob/main/cap%2004%20-%20Dockerfile/03-As%20instru%C3%A7%C3%B5es%20RUN%2C%20COPY%2C%20EXPOSE%20E%20CMD/README.md

Desafio:

- Pegar uma aplicação nodejs, levantar um servidor web com ela, mas, normalmente, não expomos essa aplicação diretamente: vamos colocar um servidor nginx na frente:

- Isto seria muito comum, apesar do Node.js ter um servidor web, o Nginx facilitaria configuração de cache, load balance, https e etc.

- Nginx vai se comportar como um proxy reverso, podemos colocar certas restrições, não ser exposto diretamente também é uma boa prática, consegue colocar https, etc.

- A imagem do nginx é de um linux debian. Então, utilizamos o comando apt install. Para poder executar comandos no processo de montagem de imagens, a gente pode utilizar a instrução RUN.

- docker build -t my-node .

```
FROM node:20-slim


RUN apt install nginx
```

Gerou o erro:

```
0.136 E: Unable to locate package nginx
------
Dockerfile:4
--------------------
   2 |     
   3 |     
   4 | >>> RUN apt install nginx
   5 |     
   6 |     
--------------------
ERROR: failed to solve: process "/bin/sh -c apt install nginx" did not complete successfully: exit code: 100
```

Pois essa imagem não vem atualizada com os repositórios das ferramentas do apt.

Então, nós temos que rodar um `apt update` antes:

```
FROM node:20-slim

RUN apt update && apt install nginx -y
```

Precisamos passar o arquivo de configuração nginx.conf para o nginx entender que tem que se comportar como um proxy reverso:

- Então, a próxima instrução é um COPY:

```
FROM node:20-slim

RUN apt update && apt install nginx -y

COPY nginx.conf /etc/nginx/nginx.conf
```

Agora, precisamos copiar a aplicação node.js, ou seja: index.js e package.json. Copiar para onde, uma vez que temos uma userland dentro da imagem, mas aonde colocar isso?

Poderia colocar em /usr/src/app: é um local comum de se utilizar.

Quando temos uma aplicação, normalmente, é recomendável definir um diretório de trabalho, para deixar claro aonde essa aplicação vai trabalhar:

- WORKDIR /usr/src/app

Então, em <dest> da instrução COPY, podemos definir simplesmente ./, representando WORKDIR.

Em seguida, podemos fazer a geração da pasta nodemodules:

- RUN npm install

```
FROM node:20-slim

RUN apt update && apt install nginx -y

COPY nginx.conf /etc/nginx/nginx.conf

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install
```

O comando npm install vai ser gerado dentro de WORKDIR, gerando a pasta nodemodules.

Falta apenas um comando: para rodar a aplicação no container: CMD. CMD é a instrução que vai informar ao docker qual é o comando inicial: quando o container iniciar, ele vai executar esse comando:

- CMD ["node", "/usr/src/app/index.js"]

Esse comando vai manter o container de pé.

Para algumas imagens, não há a conexão com todos os recursos no terminal: o CTRL+C não funciona. Então, podemos passar outra instrução: o --init, aí, o CTRL+C vai funcionar:

- docker run --init --rm --name my-node1 my-node

Então, o init habilita os recursos de conexão com o terminal.

No build, nós temos 6 layers, porque nós temos 6 comandos:

```
FROM node:20-slim

RUN apt update && apt install nginx -y

COPY nginx.conf /etc/nginx/nginx.conf

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

CMD ["node", "/usr/src/app/index.js"]
```
Falta rodarmos com nginx, para ele rodar como servidor e não o node.js.

```
FROM node:20-slim

RUN apt update && apt install nginx -y

COPY nginx.conf /etc/nginx/nginx.conf

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

CMD [ "/bin/sh", "-c", "node /usr/src/app/index.js & nginx -g 'daemon off;'" ]
```

Comando para rodar:

- docker run --init --rm --name my-node1 -p 8080:80 my-node

```
Example app listening on port 3000
```

Apesar de ter mostrado que está escutando na porta 3000, a 3000 não está exposta; somente o nginx na porta 80 está se comportando como um proxy reverso.

Instruções:

- RUN: executa comandos.

- COPY: copia qualquer tipo de arquivo para qualquer lugar dentro do container.

- WORKDIR: delimita o diretório de trabalho e facilita colocarmos coisas em um determinado lugar.

- CMD: poder iniciar o container.

Se formos executar: 

- docker exec -it my-node1 bash

```
root@8fb34aeeeb7f:/usr/src/app#
```

Vemos que o diretório setado para podermos trabalhar é o WORKDIR. Quando se acessa esse container, ele já vai diretamente para lá.

## As instruções run / copy / workdir / expose / cmd - parte 2

WORKDIR delimita qual é o diretório de trabalho: o que a gente fizer de COPY, RUN, vai ter sempre incidência na WORKDIR.

Poderia não delimitar a WORKDIR. Algumas imagens, às vezes, determinam o WORKDIR, outras não. Normalmente, imagens de linguagem de programação permitem que definamos: PHP, Node.js, Java, etc.

A WORKDIR delimita a própria raiz da imagem. Apesar de não ser obrigatória, ela ajuda-nos a organizar o que queremos fazer na imagem.

Então, WORKDIR define o lugar onde os arquivos serão colocados e acaba beneficiando quando se executa o comando docker exec, também, porque já entramos no WORKDIR.

Quando não definimos uma port exposta no Dockerfile para a imagem, ao rodar um comando docker container ls, por exemplo, não é exibido a porta na coluna PORTS:

- É uma coisa que, às vezes, é ruim: pegar uma imagem de alguém e não saber qual que é a porta. 

    - Então, nós já podemos definir algo no Dockerfile que define qual que é a porta que o container será exposto.

    - Então, nós usamos, geralmente, no final do Dockerfile, uma instrução EXPOSE: essa informação não publica porta nenhuma: ela é apenas documental: é uma forma de formalizar qual a porta que desejamos que seja exposta.

## Inspecionando imagens e containers

https://github.com/devfullcycle/mba-docker/blob/main/cap%2004%20-%20Dockerfile/04-Inspecionando%20imagens%20e%20containers.md

Inspeção de uma imagem de um container.

- docker inspect my-node

Para podermos visualizar a descrição da imagem, conforme o padrão OCI, que trabalha com runc ou com qualquer outro container runtime que está no baixo nível fazendo a execução desses containers.

Também é possível inspecionar um container:

- docker ps

- docker inspect 6b5

Então, são dois JSON's diferentes: uma coisa é a especificação da imagem e outra especificação do container.

O container também é dividido em camadas.

- docker inspect my-node1 | grep IPAddress

```
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",

```

Têm ferramentas, também, como o jq: https://jqlang.github.io/jq/, aonde é possível fazer pesquisa em JSON.

## Os layers de imagens

https://github.com/devfullcycle/mba-docker/blob/main/cap%2004%20-%20Dockerfile/05-Os%20layers%20de%20imagens.md

Toda instrução no Dockerfile acaba gerando camadas.

- Camadas nada mais são do que sub-imagens, que vão ter um hash atribuído para o nome da pasta, que é um sha-256. 

- Ali dentro, vamos ter o armazenamento dos arquivos e pastas que foram modificados naquela instrução.

Toda instrução no arquivo Dockerfile acaba gerando camadas.

Então, se, por exemplo, for um run para instalar nginx, vamos ter uma sub-imagem com todos os arquivos do nginx que foram instalados.

Se, por exemplo, for um copy, então, vai gerar uma nova sub-imagem com os arquivos. E sempre essas sub-imagens são apenas para leitura.

Então, a junção dessas sub-imagens vai formar a imagem que, depois, quando o docker gerar um container a partir dessa imagem, ele também vai gerar esse container que vai ser subdividido nessas camadas para poder controlar também o armazenamento das coisas do container.

Inclusive, tem como pegar as modificações de um container e gerar uma nova imagem a partir dele com o comando docker commit. E, aí, ele vai gerar novas camadas de acordo com as alterações que foram feitas.

Então, tudo em imagens fuciona em camadas.

Algumas instruções no Dockerfile podem não gerar camadas, como EXPOSE e CMD.

Conseguimos ver as camadas de uma imagem a partir de um backup:

- docker save my-node > image.tar

- tar -xvf ./imagenode.tar -C ./imagenode

Para ver as imagens geradas:

- sudo ls -la /var/lib/docker/overlay2

Cada uma dessas pastas representa a organização dos layers das imagens.

Como encontrar os layers da nossa imagem (my-node)?

- docker inspect my-node

Em GraphDriver / Lowdir.

Vantagens das layers:

- Otimizam o armazenamento no disco.

- Otimizam também a montagem das imagens tanto em questão do container quanto a criação dessa imagem, fazendo com que ela seja criada mais rapidamente.

- Reaproveitamento entre imagens diferentes, fazendo com que, quando for feito um push, já hajam essas camadas que o docker organiza com os hashes lá no image registry e não precisa subir.

## As instruções cmd e entrypoint

https://github.com/devfullcycle/mba-docker/blob/main/cap%2004%20-%20Dockerfile/06-As%20instru%C3%A7%C3%B5es%20CMD%20e%20ENTRYPOINT.md




### Referência
MBA ARQUITETURA FULL CYCLE. Docker & Kubernetes. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 05 dez. 2024.

