# Introdução a microsserviços

## Sistemas monolíticos

- Todos os escopos e áreas dentro de um mesmo sistema.

- Todas as equipes trabalham em um mesmo sistema.

- Mais simples de se começar um projeto:
    
    - Tudo vai estar no mesmo projeto;

    - As pastas que você vai organizar vão estar no mesmo projeto;

    - Vai evitar comunicação desnecessária;

        - Vai evitar fazer comunicação com filas, fazendo milhares de chamadas REST para acessar os mesmos sistemas;

    - Tudo vai estar no mesmo banco de dados;

    - Para gerar relatórios, vai ser mais tranqüilo.

- Atende facilmente grande partes dos projetos em diversos tipos de empresa:

    - Vai estar tudo no mesmo sistema;

    - É mais fácil fazer o deploy;

    - Vai precisar apenas de uma pipeline de CI/CD;

    - Apenas uma organização para subir esse sistema no ar;

    - Processo de testes é mais simples;

    - Economizar em infraestrutura;

        - Menos recursos computacionais para fazer o deploy.

- Restrição de uso de diversas tecnologias, como linguagens de programação:

    - Tudo com a mesma linguagem;

    - Tudo no mesmo padrão;

    - Os profissionais que vão estar trabalhando com esse sistema vão estar sendo treinados apenas em uma linguagem: tem aspectos de produtividade, de forma indireta.

- Facilidade no processo de comunicação:

    - Comunicação em memória e somente dentro da mesma aplicação;

    - Pode evitar recursos, como circuit breaker, políticas de retry, diversas complexidades se tornam desnecessárias, porque tudo está no mesmo sistema;

    - Troubleshooting facilitado; é mais fácil saber onde esse erro está (desnecessário buscar em qual sistema aconteceu o erro).

- Risco maior no processo de deploy, se mal projetado (todas as áreas da empresa vão cair):

    - Em microsserviços, se um sistema cair, todos os demais vão continuar no ar;

    - Necessário trabalhar com testes;

    - Necessário trabalhar com formatos diferentes de deploy:

        - Bluegreen:

            - Replica a mesma infraestrutura e muda o tráfego para esse outro sistema.
            
            - Se tudo estiver ok, a infraestrutura antiga é apagada. Se não estiver ok, simplesmente roteia novamente para a infraestrutura antiga.

## O que são microsserviços?

``
Aplicações comuns com propósitos e escopo bem definidos.
``

- Microsserviço é um sistema como outro qualquer. Quando se cria um monolito e quando se cria um microsserviço, basicamente, está-se criando a mesma coisa; a diferença é o tamanho, o nível de responsabilidade do sistema. Enquanto os monolitos têm diversos escopos, diversas responsabilidades, diversos motivos para mudar, os microsserviços, normalmente, têm apenas um motivo para mudança.

- Independentes e autônomos:

    - Criar mecanismos de fallback: caso o microsserviço que você dependa diretamente estiver fora do ar, você vai degradar a entrega que você vai dar para o usuário final.

    - Exemplo: Mostrar saldo do usuário. No momento em que vai mostrar o saldo do usuário, o microsserviço que gera toda a conta para trazer o saldo está fora do ar. Para não entregar uma experiência ruim para o usuário, como um erro 500, você pode recuperar o saldo que foi entregue pela última vez e mostrar para o usuário: pelo menos, conseguiu-se gerar valor para o usuário final. Chama-se `gracefull degradation`. Para informações críticas, é necessário ter essa estratégia para não estourar um erro na cara do cliente.

- Participam de um ecossistema maior. Uma parte da engrenagem.


## Arquitetura baseada em microsserviços

- Possibilidade de múltiplas tecnologias;

    - Considerando política de governança.

- Processo de deploy menos arriscado;

    - Para um cenário de 100 microsserviços e, na hora do deploy de uma aplicação, o deploy der errado. 
        
        - Ainda temos 99 microsserviços no ar, com um problema em um deploy: apenas uma parte é afetada no sistema.

        - E, se foram criadas políticas, mecanismos de fallback, o sistema vai se degradar parcialmente e, ainda, vai conseguir atender requisições.

