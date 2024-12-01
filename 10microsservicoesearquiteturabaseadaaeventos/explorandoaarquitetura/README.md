## Monolito clássico

- Um monolito clássico que consegue escalar.

![Monolito clássico e escalável](/10microsservicoesearquiteturabaseadaaeventos/imagens/monolito_classico_e_escalavel.png)

- Toda vez que estamos falando de um monolito clássico, trata-se de um sistema e, dentro desse sistema, nós temos diversos componentes:

    - Módulos dentro do mesmo sistema;

    - Classes dentro do mesmo sistema;

    - Tem tudo aquilo que faz com que o sistema funcione:

        - Normalmente, tem um ou mais bancos de dados;

        - Normalmente, também tem um sistema de cache;

        - E, conforme a demanda de acesso for crescendo, tende a ser necessário colocar um Load Balancer (ou API Gateway) na frente e o sistema passa, então, a crescer horizontalmente.
  
- No caso em que for necessário utilizar um Load Balancer (ou API Gateway):

    - O disco deve ser efêmero;

    - No momento em que for fazer o upload de um arquivo, não vai fazer upload no próprio disco da máquina em que estiver a aplicação, porque a aplicação pode morrer a qualquer momento; no momento em que essa máquina for removida, todos os dados armazenados na máquina também serão removidos. Nesse caso, pode-se subir em um cloud storage, s3, ftp, outro local que guarda arquivos, um local persistente de dados;

    - Os logs devem ser uploadados. Tudo que tiver de logs na aplicação também não vão poder ficar armazenados na máquina, porque, se a máquina for apagada, vai-se perder os logs de tudo o que aconteceu;

    - Sessões dos usuários não podem ficar armazenadas em disco. Em um momento o usuário vai estar autenticado, em outro momento, não: quando a máquina morrer, vai perder a sessão e o controle de autenticação do usuário;

    - Não armazenar estado;

    - Regra: o software deve estar pronto para ser criado e recriado sem nenhuma perda de dados;
        
        - Se o comportamento da aplicação mudar pelo fato de você destruir a aplicação e subir de novo, significa que você está armazenando estado.

## Arquitetura baseada em microsserviços

Agora, cada componente que estava no monolito pode virar um serviço, pode virar uma aplicação.

![Arquitetura baseada em microsserviços](/10microsservicoesearquiteturabaseadaaeventos/imagens/arquitetura_baseada_em_microsservicos.png)

Isso acaba gerando muitos efeitos colaterais:

- Os microsserviços vão se comunicar;

- Uma arquitetura de microsserviços pode ser mais lenta, se fizer uso de comunicação síncrona, em que cada chamada a uma aplicação que chama outra aplicação que chama outra aplicação, etc., pode ir adicionando cada vez mais latência no processamento do request (dupla latência).

- Tem que haver estratégias para evitar essa chamada dominó. É necessário saber, também, quando disponibilizar o dado em tempo real: quanto maior é o sistema, raramente vai ser necessário entregar o dado quente ou o dado na hora;

- Cada microsserviço tem o seu próprio banco de dados. Então, é possível explorar outras opções como, por exemplo, um key-value store ou outro banco de dados não-relacional;

## Aumento de complexidade

![Aumento de complexidade](/10microsservicoesearquiteturabaseadaaeventos/imagens/aumento_de_complexidade.png)

- Escala horizontal;

- Todos os microsserviços vão ter um Load Balancer;

    - Antes, para um monolito, era necessário apenas um Load Balancer. Agora, para 10 micorsserviços, são necessários 10 Load Balancers, pele menos.

- Múltiplos processos de CI/CD;

- Múltiplas verificação de segurança;

- Mais provisionamento de infraestrutura;

- Mais custo;

- Governança mais complexa;

- Mais risco / efeito dominó:

    - Se um microsserviço cai, toda a rede pode ficar mais lenta, porque as aplicações dependentes vão ficar presas no microsserviço que está caindo ou está lento, gerando, então, um efeito dominó (requisições síncronas (dupla Latência)); 
    
    - Por isso, não é recomendado trabalhar com requisições síncronas;

- Mais latência;

- Observabilidade mais complexa;

- Troubleshooting;

    - Se não consegue encontrar a origem do erro via tracing, o troubleshooting fica mais complicado.

## Coreografia versus Death Star

