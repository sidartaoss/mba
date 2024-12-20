# CQRS (Command Query Responsibility Segregation)

Padrão ao qual sistemas que têm que ter leitura e escrita intensivos acabam utilizando.

## CQS

Para definirmos o conceito de CQRS, antes, é necessário definir o conceito de CQS (Command Query Separation), criado por Bertrand Meyer:

- Um método tem que ser responsável:
    
    - Ou por escrever, processar uma informação;

    - Ou para retornar uma informação;

    - Ou seja, temos um comando que executa uma operação ou um comando que traz os dados de uma operação;

    - Mas, esse sistema, ao executar um comando, não traz o dado.

    - Exemplo:

        - Método POST em uma API REST. Normalmente, quando o dado é persistido, é retornado os dados que foram persistidos e o id do novo dado.

        - Nesse caso, nós temos duas responsabilidades: 

            - Inserir o dado;

            - Ler esse dado e retornar para o usuário final.

    - O CQS não retorna o dado. Ele parte do princípio de que os dados que ele insere como informação nunca vão ser retornados.

- O conceito de CQS foi evoluído para o conceito de CQRS por Greg Young.

## CQRS

![CQRS](/10microsservicoesearquiteturabaseadaaeventos/imagens/cqrs.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

CQS:

- Quando, dentro de um sistema, você tem comandos, esses comandos são executados, normalmente, gravam alguma coisa, mas não vão responder com uma resposta; nunca vão consultar dados para retornar uma resposta. Os comandos somente executam.

Quando estamos falando de CQRS, damos um passo além, porque separamos o sistema completamente. Lembrando que, com CQS, estávamos de falando de métodos, ou seja, algo mais interno do sistema.

### Comandos

No CQRS, estamos falando de algo mais arquitetural, porque, na metade do sistema, temos comandos, ou seja, intenções do usuário: processar pagamento, criar cliente, etc.

Quando um comando é executado, esse comando vai ter que processar regras de negócio, falar com banco de dados, etc., então, quando o comando chega no sistema, ele vai interagir com o modelo de domínio, agregados, value objects, repositórios, para trabalhar com as informações do comando.

Essa parte do sistema deveria ter uma modelagem rica de domínio, porque é a parte onde os dados são processados. Por exemplo, uma simulação de cálculo de juros. Então, é enviado um comando contendo quanto gostaria de financiar, em quantas vezes, passando por todas as regras de negócio e gravando no banco de dados todas as parcelas de juros. 

Quando é emitido esse comando e as parcelas são gravadas no banco de dados, acabou o processamento.

Os dados das parcelas não são retornados para o usuário final.

O comando apenas executa a informação e nada mais. O comando nunca vai trazer de volta o resultado da operação para o usuário final.

Se deu erro, será tratado, vai gerar um evento para ser processado mais tarde.

Se deu certo, vai gravar, fazer o processamento e nada mais.

Então, a parte de comando, normalmente, vai ser bastante complexa, porque é aí que mora o coração do software. É importante ter a clareza de que, por trás dos comandos, normalmente, se tem as regras de negócio da aplicação.

### Queries

Da mesma forma que temos os comandos, temos, também as queries, por isso o nome command query responsibility segregation, porque é separado a responsabilidade entre comandos e consultas.

Imaginemos, então, que o usuário deseja saber os juros das parcelas do financiamento que ele fez.

No primeiro momento, um comando foi executado e, agora, uma consulta será executada.

Essa consulta vai lá no banco de dados trazer a informação num determinado padrão.

A camanda de queries não contém regras de negócio. Ela somente lê, é informado o formato que o usuário quer trabalhar, busca no banco de dados e retorna no formato que o usuário deseja.

Nesse caso, não é necessário trabalhar com domain-driven design, repository, etc. Normalmente, trabalha-se com dao's, pois é necessário apenas consultar os dados e retornar para o usuário final.

## Banco de dados e cqrs

Com CQRS, temos uma forma opcional de trabalhar com banco de dados.

Muitas pessoas também trabalham, conforme, também, as necessidades do sistema, com dois bancos de dados: um banco de dados para escrita e um banco de dados para consulta.

E, eventualmente, esses bancos de dados vão ser sincronizados.

![CQRS e bancos de escrita e leitura](/10microsservicoesearquiteturabaseadaaeventos/imagens/cqrs_bancos_escrita_leitura.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Existem alguns motivos para se trabalhar dessa forma:

- Muitas vezes, tem-se um banco de dados que tem uma operação muito intensa de escrita e você não deseja sobrecarregar esse banco ainda mais com operações de leitura, porque vai fazer com que você tenha baixa performance.

- Muitas vezes, temos a situação contrária: um banco de dados com uma operação intensiva de leitura, mas tem uma baixa operação de escrita ou, eventualmente, ambos têm uma operação intensiva de escrita e leitura.

- Às vezes, você quer gravar os dados e, na hora que você vai gravar os dados, você vai precisar de isolamento (ACID). Então, você vai gravar os dados de forma estruturada em um banco de dados relacional.

    - Porém, para fazer as consultas, os dados estão estruturados de uma forma tão complexa, que fica difícil fazer a leitura.

    - Por exemplo, para fazer um relatório onde é necessário fazer uma query gigantesca e otimizada.

    - Para facilitar, o que se faz? 

    - Grava-se os dados no banco de dados de forma estruturada, com um banco de dados relacional, mas, também, vai-se ter uma cópia dessas informações estruturada de tal forma que fique muito fácil fazer as consultas. E, por ficar mais fácil de fazer as consultas dessa forma, eventualmente, pode ficar até mais performático fazer as consultas.

    - Então, nesse caso, faz sentido ter dois bancos de dados: um para leitura e um para escrita. Mas a estrutura desses bancos de dados são diferentes, porque queremos mais simplicidade e velocidade na leitura, mas queremos ter mais estrutura e consistência na escrita.

    - Recomenda-se separar os bancos de dados quando temos queries muito complexas, porque, dessa forma, pode-se, por exemplo, separar um relatório inteiro por colunas, tem-se opções diferentes de bancos de dados: banco de dados orientado a documentos, banco de dados colunar, banco de dados específico para search, etc.

    - Desvantagem:

        - O problema de trabalhar dessa forma é a consistência eventual.

        - Nem sempre o dado para leitura vai estar atualizado.

        - Esses dados precisam de um tempo para sincronizar, nem que seja 1 milissegundo. Mas, para alguns sistemas, 1 milissegundo faz diferença.

        - É necessário saber se o sistema que estamos trabalhando poderá estar inconsistente em algum momento. Se, por algum motivo, ele não puder estar inconsistente em algum momento, então, teremos que ter somente um banco de dados.

## Mecanismos de sincronização

Podemos ter diversos mecanismos de sincronização: alguns são muito simples e alguns são muito complexos.

É importante saber qual mecanismo utilizar para, de forma intencional, saber a melhor forma de trabalhar no projeto.

### Message queues or pub-sub system

![Message queues or pub-sub system](/10microsservicoesearquiteturabaseadaaeventos/imagens/message_queues_pub_sub_system.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

- Nesse mecanismo, temos o comando que vai gravar os dados no banco de dados de escrita, mas esse comando também vai gerar um evento;

- Esse evento vai cair em um tópico;

- O consumidor vai ler os dados desse tópico e vai gravar no banco de dados de leitura.

Vantagens:

- Isso é interessante, porque o consumidor, quando lê o evento, consegue obter o evento e gravar no banco de dados no melhor formato possível em termos de desempenho.

- Também não tem perigo de perder dados durante a sincronização, porque estamos guardando as informações em um tópico. Então, caso o consumidor caia, quando ele subir novamente, ele vai ler o tópico e gravar no banco de dados. Permite bastante resiliência, porque a aplicação vai garantir que os dados não vão ser perdidos.

Quando utilizar:

- Quando é necessário garantir resiliência;

- Quando é necessário garantir, de forma muito rápida, que o dado seja sincronizado, porque, normalmente, esses sistemas de mensageria e pub-sub têm latências muito baixas de 10 milissegundos ou até menos.

### Scheduled synchronization jobs

![Scheduled synchronization job](/10microsservicoesearquiteturabaseadaaeventos/imagens/scheduled_synchronization_job.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

- O comando grava os dados no banco de dados de escrita;

- Temos um job e um consumidor:

    - De tempos em tempos, o consumidor vai ler o banco de dados de escrita e vai gravar no banco de dados de leitura;

    - Exemplo: a cada 10 segundos, o job vai obter todos os dados que mudaram no banco de dados de escrita e vai jogar no banco de dados de leitura de uma vez (via consumidor).

Vantagens:

- Você não fica a todo momento batendo no banco de dados e não é necessário ficar disparando eventos para trabalhar.

- Trabalhar dessa forma acaba sendo bastante simples e não é necessário um mecanismo de fila para trabalhar. A única coisa necessária é de um job que rode, obtenha os dados e guarde:

    - É facil de implementar;

    - Não é necessário gastar com infraestrutura, ter um sistema de filas, etc.

Desvantagem:

- Tempo de sincronização vai ser maior, conforme o intervalo de execução do job: a cada 10, 15,... segundos.

É necessário saber o quão inconsistente o sistema pode estar entre a base de leitura e escrita. Às vezes, não tem problema o intervalo de tempo de inconsistência.

### Dual writes

![Dual writes](/10microsservicoesearquiteturabaseadaaeventos/imagens/dual_writes.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

- Temos um comando e esse comando vai gerar a gravação no banco de dados de escrita.

- Porém, esse comando, internamente, pode gerar um evento interno no sistema para gravar em um banco de dados de leitura.

- O mesmo sistema, durante o comando, grava ao mesmo tempo nesses dois lugares:

    - Banco de dados de escrita;

    - Banco de dados de leitura.

- No momento em que é enviado o evento do mecanismo de gravação no banco de dados de escrita, também será gravado no banco de dados de leitura.

    - Tendo algum problema para gravar no banco de dados de leitura, deve ser feito o rollback no banco de dados de escrita.

    - Vão haver duas operações distintas em bancos de dados diferentes. Se uma falhar, a outra operação também precisa voltar atrás, porque esses bancos de dados não têm outro mecanismo de sincronização. Então, é necessário garantir que o dado seja gravado no banco de dados de escrita e leitura.

### Change data capture (CDC)

![Change data capture](/10microsservicoesearquiteturabaseadaaeventos/imagens/change_data_capture.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

- O comando envia dados para o banco de dados de gravação.

- Temos um sistema que fica lendo os logs do banco de dados de gravação, obtendo essas informações e gravando no banco de dados de leitura.

Normalmente, CDC's são soluções prontas para ajudar a ter mais facilidade para trabalhar.

Exemplo:

- Kafka Connect:

    - Tem um cluster rodando; 

    - Tem um plugin, chamado Debezium, responsável por ficar lendo o banco de dados através de um conector source;

    - Tem outro conector sink, aonde ele despeja os dados para o banco de dados de leitura.

Vantagens e quando usar:

- Para fazer integração entre sistemas, com gravação em múltiplos bancos de dados, vai facilitar todo o processo, porque foi projetado para isso.

## Exemplo prático de commands

``
https://github.com/devfullcycle/cqrs
``

Neste caso, estamos trabalhando com eventos dual write de mecanismos de sincronização, o que significa que estamos escrevendo em dois bancos de dados diferentes, simultaneamente, dentro da própria aplicação, sem utilizar um mecanismo de sincronização externo.

![Parte interna da aplicação](/10microsservicoesearquiteturabaseadaaeventos/imagens/parte_interna_aplicacao.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Se formos ver a parte interna da aplicação, nós temos duas partes, duas camadas básicas: uma camada de comando, que são intenções do usuário e uma camada de consulta.

### Camada de comando (onde os dados são gravados)

Na camada de comando, temos:

- Uma área de domínio, onde nós temos regras de negócio, com pedidos, produtos, itens do produto, podemos adicionar itens ao pedido, obter o total, multiplicando a quantidade de itens de cada pedido.

- O repositório, onde gravamos os dados em um banco de dados de escrita.

- Um comando, com um método handle, que cria um novo produto e salva esse produto no banco de dados de escrita.
    
    - Esse método handle não retorna nada.

    - Com relação ao dual write, quando é gerado a transação, é disparado um evento para que seja utilizado para gravação dos dados no banco de dados de leitura. Uma vez que o evento é preparado, ele é disparado para o sistema.

    - Após disparar o evento, temos handler's que vão cuidar da operação quando o evento for disparado.

    - No caso do handler de produto criado, o handler grava no mongodb como banco de dados de leitura; o comando grava em um banco relacional como banco de dados de escrita e o handler grava em um banco orientado a documentos como banco de dados de leitura.

![Método handle](/10microsservicoesearquiteturabaseadaaeventos/imagens/metodo_handle.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

- Criou um comando;

- Esse comando gravou no banco de dados de escrita e gerou um evento, o qual pode gravar também em um banco de dados de leitura;

- Neste momento, nós temos dois bancos de dados de diferentes tipos, de formas diferentes de leitura os quais nós vamos conseguir trabalhar.


### Camada de leitura

Saindo da parte de comando, nós chegamos na parte de consulta (query).

A parte de query serve apenas para consulta: não vai ter nenhuma gravação de dados e não vai ter regra de domínio.

``
A grande quebra de paradigma é conseguir executar as transações sem retornar nada e ler essas transações sem depender de gravação também.
``

Na parte de consulta, só vai consultar: consultas específicas, com mais performance, com mais velocidade e com um banco de dados organizado especificamente para os formatos de consulta que você deseja.

Quando usar:

- Isso vai ajudar muito em relação a microsserviços, comunicação entre sistemas, quando os sistemas trabalham com comunicação assíncrona.
