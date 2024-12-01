# EDA (Event-driven architecture)

## Eventos

Informações que já aconteceram.

## Tipos de eventos

### Event notification

- Dados mínimos sobre algum evento que aconteceu;

- Mostrar quando um estado da informação muda. Trabalha muito com estado: quando algo muda no sistema, notifica sistemas que têm interesse nesse tipo de informação;

- A principal vantagem é trabalhar com uma quantidade de dados muito pequena, impactando minimamente no custo de processamento, evitando congestionamento da rede.

```
 {
 "order_id": 1,
 "status": "aprovado"
 }
```

### Event-carried state transfer

- Traz o evento em si (todos os dados);

- Somente com todo o conjunto de dados, é possível executar o processamento e a execução de determinadas tarefas de negócios. Por exemplo: emissão de notas fiscais;

- Cuidado para não utilizar em todos os cenários de negócio: pode gerar um volume muito alto de dados sendo trafegados e, congestionando a rede, os sistemas ficam mais lentos.

```
{
 "order_id": 1,
 "status": "aprovado",
 "product_id": 1,
 "product_name": "Carrinho de brinquedo",
 "descricao": "O melhor carrinho do mundo",
 "preco": 100,
 "user_id": 1,
 "nome": "Wesley",
 "email": "w@www.com"
 }
```

### Event sourcing

- Versiona cada evento, sendo possível replay;

- A utilização do event sourcing é muito específica: ajuda a guardar o histórico de tudo o que aconteceu no sistema. Caso de uso: auditoria;

- A cada vez que um evento ocorre, é criado um novo registro e os dados são persistidos em um banco de dados. Isso quer dizer que, se os registros forem lidos na ordem em que os dados foram persistidos, é possível recriar todo o histórico de mudanças de estados que ocorreram ao longo do tempo.

```
{
 "order_id": 1,
 "status": "aprovado",
 "timestamp": "x"
 },
 {
 "order_id": 1,
 "status": "enviado",
 "timestamp": "y"
 },
 {
 "order_id": 1,
 "status": "entregue",
 "timestamp": "z"
 }
```