- Maior complexidade no setup;

    - Apesar de conseguir fazer o deploy de forma independente, você tem uma complexidade muito maior no setup desses microsserviços;

    - Exemplo: Para 10 microsserviços:

        - 10 processos de setup;

        - 10 esteiras de CI;

        - 10 esteiras de CD;

        - 10 processos de análise estática do código;

        - Pessoas diferentes para o code review;

        - Quando precisa se comunicar, precisa se comunicar com outros 9 sistemas;

        - Para uma microsserviço:

            - Rodar os testes;

            - Rodar a imagen docker;

            - Criar os POD's no Kubernetes;

            - Fazer testes de stress, para saber quanto vai utilizar de memória e CPU;

        - Agora, multiplique isso pela quantidade total de microsserviços;

        - Por isso que empresas grandes, normalmente, têm times de plataforma que fazem esse setup de forma mais organizada e padronizada;

- Equipes especializadas por microsserviço;

    - Especialista em resolver problemas de negócio;

- Auxilia no processo de escala de aplicações;

    - Pode escalar uma parte específica do sistema, ao invés do sistema inteiro;

        - Pode parecer que isso gera economia;

        - Mas, se você tem diversas aplicações no ar, você vai ter mais:

            - Processos de CI/CD, onde você precisa de recurso computacional para rodar isso;

            - Você vai precisar de mais máquinas para rodar esses microsserviços, uma vez que eles são independentes e, provavelmente, eles vão rodar em máquinas diferentes;

            - Provavelmente, vai precisar fazer uso de um bom gerenciamento do Kubernetes para fazer a orquestração das aplicações;

            - Vai precisar de processos diferenciados de backup para fazer isso.

        - Ou seja, para escalar um único microsserviço, para essa suposta vantagem de escalar apenas uma parte do sistema, você tem uma grande mudança na sua infraestrutura, que vai elevar muito o custo, sendo que, muitas vezes, esse custo elevado para subir essa infraestrutura vai ser muito maior do que se você tivesse escalado vários monolitos naquele momento;

        - Monolitos também escalam, tanto verticalmente quanto horizontalmente. Isso vai reduzir demais a necessidade de setup, a necessidade de orquestração dos microsserviços, vai diminuir muito a complexidade e, eventualmente, o custo de escalar diversos monolitos vai ser mais baixo do que escalar somente um microsserviço e manter toda a infraestrutura e toda a arquitetura baseada em microsserviços funcionando;

        - Além disso, a comunicação vai acontecer, muitas vezes, de forma assíncrona e, para que isso funcione, vai ser necessário um sistema de stream de dados, um sistema de mensageria. Isso quer dizer que vai ter mais um serviço no ar, mais um serviço em que vai ser necessário gerenciar, mais um serviço em que vai ser necessário treinar a equipe e, fundamentalmente, você vai ter mais um custo para a infraestrutura;

        - É contraintuitivo, mas, muitas vezes, o argumento de que escalar microsserviços é melhor do que escalar monolitos pode nem sempre ser verdade.

## Quando optar por microsserviços

Arquiteturas distribuídas são necessárias em grandes ambientes (grandes empresas, grandes negócios, onde têm muitos times, muitos desenvolvedores);

Aplicações em ecossistemas com 1.000, 2.000, 10.000, 15.000 desenvolvedores, onde você tem uma empresa tão gigante, onde os desenvolvedores nem se conhecem.

## Quando optar por monolitos

- Provas de conceito de um sistema que tem possibilidade de crescer;

- Projetos onde não conhecemos todos os contextos;

- Necessidade de realizar mudanças bruscas nas regras de negócio e funcionalidades do sistema;

    - Sistema novo, startups;

- Governança simplificada;

    - Empresa não é muito grande ou a área de tecnologia dessa empresa não é muito grande, onde ela não tem uma governança, uma arquitetura corporativa clara para os desenvolvedores;

- Restrições de orçamento;

    - Trabalhar com monolitos, geralmente, você gasta menos em relação à infraestrutura, em relação a colocar o software no ar - observabilidade, logs, cache, tracing, monitoramento. É mais barato, principalmente, porque você vai ter apenas um sistema para fazer isso.

