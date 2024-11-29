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

Aplicações comuns com propósitos e escopo bem definidos.

- Microsserviço é um sistema como outro qualquer. Quando se cria um monolito e quando se cria um microsserviço, basicamente, está-se criando a mesma coisa; a diferença é o tamanho, o nível de responsabilidade do sistema. Enquanto os monolitos têm diversos escopos, diversas responsabilidades, diversos motivos para mudar, os microsserviços, normalmente, têm apenas um motivo para mudança.

- Independentes e autônomos:

    - Criar mecanismos de fallback: caso o microsserviço que você dependa diretamente estiver fora do ar, você vai degradar a entrega que você vai dar para o usuário final.

    - Exemplo: Mostrar saldo do usuário. No momento em que vai mostrar o saldo do usuário, o microsserviço que gera toda a conta para trazer o saldo está fora do ar. Para não entregar uma experiência ruim para o usuário, como um erro 500, você pode recuperar o saldo que foi entregue pela última vez e mostrar para o usuário: pelo menos, conseguiu-se gerar valor para o usuário final. Chama-se `gracefull degradation`. Para informações críticas, é necessário ter essa estratégia para não estourar um erro na cara do cliente.

- Participam de um ecossistema maior. Uma parte da engrenagem.


## Arquitetura baseada em microsserviços

- Possibilidade de múltiplas tecnologias;

    - Considerando política de governança.

- Processo de deploy menos arriscado;

    - 

