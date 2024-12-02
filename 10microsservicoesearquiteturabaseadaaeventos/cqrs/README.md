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

Quando estamos falando de CQRS, damos um passo além, porque separamos o sistema completamente. Lembrando que, com CQS, estávamos de falando de método, ou seja, algo mais interno do sistema.

### Comandos

No CQRS, estamos falando de algo mais arquitetural, porque, na metade do sistema, temos comandos, ou seja, intenções do usuário: processar pagamento, criar cliente, etc.

Quando um comando é executado, esse comando vai ter que processar regras de negócio, falar com banco de dados, etc., então, quando o comando chega no sistema, ele vai interagir com o modelo de domínio, agregados, value objects, repositórios, para trabalhar com as informações do comando.

Essa parte do sistema deveria ter uma modelagem rica de domínio, porque é a parte onde os dados são processados. Por exemplo, uma simulação de cálculo de juros. Então, é enviado um comando contendo quanto gostaria de financiar, em quantas vezes, passando por todas as regras de negócio e grava no banco de dados todas as parcelas de juros. 

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



