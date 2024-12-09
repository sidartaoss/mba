# Criando volumes pelo volume create

https://github.com/devfullcycle/mba-docker/blob/main/cap%2006%20-%20Volumes/02%20-%20Criando%20volumes%20pelo%20volume%20create/README.md

Como criar um volume zerado:

- docker volume create my-volume

- docker volume ls

```
DRIVER    VOLUME NAME
local     4de543a400321be08aaf4cbf57daf64dd3c1f37503e12149733abab2304d1134
local     51ad632dd379e45872fd1e1badc04711afdf17af34d328a203288ed0bc84f3c6
local     120afba4995335d742e46480247f69d062498216bb1ee1a7cb7fae89e6cc3a66
local     df05f2edb369d5e2378dc433ea0a09e783141ff895d12eff226a07e8e29204ed
local     my-volume
```

O volume vai ficar na nossa máquina e vai ser montado no container.

- sudo ls /var/lib/docker/volumes/my-volume

```
_data
```

Checar o conteúdo do volume:

- sudo ls /var/lib/docker/volumes/my-volume/_data

Como usar o volume para não perder os dados do banco de dados?

- docker run --rm -e MYSQL_ROOT_PASSWORD=root --name mysql1 -v my-volume:/var/lib/mysql mysql:8.0.30-debian

Para onde o volume vai ser montado dentro do container? (É o caminho fixo que o mysql vai fazer o armazenamento de dados.)

- :/var/lib/mysql

Se matar o container do mysql, os dados do volume continuam lá.

Se criar um outro container, apontando para o volume my-volume, no momento de o container iniciar, ele percebe que o banco de dados já existe e não vai recriar. Então, não se perde os dados.

- docker volume inspect my-volume 

```
[
    {
        "CreatedAt": "2024-12-09T17:34:06-03:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-volume/_data",
        "Name": "my-volume",
        "Options": null,
        "Scope": "local"
    }
]
```

Docker-compose

```
volumes:
  my-volume:
    external: true
```

- Significa que esse volume está vindo de uma configuração externa.

Volume seria bem útil para um servidor web: os arquivos estáticos do site poderiam ficar dentro de um volume.

- docker volume create my-volume-nginx

- docker compose exec nginx sh

- apk update && apk add vim

- vim /usr/share/nginx/html/index.html

Se editar no volume, é editado automaticamente no container.

- sudo cat sudo /var/lib/docker/volumes/my-volume-nginx/_data/index.html

O Docker trabalha com padrões: cached, delegated e consistente (padrão).

No modo consistente, se escrever algo no volume, já está disponível no container e vice-versa: se escreve algo no container, já está disponível no volume; existe esse sincronismo e vai garantir, às vezes, que se tenha mais performance, porque, muitas vezes, não é necessário ter o dado na máquina na hora: pode escrever no container e, depois, vai se sincronizando.