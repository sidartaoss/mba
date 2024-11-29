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

