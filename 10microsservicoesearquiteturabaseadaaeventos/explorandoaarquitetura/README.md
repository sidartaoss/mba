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

![Enterprise gateway](/10microsservicoesearquiteturabaseadaaeventos/imagens/enterprise_gateway.png)

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





### Referência
MBA ARQUITETURA FULL CYCLE. Microsserviços e arquitetura baseada a eventos. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 29 nov. 2024.