A comunicação síncrona entre microsserviços pode gerar a coreografia de microsserviços, onde há a livre comunicação entre as aplicações: o A chama o B, que chama o C, etc., e começa a ter um relacionamento e uma forte dependência entre os microsserviços.

Quando você vai olhar na rede para entender como está a chamada de um microsserviço para o outro, a coisa parece muito caótica, gerando um problema de gestão de rede e de custos de infraestrutura.

Quando temos esse cenário, a gente começa a viver alguns efeitos colaterais, sendo um deles a chamada Estrela da Morte (Death Star). Significa que, quando você vai olhar o gráfico das chamadas de rede, você vai perceber que você tem um ambiente caótico e, aí, você começa a perceber o caos que a sua rede pode se tornar.

## Topologia de microsserviços

O que você pode fazer para melhorar esse cenário, melhorar a gestão dos microsserviços, evitando uma death star, por exemplo, é começar a trabalhar com topologia de microsserviços.

![Topologia de microsserviços](/10microsservicoesearquiteturabaseadaaeventos/imagens/topologia_de_microsservicos.png)

Dica de ouro:

``
Separar/Delimitar um conjunto de microsserviços (exemplo, Cobrança), correspondente a um subdomínio de negócio, por API Gateways.
``

A API Gateway vai agrupar os microsserviços por contexto de negócio. 

Todas às vezes que um microsserviço precisar chamar um outro microsserviço de um outro contexto, não vai mais existir a chamada direta. 

Ao invés de um microsserviço chamar outro microsserviço diretamente, a chamada é feita pela API Gateway para outra API Gateway, que corresponde a outro subdomínio de negócio (exemplo, Logística).

Vantagens:

- Times não precisam saber com qual microsserviço eles vão se comunicar;

    - Eles vão chamar uma API Gateway em determinado endereço e vão ter esse resultado (response);

- Equipes em outros contextos poderão mudar, refazer, reconstruir, trocar a linguagem de programação, mantendo o mesmo endpoint; TRANSPARENTE para quem utiliza o microsserviço;

- Ao utilizar API Gateway, pode-se fazer uso de políticas de retry, segurança, rate limit, de forma pré-configurada no API Gateway e o microsserviço não vai precisar se preocupar com essas configurações;

- Deve-se deixar as API Gateways de forma stateless, com arquivo declarativo e sem banco de dados.

    - Dessa forma, é possível matar e subir a API Gateway novamente a qualquer momento que se desejar. Quando criar um novo contexto, basta subir uma nova API Gateway dentro desse contexto;

    - Exemplo: Kong, KrakenD;

    - É necessário preparar o time para não fazer essas chamadas diretas entre microsserviços. Deve haver uma Network Policy proibindo chamadas diretas entre microsserviços de contextos diferentes.

## Enterprise gateways

- Têm mais recursos e mais facilidades;

- Gerenciamento de APIs externas/internas;

- A exposição fica na borda dos microsserviços;

- De forma automática, através de um portal do desenvolvedor ou um portal de gerenciamento de API Gateway, consegue fazer o gerenciamento das rotas;

- Sistemas de administração de APIs;

- Suporte a vários ambientes (dev, prod);

    - Consegue trabalhar de uma forma mais multi-tenant;

- Recursos específicos do vendor (lock-in):

    - Recursos de conversão de dados;

    - Recursos para fazer acesso a dados diretamente;

    - Recursos para injetar dados no cabeçalho;

    - Dentro de uma única chamada REST, API Gateway pode fazer múltiplas chamadas ao mesmo tempo: obter os dados, combinar em um único formato e retornar;

- Possui dependência externa:

    - Precisa controlar estado; para isso, utilizam banco de dados;

    - Nesse caso, você vai brigar ou pela consistência ou pela disponibilidade dos dados no API Gateway;

    - Exemplos:

        - API Gateway da Sensedia;

        - API Gateways dos Cloud Providers.

## Micro gateways

![Enterprise gateway](/10microsservicoesearquiteturabaseadaaeventos/imagens/enterprise_gateway.png)

- Roteamento de tráfego;

    - Atuando como um proxy reverso;

- Não há sistemas de administração de APIs (painel de controle). Gestor do time de API's vai lá e cadastra diversas rotas;

