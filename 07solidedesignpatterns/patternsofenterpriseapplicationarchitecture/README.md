# Patterns of Enterprise Application Architecture

## Layering (camadas)

## Separação de responsabilidades

Quando a gente está falando em layering, a gente está falando em separação de camadas, a gente está falando, no final do dia, em separação de responsabilidades.

A gente está pensando em uma forma de abstrair uma parte do sistema em relação a outra parte do sistema para que o sistema em si não tenha apenas uma única responsabilidade e todo o código misturado.

Se eu conseguir separar responsabilidades do meu sistema em partes que façam sentido, ou seja, camadas, vai ficar muito mais fácil eu conseguir olhar para ele e ver responsabilidades.

Mas, para isso, eu tenho que conseguir pensar em abstração. Por quê? Porque na hora em que uma chamada bate no meu sistema e essa chamada vai chamar uma camada, eu tenho que saber, naquele momento que, eu chamando aquela camada, não interessa o que está por trás dela.

## Layer versus tier

Camadas não são pastas.

Pasta é uma separação física.

Camada é uma separação lógica.

Toda vez que a gente está em separação de camadas ou layers, a gente está querendo dizer que é uma separação lógica. Então, eu tenho a minha aplicação, e essa minha aplicação tem pedaços lógicos, que fazem coisas diferentes; com responsabilidades diferentes.

Então, quando eu estou trabalhando com a parte lógica, eu posso ter uma parte que vai acessar banco de dados, eu posso ter uma parte que vai trabalhar com apresentação; eu tenho uma parte que vai cuidar da lógica de negócios.

Então, se você começa a olhar dessa forma, você começa a entender que a sua aplicação é dividida, fatiada por responsabilidades, onde uma camada chama a outra e a camada que está chamando não tem que entender o que está acontecendo por trás da outra, porque a única coisa que ela precisa entender é de abstração.

Ela sabe que, se ela chamar aquele método, ela vai receber um dado do banco de dados, e não interessa qual é o banco de dados, como aquilo foi chamado, qual é a query que foi executada, ela simplesmente chama e acabou.

Conceito de Tier: A Tier é a separação física. O que que isso significa? É aonde a responsabilidade é colocada. Então, a separação lógica é como que as coisas interagem. A separação física é aonde essas coisas vão interagir.

Tier: Significa que você tem pedaços físicos da sua aplicação em lugares diferentes, sejam eles em nodes (i.e., servidores) diferentes, sejam eles em pastas diferentes dentro da sua aplicação.

## 3 Layers architecture (arquitetura em 3 camadas)

- Presentation
	- Display information;
    - Retorno da informação em formatos diferentes;
	- Quando você está retornando a API REST, você está na camada de apresentação;
	- CLI - (Formato de apresentação CLI), AWS CLI, Google CLI, etc.
	- HTTP Requests - REST, formato de apresentação: todo HTTP request vai retornar em um formato específico, JSON, XML, etc.;
	- Apresentação não possui lógica, ela retorna dados, é uma camada burra, ela entrega a informação da maneira que ela é.

- Domain
	- Coração da aplicação
	- É a camada onde nós trazemos a resolução do problema que está sendo proposto;
	- Regras de negócio; é a camada que resolve o problema.

- Data Source
	- Banco de dados;
	- Camada em que vai ter acesso, onde recupera ou persiste dados;
	- Mensageria - também traz informações, ou recebe informações também, então, também faz parte da camada de Data Source.