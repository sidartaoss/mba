# Event-driven architecture (EDA)

## Eventos

Evento é algo que aconteceu no sistema (algo no passado).

Informações que já aconteceram.

Exemplos:

- Compra aprovada, pagamento efetuado, nota fiscal emitida...

## Tipos de eventos

Existem alguns tipos de eventos nos sistemas.

- Esses tipos de eventos vão mudar a estratégia de comunicação a ser utilizada entre os sistemas que vão produzir os eventos e os sistemas que vão consumir esses eventos;

- Esses tipos de eventos são utilizados em casos de uso diferentes.

### Event notification

- Esse evento vai trazer dados mínimos sobre algum evento que aconteceu;

- O event notification vai mostrar quando o estado da informação muda;

    - Trabalha muito com estado: quando algo muda no sistema, ele notifica sistemas que têm interesse nesse tipo de informação;

- Por que (quando) trabalhar com event notification?

    - A principal vantagem é trabalhar com uma quantidade de dados muito pequena;
    
        - Quando temos um ecossistema de muitos microsserviços, por exemplo, Mercado Livre: milhões de eventos sendo processados por minuto;
        
        - Se cada evento contivesse muita informação, iria fazer com que o custo de processamento ficasse muito grande, além de congestionar a rede;
        
        - Nesse caso, o event notification impacta minimamente o custo de processamento e evita congestionamento de rede.

    - Muito utilizado quando os demais microsserviços têm interesse em mudanças de estado.

```
 {
 "order_id": 1,
 "status": "aprovado"
 }
```

### Event-carried state transfer

É um evento de stream, ou seja, carrega todos os dados que aconteceram.

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

- A utilização do event sourcing é muito específica: ajuda a guardar o histórico de tudo o que aconteceu no sistema. 

    - Casos de uso: 
        
        - Auditoria;

        - Erros no sistema. Ao rodar novamento o histórico de tudo o que aconteceu no sistema (replay), consegue entender o que está acontecendo.

- A cada vez que um evento ocorre, é criado um novo registro e os dados são persistidos em um banco de dados. Isso quer dizer que, se os registros forem lidos na ordem em que os dados foram persistidos (replay), é possível recriar todo o histórico de mudanças de estados que ocorreram ao longo do tempo.

- Garante muito mais segurança.

- Datomic (Nubank):

    - Banco de dados imutável. Tudo que acontece de mudança, para todos os update's que acontecem, é gerado um novo registro. Ou seja, por padrão, é feito o event sourcing e você essa auditoria.

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

## Schema evolutivo

Sempre que trabalhamos com eventos, nós temos o schema da mensagem.

Conforme o tempo passa, os sistemas mudam, os dados mudam e o schema de publicação de uma mensagem muda também.

Imaginando que temos 10 microsserviços lendo um microsserviço que está enviando uma mensagem. Se esse microsserviço começa a mandar a mensagem em um formato diferente:

- Os 10 microsserviços, ao ler a mensagem, vão começar a quebrar.

Então, é necessário uma maneira de reforçar o padrão de envio e recebimento das mensagens desses eventos.

A forma a qual fazemos isso, normalmente, é através de schemas. 

Ou seja, definimos para o consumidor e o publicador o schema de envio e recebimento de mensagens a ser seguido.

Quando é necessário evoluir o schema, (i.e., o schema precisa ser alterado), é necessário ter técnicas e mecanismos para manter a compatibilidade dos schemas.

Existem algumas formas de manter a compatibilidade:

- Forward compatibility:

    - Os dados são produzidos com um novo schema, mas ainda mantém compatibilidade de leitura com o schema antigo; 
    - Não há alteração de código pelo consumidor.

    - Exemplo: o produtor adiciona um novo campo na mensagem.

- Backward compatibility:

    - O dado produzido com um schema antigo pode ser lido como se fosse um novo schema;
    - Permite o consumidor se preparar para uma nova feature;
    - Reprocessamento de mensagens antigas.

    - Exemplo: o consumidor adiciona um novo campo na mensagem.

- Full compatibility:

    - Combinação dos dois mundos;
    - Difícil conseguir.


## Schema registry

![Schema registry](/10microsservicoesearquiteturabaseadaaeventos/imagens/schema_registry.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Schema registry tem a ver com a parte de compatibilidade de schemas.

O Confluence Schema Registry é o schema registry que é utilizado em conjunto com o Apache Kafka.

O que o schema registry faz?

- Cria-se um registry, ou seja, um padrão de mensagem;

- Esse padrão, normalmente, pode ser feito no padrão:

    - Avro - espécie de JSON, mas que, antes de trazer os dados, especifica os tipos dos dados;

    - Protocol Buffer - vai mostrar qual é o schema de envio das mensagens;

    - JSON.

- É criado uma versão do schema;

- Ao enviar a mensagem:

    - O produtor vai ler o schema;

    - Vai pegar o id do schema e vai enviar a mensagem para o Apache Kafka;

    - O Apache Kafka, para aceitar essa mensagem, vai saber que o formato dos dados tem que estar exatamente no formato do id daquele schema;

    - Se não estiver baseado naquele schema, o Apache Kafka não vai deixar ser enviado essa mensagem.

    - Obs.: para que o producer não tenha que ficar, a cada mensagem, lendo esse schema, ele vai gravar localmente em um cache para que ele não precise ficar enviando a todo momento.

    - Do outro lado, o consumidor faz a mesma coisa. Ele lê o schema, vai receber os dados e, agora, ele sabe qual é o padrão dos dados que ele vai ter que serializar para processar a informação.

    - Imaginando que esse schema evoluiu (foi alterado), vai ser gerado uma versão 2:

        - Quando o produtor for enviar a mensagem, o id que ele vai enviar é o id 2, o Kafka já vai saber qual que é o formato e o consumidor também vai saber qual que é o formato.

Quando não é possível trabalhar com schema registry:

- Recomenda-se criar um repositório;

- Nomear e criar os schemas para se trabalhar;

- Informar ao time que esse é o contrato a ser seguido.

### Referência
MBA ARQUITETURA FULL CYCLE. Microsserviços e arquitetura baseada a eventos. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 29 nov. 2024.