- API's são configuradas de forma declarativa, dentro de uma configuração, normalmente dispõe um arquivo para o Kubernetes subir e ler esse arquivo;

- Equipe precisa, manualmente, configurar os endpoints e fazer o deploy para cada mudança;

- Não possui dependência externa:

    - Não armazenam estado;

    - Pode derrubar e pode subir quantas vezes quiser;

    - Não depende de banco de dados externo;

    - Não tem problemas de disponibilidade e consistência, como em relação a enterprise gateways.

- Não possui suporte a vários ambientes (dev, prod);

    - Se for necessário suporte a diversos ambientes, será necessário criar diversas micro gateways;

- Não possui vendor (lock-in), porque, na maioria das vezes, esses tipos de gateways são open source;

- Roda standalone sem dependência externa:

    - Não precisa de banco de dados;

    - Não precisa de cache.

- As enterprise gateways recebem as chamadas externas e apontam para as micro gateways;

- As micro gateways podem ser utilizadas internamente para fazer a comunicação com outras micro gateways, atuando na frente de microsserviços que compreendem contextos (subdomínios) de negócio;

- Nem a enterprise gateway sabe diretamente qual microsserviço que ela vai chamar, ela sabe qual é o escopo que ela vai chamar;

    - Exemplo:
        
        - Enterprise gateway /charge => micro gateway /bill;

## Autenticação e autorização

Quando se trabalha com microsserviços, não necessariamente você vai precisar fazer a autenticação de cada microsserviço. 

Dependendo da situação, o seu microsserviço já vai receber a requisição já autenticada.

![Autorização. Situação 1.](/10microsservicoesearquiteturabaseadaaeventos/imagens/autorizacao_situacao_1.png)

1. Usuário vai enviar a sua requisição para um enterprise gateway;

2. Essa requisição de autenticação vai acessar diretamente um servidor de autenticação (servidor de identidade);

3. O servidor de identidade vai verificar se as credenciais estão corretas;

4. Caso estejam corretas, o servidor de identidade vai retornar para o enterprise gateway retornar para o usuário um token (access token) e também um id token;

5. (O id token, em cima do openid connect, vai prover, inclusive, as informações sobre o usuário; não necessariamente sobre o que ele pode acessar.);

6. Esse token (access token) vai ser a chave que o usuário vai utilizar para fazer a próxima requisição;

7. É muito comum esse token ser no formato jwt (json web token); além de carregar informações do usuário, também pode carregar as roles;

8. Nas próximas requisições que o usuário fizer, ele vai passar o jwt.

Situação #1:

- Usuário envia a requisição passando o token (jwt);

- Vai bater na api gateway;

- A api gateway envia a requisição para o servidor de identidade;

- O servidor de identidade verifica se o token é válido;

- Se for válido, o processo de acesso continua e o microsserviço é acessado diretamente.

Os microsserviços não estão verificando a autenticação em nenhum momento, porque a autenticação já passou pelo api gateway.

Essa abordagem tem um problema grave: a cada requisição que o usuário faz, toda requisição vai bater no servidor de autenticação. Isso torna o servidor de autenticação um ponto único de falha, porque ele vai estar sempre sobrecarregado.

O servidor de autenticação consegue emitir novos tokens, porque ele tem uma chave privada. Mas, para verificar se o token é válido, basta ele nos prover uma chave pública. Com essa chave pública, conseguimos verificar se o token do usuário foi realmente emitido pelo nosso servidor de autenticação.

Nesse caso, podemos deixar essa chave pública disponível na api gateway. 

Então, toda vez que o usuário enviar uma requisição para qualquer microsserviço, ele vai enviar o token jwt e a api gateway apenas vai verificar se o token é válido, se foi realmente emitido pelo servidor de autenticação, porque, agora, ela tem uma chave pública, que permite validar se o token é real ou não. 

Sendo real, a api gateway já encaminha a requisição para os microsserviços dentro da aplicação.

## Efeitos colaterais

No caso em que, no momento em que o servidor de identidade gerar o token, ele tiver que gerar, também, uma expiração, ou seja, por quanto tempo esse token for válido.

Digamos que o tempo seja uma hora e, depois de um minuto, desejar-se invalidar a autenticação do usuário.

Como a api gateway não está a todo momento batendo no servidor de autenticação, durante essa uma hora que o usuário enviar o token para a api gateway, ela vai validar esse token como sendo válido.

