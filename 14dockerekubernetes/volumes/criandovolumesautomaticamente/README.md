# Criando volumes automaticamente

https://github.com/devfullcycle/mba-docker/tree/main/cap%2006%20-%20Volumes/03%20-%20Criando%20volumes%20automaticamente

- docker run --rm -e MYSQL_ROOT_PASSWORD=root --name mysql1 -v my-volume:/var/lib/mysql mysql:8.0.30-debian

Volume não é mais external. Neste caso, é local:

```
version: '3'

services:
  db:
    image: mysql:8.0.30-debian
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - ./dbdata:/var/lib/mysql
  
  nginx:
    image: nginx:1.19.10-alpine
    volumes:
      - my-volume-nginx:/usr/share/nginx/html

volumes:
  my-volume-nginx:
    external: true
```

Se tem o volume não tiver ponto-barra (./), quer dizer que trata-se de um volume totalmente gerenciado pelo docker.

Se tem o ./ na frente do volume, quer dizer que está sendo o feito o armazenamento local no projeto.

