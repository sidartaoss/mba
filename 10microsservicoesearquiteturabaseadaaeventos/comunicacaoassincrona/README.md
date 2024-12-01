# Comunicação assíncrona

![O perigo do rest](/10microsservicoesearquiteturabaseadaaeventos/imagens/perigo_do_rest.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Utilizar REST para todas as comunicações entre microsserviços tende a gerar um risco muito grande de termos efeitos colaterais:

- De perda de informação;

- De ter uma latência maior e um response time muito baixo caso os microsserviços, mesmo que não estejam fora do ar, estejam mais lentos;

Para evitar esse tipo de problema, temos mecanismos de comunicação assíncrona:

- Os sistemas não mais vão se comunicar diretamente;

- Os processamento vão ocorrer em momentos diferentes;

- Um sistema vai estar desacoplado de outro sistema.

A comunicação assíncrona sempre vai ser mais segura em relação à parte de resiliência, permitindo que o sistema fale com mais sistemas ao mesmo tempo sem ter um acoplamento muito forte.

## Economia de recursos

Vantagens:

- Falar com diversos sistemas, sem precisar falar imediatamente:

    - Se esses sistemas estiverem fora do ar, não vai perder a mensagem.

- Se a organização tem muitos dados a serem processados, vai economizar milhões em infraestrutura:

    - Exemplo:

        - 100 pessoas para fazer o checkin no aeroporto;

        - E apenas 5 atendentes para trabalhar;

        - Se, nesse caso, tivesse uma comunicação síncrona, teria que ter 100 atendentes para atender essas 100 pessoas para fazer o checkin, porque só é possível falar com 1 pessoa de cada vez;

        - Mas, se só tem 5 pessoas para atender, nós vamos enfileirar as requisições;

        - Seria muito bom se tivéssemos 1 atendente para cada pessoa, pois o tempo de atendimento seria muito menor. Mas o gasto seria exponencial;

        - Nesse caso, ao enfileirar as requisições, nós podemos ter menos recursos para atender uma quantidade maior de solicitações;

        - A diferença, nesse caso é que, para atender esse número maior de solicitações, essas solicitações vão ter que ficar enfileiradas por alguns momentos para que sejam atendidas.

        - Similarmente, você consegue resolver muito mais solicitações do que você tem de hardware para resolver em tempo real de forma assíncrona.

        - Imaginando um cenário onde temos 1 milhão de visitas no site para realizar um pagamento, seria inviável ter 1 milhão de máquinas para atender todos esse pagamentos ao mesmo tempo.

        - Então, o que fazer?

        - Enfileiramos essas requisições de pagamento e, conforme as máquinas que temos disponíveis vão processando, os próximos pagamentos vão sendo processados.

        - O cliente não vai ter o pagamento processado em tempo real, mas, daqui 1 minuto, ele sabe que será processado.

        - Isso vai gerar uma economia muito grande, porque, mesmo que não tenhamos uma quantidade muito grande de recursos, nós sabemos que o processamento vai acontecendo aos poucos.

## Tópicos versus exchange

Temos duas abordagens principais para se trabalhar com tecnologias diferentes para utilizar comunicação assíncrona.

### Tópicos

Uma forma é utilizando mecanismos que trabaham com o padrão pub/sub (publisher/subscriber).

![Pub/Sub](/10microsservicoesearquiteturabaseadaaeventos/imagens/pub_sub.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

- Um sistema produz a mensagem (publisher);

- Essa mensagem vai ficar guardada em um tópico, que é o local aonde essas mensagens serão enviadas. (Normalmente, chamamos de tópico, dependendo do sistema, podemos chamar de fila também);

- Temos sistemas (subscriber) pendurados nesse tópico, aguardando as mensagens chegarem. Conforme as mensagens forem chegando, os sistemas vão lendo as mensagens.

- Exemplo:

    - Apache Kafka;

    - Apache Pulsar;

    - AWS Kinesis;

    - Google Pub/Sub;

    - AWS SQS.

### Exchange

![RabbitMQ](/10microsservicoesearquiteturabaseadaaeventos/imagens/rabbit_mq.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Uma outra abordagem é trabalhar com exchanges, que é o caso do RabbitMQ.

Ao trabalhar com exchanges, temos mais flexibilidade no momento de consumir os dados, porque, no momento de enviar uma mensagem, essa mensagem não enviada diretamente para uma fila, ela é enviada para uma exchange.

O que a exchange faz? 

- A exchange tem regras de distribuição de uma mensagem;

- Assim que os sistemas lêem uma mensagem, essas mensagens são apagadas (dropadas) da fila. Se precisar ler novamente a mensagem, não vai conseguir;

    - Existe um outro tipo de fila do RabbitMQ que se assemalha ao tópico do Apache Kafka, que é o tipo stream. O stream de dados as mensagens guardadas na fila, caso seja necessário fazer a releitura dessas mensagens.

- Em abordagens como o Apache Kafka, quando a mensagem é lida do tópico, essa mensagem ainda fica guardada nesse tópico e, caso se queira ler novamente a mensagem do tópico, é possível ler. E é possível escolher, também, por quanto tempo a mensagem deve ficar armazenada no tópico.