Então, se for necessário cancelar o login do usuário, durante o tempo de expiração, o usuário ainda vai poder acessar o sistema.

Nesse momento, é necessário ter estratégias de verificação para saber quanto tempo vai ser necessário utilizar de expiração para cada token.

Caso o tempo de expiração seja alterado para 5 minutos, por exemplo, garante-se que, por 5 minutos, o usuário vai sempre acessar, mesmo que o seu login seja cancelado. Passando 5 minutos, esse login é invalidado e, então, o usuário vai ter que utilizar um refresh token. Assim, uma nova solicitação vai bater no servidor de autenticação para obter um novo token para validar por mais 5 minutos.

Então, quanto maior for o tempo de expiração do token, maior vai ser o tempo em que se perde o controle sobre o acesso e o usuário vai poder acessar a aplicação. Porém, quanto maior for esse tempo, menor vai ser a carga no servidor de autenticação.

Quanto menor o tempo, consegue-se ter um controle maior, porque o token vai expirar mais rapidamente, porém, a carga no servidor de autenticação vai ser maior.

## Microsserviços versus redes internas

Situação #2:

Às vezes, não temos uma api gateway ou, às vezes, mesmo tendo uma api gateway, não queremos fazer a validação desse token na api gateway; podemos fazer a autenticação no próprio microsserviço.

Nesse caso, a chave pública que estava na api gateway que foi gerada pelo servidor de autenticação pode ficar disponível em cada microsserviço.

Assim, toda vez que a gente receber uma requisição, essa requisição vai bater diretamente no microsserviço, o microsserviço vai validar o jwt usando essa chave, se estiver ok, ele processa a requisição, se não estiver ok, ele não processa a requisição, porque não está autorizado.

Qual é o problema com essa abordagem?

- O microsserviço vai começar a receber requisições as quais o usuário não tem autenticação. Mas, mesmo assim, ele vai ter que tratar esse tráfego;

- Imaginando um monte de requisições não autorizadas, o que vai acontecer? O microsserviço vai ter que ficar negando essas requisições o tempo todo;

- Isso não é bom, porque tende a degradar a performance da aplicação.

Sendo assim, é necessário decidir:

- Ou o microsserviço valida, realmente, todo jwt que chegar para verificar se é válido ou não;

- Ou o microsserviço acredita, por padrão, que todo jwt que chegar vai ser válido, porque, em algum momento, ele passou pela validação da api gateway;

    - A aplicação acredita em todos os dados que vão chegar dentro da sua rede interna, porque parte-se do princípio de que tudo que chegou é válido, por conta de que essa validação bateu na api gateway;

    - Existe algum problema em acreditar? Absolutamente nenhum problema. Principalmente, se os microsserviços não estiverem expostos pela Internet.

    - O que fazer para garantir que os microsserviços não estejam expostos pela Internet?

        - No momento de configurar os microsserviços, pode-se jogá-los em uma rede fora da Internet (i.e., sem ingress direto, sem um Internet gateway para que se consiga acessar), ou seja, dentro de uma subnet sem acesso a Internet. Quem vai ter acesso a esses microsserviços é a api gateway, porque ela consegue falar nessa rede, ela faz parte de outra subnet pública, que consegue acessar a subnet dos microsserviços;

        - Assim, será possível ter acesso aos microsserviços somente se a requisição bater na api gateway. A api gateway deve ter acesso a Internet, ou seja, o usuário final vai bater na api gateway.

![Autorização. Situação 2.](/10microsservicoesearquiteturabaseadaaeventos/imagens/autorizacao_situacao_2.png)
<center>Fonte: Full Cycle, 2024.</center>

Qual é o problema de colocar as chaves públicas nos microsserviços?

- Imaginando que tenhamos 200 microsserviços, então, será necessário copiar essa chave pública para esses 200 microsserviços;

- Se, por algum motivo, for alterado essa chave pública, será necessário mudar essa chave pública nesses 200 microsserviços; será necessário fazer um redeploy dessas chaves públicas. 

- Dessa forma, recomenda-se que, quanto menos esforço for necessário para lidar com autenticação, melhor.



### Referência
MBA ARQUITETURA FULL CYCLE. Microsserviços e arquitetura baseada a eventos. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 29 nov. 2024.