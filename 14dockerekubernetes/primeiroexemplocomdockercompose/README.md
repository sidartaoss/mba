# Primeiro exemplo com docker compose

https://github.com/devfullcycle/mba-docker/blob/main/cap%2005%20-%20Docker%20Compose/03%20-%20Primeiro%20exemplo%20com%20Docker%20Compose/README.md

## Comunicação entre os containers

## Limitando uso de recursos dos containers

É possível limitar o uso de memória e cpu:

```
    deploy:
      resources:
        limits:
          cpus: "0.3"
          memory: 1G
```

## Separando Nginx e Node.js entre containers diferentes

Comando para ver os logs de um serviço:

- docker compose logs nginx:

Habilitar a porta:

```
services:
  app:
    build: .
    restart: always
    # ports:
    #  - 8080:3000
```

```
  nginx:
    build:
      context: .
      dockerfile: Dockerfile.nginx
    ports:
      - 8080:80
```

Nginx é um servidor web robusto, vai suportar grande número de acessos, tem muito mais recursos em relação a ser um servidor web do que o node.

Então, poderia haver 10 réplicas de nodejs e, no arquivo conf, poderia fazer uma configuração para o nginx balancear essa carga entre essas réplicas. Se acontecer de o app crashear, o nginx continua funcionando, não consegue mais servir a aplicação do nodejs, mas continua servido outras aplicações. 

Então, quanto menor o número de ferramentas que são colocadas dentro de um container, melhor.

Regra de ouro:

Coloque uma ferramenta dentro do container.

Assim, é melhor para balancear, gerenciar mais as falhas. 

Caso o nginx crashear, é mais fácil de verificar que é o nginx que está falhando também, porque ele está separado das aplicações.

## Administrando containers do docker compose

https://github.com/devfullcycle/mba-docker/blob/main/cap%2005%20-%20Docker%20Compose/07-Administrando%20containers%20do%20Docker%20Compose.md

## Depends on e health check

https://github.com/devfullcycle/mba-docker/blob/main/cap%2005%20-%20Docker%20Compose/08%20-%20Depends%20on%20e%20health%20check.md

Com a definição de um healthcheck, podemos usar o depends_on para garantir que o container só será iniciado após o container que ele depende ser iniciado e o healthcheck para garantir que o container já esteja pronto para receber conexões.

```
app:
    build: .
    ports:
      - 3000:3000
    depends_on:
      db:
        condition: service_healthy
```

```
db:
    image: mysql:8.0.30-debian
    environment:
      MYSQL_ROOT_PASSWORD: root
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 1m30s
      timeout: 10s
      start_period: 40s
      retries: 3
```

O depends_on possui a opção condition, que pode ser service_started ou service_healthy. A opção service_started é o padrão, ela não verifica se o container está saudável, ela só verifica se o container já está em execução. A opção service_healthy verifica se o container está saudável, ou seja, se o healthcheck do container retornou sucesso. Quando usamos o depends_on, como no exemplo lá em cima, a condição padrão é o service_started.