- Baixa quantidade de desenvolvedores;

    - Se você tem uma equipe pequena, não faz sentido criar 10 microsseriços por desenvolvedor: é muito complexo;

    - Ou, se você não tem uma governança, um time de plataforma, cada vez isso vai ficar mais inviável;

    - A equipe tem que ter maturidade para trabalhar com isso; se nenhuma pessoa já trabalhou ou tem maturidade para trabalhar com microsserviços, jamais coloque microsserviços em um ambiente assim.

    ## Quando optar por microsserviços

    - Escalar times / alta quantidade de desenvolvedores;

        - Necessidade muito grande de escala - escala de times;

        - Quando tem uma quantidade muito grande de desenvolvedores, não vai conseguir escalar as equipes, porque é muito complexo ter um único sistema e ter 1.000, 2.000, 10.000 desenvolvedores mexendo no mesmo sistema;

        - Não é hype, é uma questão de necessidade - trabalhar com sistemas monolíticos acaba se tornando inviável.

    - Contextos e unidades de negócio definidos;

    - Necessidade de escalar partes específicas de um sistema;

        - No momento em que o sistema tem uma alta carga de trabalho, Black Friday, imaginando a quantidade de acessos que acontece no catálogo de produtos, escalar todo um sistema da Amazon, por exemplo, simplesmente, para fazer o catálogo de produtos estar disponível, não faz sentido;

        - Nesse caso, o custo com infraestrutura vale a pena. Em um sistema onde você não tem tanto acesso, você não tem tanta demanda, você falar que isso é uma vantagem, na verdade, não é; isso é só custo. Mas, em um ecossistema gigantesco, como na Amazon, por exemplo, provavelmente, escalar somente uma parte do sistema vai gerar uma grande vantagem e vai chegar ao ponto de economizar dinheiro. 
        
        - Mercado Livre, onde tem mais de 120.000 máquinas no ar: cada máquina está rodando alguns microsserviços diferentes. Não faz sentido subir tudo de uma vez só e escalar tudo de uma vez só. Nesse caso, o custo-benefício tende a ser maior.

    - Necessidade de tecnologias específicas para resolver problemas específicos;

        - Quando você tem um ambiente muito grande e um ambiente que necessita de recursos adicionais para trazer mais eficiência para o negócio: alto grau de processamento, trabalhar com machine learning, trazer informações em tempo real, toda vez que você tem linguagens de um portfólio muito maior de tecnologias para você trabalhar, você vai ter uma vantagem competitiva. Mas isso não faz sentido para, por exemplo, um site de 1/2 dúzia de visitas.

    - Se você tem baixo risco, baixo orçamento, baixo volume, provavelmente, microsserviços não vai ser algo que vai fazer uma grande diferença.

# Provocação sobre microsserviços

## O mínimo para trabalhar com microsserviços

- Seu time tiver maturidade:

    - CI;

    - CD;

    - CR (Container Registry).

- Arquitetura de software;

- Arquitetura de soluções;

- Comunicação assíncrona:

    - Message brokers.

- Concorrência:

    - Lock pessimista/otimista;

    - Locks distribuídos;

    - Mutex (mutual exclusion);

    - Race condition.

- Cache distribuído:

    - Tipos.

- Banco de dados para cada microsserviço;

- Como gerar relatórios:

    - Um microsserviço específico para gerar relatórios;

    - Trabalhar com event sourcing:

        - Flex tables (ir preenchendo os relatórios conforme os eventos vão chegando).

- DDD

    - Entender escopo, linguagem daquele escopo.

- Observabilidade

    - Logs padronizados;

        - Padronizado para conseguir fazer buscas e cruzamentos de logs entre microsserviços do sistema de observabilidade.

    - Métricas;

        - Apontamentos de anomalias. Para fazer isso de forma cruzada entre várias aplicações é mais complexo.

    - Tracing distribuído;

        - Sistema deu problema: aonde, em qual aplicação ocorreu o problema?

        - Passar o correlation id para conseguir identificar o request.

- Não é necessário saber tudo de uma vez, mas é importante saber para começar a estruturar o ambiente.

### Referência
MBA ARQUITETURA FULL CYCLE. Microsserviços e arquitetura baseada a eventos. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 29 nov. 2024.