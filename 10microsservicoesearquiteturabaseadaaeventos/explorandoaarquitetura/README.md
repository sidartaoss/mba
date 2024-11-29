## Monolito clássico

- Um monolito clássico que consegue escalar.

![Monolito clássico e escalável](/10microsservicoesearquiteturabaseadaaeventos/imagens/monolito_classico_e_escalavel.png)

- Toda vez que estamos falando de um monolito clássico, trata-se de um sistema e, dentro desse sistema, nós temos diversos componentes:

    - Módulos dentro do mesmo sistema;

    - Classes dentro do mesmo sistema;

    - Tem tudo aquilo que faz com que o sistema funcione:

        - Normalmente, tem um banco de dados ou mais bancos de dados;

        - Normalmente, também tem um sistema de cache;

        - E, conforme a demanda de acesso for crescendo, é colocado um Load Balancer (ou API Gateway) na frente e o sistema passa a crescer horizontalmente.
  
- No caso em que se utilizar um Load Balancer (ou API Gateway):

    - O disco deve ser efêmero;

    - No momento em que for fazer o upload de um arquivo, não vai fazer upload no próprio disco da máquina em que estiver a aplicação, porque a aplicação pode morrer a qualquer momento; no momento em que essa máquina for removida, todos os dados armazenados na máquina também serão removidos. Nesse caso, pode-se subir em um cloud storage, s3, ftp, outro local que guarda arquivos, um local persistente de dados;

    - Os logs devem ser uploadados. Tudo que tiver de logs na aplicação também não vão poder ficar armazenados na máquina, porque, se a máquina for apagada, vai-se perder os logs de tudo o que aconteceu;

    - Sessões dos usuários não podem ficar armazenadas em disco. Em um momento o usuário vai estar autenticado, em outro momento, não: quando a máquina morrer, vai perder a sessão e o controle de autenticação do usuário;

    - Não armazenar estado;

    - Regra: o software deve estar pronto para ser criado e recriado sem nenhuma perda de dados;
        
        - Se o comportamento da aplicação mudar pelo fato de você destruir a aplicação e subir de novo, significa que você está armazenando estado.
