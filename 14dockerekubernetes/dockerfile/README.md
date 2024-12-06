# Dockerfile

## Build de uma nova imagem

https://github.com/devfullcycle/mba-docker/blob/main/cap%2004%20-%20Dockerfile/01-Build%20de%20uma%20nova%20imagem/README.md

Construir imagens é uma necessidade muito comum, dado que temos várias imagens no DockerHub, imagens existentes que têm bancos de dados, linguagens de programação, outras ferramentas, vamos acabar tendo a nossa necessidade específica, principalmente, com ferramentas e linguagens de programação, de alterar aquela imagem para que o container já tenha arquivos e ferramentas necessárias para trabalharmos.

- Dockerfile: manifesto da nossa imagem.

    - É feito com instruções: FROM, RUN, COPY, EXPOSE, CMD, ENTRY POINT, etc.

- alpine é uma imagem muito pequena. Quanto menor, mais rápida para podermos carregar, ocupa menos espaço.

Construir uma nova imagem:

- docker build -t my-site .

- O parâmetro -t indica o nome da imagem que será criada. Nesse caso, o nome da imagem será meu-site. O ponto indica que o Dockerfile está no diretório atual.

Se quiser apontar para um manifesto Dockerfile que está em outro local, utilizar -f.

- docker images
REPOSITORY    TAG              IMAGE ID       CREATED          SIZE
my-site       latest           949e1fd9397b   57 seconds ago   22.6MB

- docker run --rm --name mysite1 -p 8080:80 my-site

Supondo que queiramos fazer uma alteração só para testar e não queira fazer o build da imagem novamente: copiar para o container:

- docker cp index.html mysite1:/usr/share/nginx/html

## Publicando imagens no dockerhub

https://github.com/devfullcycle/mba-docker/blob/main/cap%2004%20-%20Dockerfile/02-Publicando%20imagens%20no%20Docker%20Hub.md

Agora com a imagem no Docker Hub, poderíamos publicar o site em qualquer hospedagem que suporte Docker, nós já temos clouds que facilitam e muito o deploy de containers através das imagens.

Sempre teremos que ter imagens num container registry para realizar isto. Um container registry é um repositório de imagens, como o Docker Hub, que é um container registry público, mas podemos ter um container registry privado, como o Azure Container Registry, AWS ECR, Google Container Registry, Gitlab Container Registry, etc.

Quando estamos falando de publicação de imagens, o repositório de imagens é, normalmente, chamado de registry, images registry ou container registry. O DockerHub não é único; podemos ter, também, um repositório de imagens locais, privado.

Para autenticar o CLI do docker, devemos fazer isso pelo token:

- dockerhub.com

- logar

- Ir em: Account Settings / Security / Personal access tokens

- Generate new token

- Run

    - docker login -u sidartasilva

- At the password prompt, enter the personal access token.

    - dckr_pat_qkfmkvEbEyphIXFf9Tgkh0Qxhqc

O nome da imagem precisa ser prefixado pelo seu nome de usuário / o nome da imagem:

- docker build -t sidartasilva/mysite .

Para publicar:

- docker push sidartasilva/mysite

Ele vai fazer a publicação: pegar o que chamamos de layers dessa imagem e fazer a publicação.

Essas camadas servem para escrever cada passo que está definido no Dockerfile, vai sendo gerado uma camada com os arquivos.

Isso tem vaŕios benefícios, por exemplo, no momento de fazer o push, se já existir uma camada no dockerhub, não vai subir, essa camada pode ser bem grande, não seria produtivo estar subindo alguma coisa que poderia ter vários MB's.

Então, o docker percebe que já tem uma camada lá e não sobe, passa para a próxima e, por fim, acaba subindo toda a imagem.

Para acessar a imagem:

- Ir no profile.

- É exibido a listagem de imagens.

É aplicado compressão sobre cada imagem que sobe no dockerhub. Nesse caso, a imagem passou de 22.6MB para 9.36MB.

Para subir uma nova versão da imagem, é necessário apenas alterar a tag:

- docker build -t sidartasilva/my-site:v2 .

- docker push sidartasilva/my-site:v2

Com relação às camadas:

- docker history sidartasilva/my-site

Consegue-se ver o histórico das alterações dessa imagem, que vai acabar resultando em layers.

Image registry:

- https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages

Vamos supor que tenhamos alguma coisa publicada GitHub em que seja possível gerar uma imagem que queiramos distribuir:

- https://github.com/sidartaoss/mba-fullcycle-clean-architecture

- É possível distribuir através do GitHub sem publicar no DockerHub.

- Basta ir no link: Publish your first package. (https://github.com/sidartaoss/mba-fullcycle-clean-architecture/packages). São imagens públicas. O GitHub packages é concorrente do dockerhub.

Para imagens públicas, não é necessário pagar.

- Nos processos de CI, consegue-se muito mais velocidade para poder gerar as imagens, porque elas vão estar dentro do próprio GitHub, com GitHub Actions.

Podemos encontrar bastante projetos open source que distribuem tanto no DockerHub quanto no GitHub Packages.

### Referência
MBA ARQUITETURA FULL CYCLE. Docker & Kubernetes. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 05 dez. 2024.

