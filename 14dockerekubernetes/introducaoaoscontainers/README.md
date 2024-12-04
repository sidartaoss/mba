# Introdução aos containers

## O que é virtualização

``
https://github.com/devfullcycle/mba-docker/
``

01 - O que é virtualização

A virtualização surgiu na década de 60, com o objetivo de compartilhar recursos de hardware entre vários usuários. 

A idéia era que cada usuário tivesse a sensação de que o hardware era dedicado para ele, mas na verdade, o hardware era compartilhado entre vários usuários.

O que é hypervisor:

- A virtualização de hardware é feita através de um software chamado de hypervisor, que é responsável por criar e gerenciar as máquinas virtuais.

- Existem dois tipos de hypervisor:

    - Tipo 1: O hypervisor é instalado diretamente no hardware e é responsável por criar e gerenciar as máquinas virtuais.

    - Tipo 2: O hypervisor é instalado sobre um sistema operacional e é responsável por criar e gerenciar as máquinas virtuais.

- O conceito de hypervisor vem de supervisor, que é um software que gerencia o hardware.

- O hypervisor é um supervisor de um supervisor, ou seja, é um software que gerencia um software que gerencia o hardware.

- Este termo foi usado pela primeira vez em 1966 pela IBM por R Adair, R Bayles no paper "A Virtual Machine System for the 360/40".

Vantagens (por que utilizar):

- A virtualização de hardware é muito utilizada em ambientes corporativos, pois permite que um único servidor seja dividido em vários servidores virtuais, o que reduz o custo de aquisição de hardware.

Quando surgiu:

- Na década de 60, a IBM desenvolveu o CP-40, a primeira versão do CP/CMS, que foi o primeiro sistema operacional a implementar a virtualização de hardware.

- O CP/CMS foi o precursor do VM/CMS, que foi o primeiro sistema operacional a implementar a virtualização de hardware em larga escala.

## Hypervisor

O que é:

- Um pedaço de software que permite que um computador hospede vários sistemas operacionais ao mesmo tempo.

- O hypervisor é instalado diretamente no hardware ou sobre um sistema operacional e é responsável por criar e gerenciar as máquinas virtuais.

![Hypervisor](/14dockerekubernetes/imagens/hypervisor-vmm.webp)
<p align="left">Fonte: Full Cycle, 2024.</p>

## Virtual machine monitor

O que é:

- Quando o hardware não está disponível diretamente na máquina virtual, o Virtual Machine Monitor (VMM) engana o sistema operacional convidado para que ele acredite que está sendo executado diretamente no hardware.

- O VMM intercepta as instruções do sistema operacional convidado e as traduz para o hardware real.

A VMM precisa satisfazer 3 propriedades:

- Equivalência: Precisa ser o mesmo, com ou sem a virtualização. Isto significa que a maioria das instruções precisam ser executadas sem nenhuma tradução.

- Performance: Precisa ser rápido o suficiente para não degradar o desempenho do sistema operacional convidado.

- Isolamento: Precisa isolar os convidados uns dos outros.

## Virtualização de memória

O que é:

- A virtualização de memória é uma técnica que permite que um sistema operacional convidado acesse a memória física do sistema.

- O sistema operacional convidado acredita que está acessando a memória física, mas na verdade está acessando a memória virtual.

Algumas técnicas de virtualização de memória:

- Guest Virtual Memory: É o que o processo rodando consegue ver.

- Guest Physical Memory: É o que o SO convidado consegue ver.

- System Physical Memory: É o que o a VMM consegue ver.

## Virtualização de cpu

O que é:

- A virtualização de CPU é uma técnica que permite que um sistema operacional convidado acesse a CPU do sistema.

- O sistema operacional convidado acredita que está acessando a CPU, mas na verdade está acessando a CPU virtual.

## Binary Translation

O que é:

- O SO convidado é usado sem nenhuma modificação.

- As instruções do SO convidado são interceptadas e traduzidas para o hardware real.

Desvantagem:

- Isso causa uma degradação no desempenho.

## Paravirtualização

O que é:

- Para resolver problemas de performance, o SO convidado é modificado para que ele saiba que está rodando em um ambiente virtualizado.

- A interação do SO convidado com o host é otimizado para evitar a degradação de desempenho.

Fontes:

- Livro: Linux Containers and Virtualization (Shashank Mohan Jain)
- https://en.wikipedia.org/wiki/Timeline_of_virtualization_development
- https://en.wikipedia.org/wiki/Hypervisor
- https://softwareengineering.stackexchange.com/questions/196405/how-did-the-term-hypervisor-come-into-use

## Conteiners versus virtualização

Precisamos entender, de fato, o que é virtualização, porque é muito confundido com containers.

Eles estão relacionados, principalmente por conta do objetivo dois dois:

- Virtualizar, rodar vários tipos de software isolados, dentro do mesmo hardware.

Mas, a forma que virtualização faz e a forma que container faz são diferentes.

## Virtualização

O que é:

- Um sistema operacional rodando dentro do outro sistema operacional. 

- Esse sistema operacional pode ser totalmente diferente do sistema operacional original. Podemos ter um Windows que roda Windows 10, que vai virtualizar um outro Windows 10 ou pode virtualizar um Mac ou um Linux ou um Windows XP ou outro sistema operacional totalmente diferente.

- Pode-se rodar vários sistemas operacionais diferentes, chamados de máquinas virtuais, dentro do nosso sistema operacional.

- O sistema operacional base nós chamamos de host (hospedeiro) e a máquina virtual é o sistema operacional convidado. Podemos ter máquinas virtuais diferentes e elas estão totalmente isoladas.

Por que queremos virtualizar sistemas operacionais?

- No contexto de um data center, a idéia é compartilhar os recursos de hardware com vários usuários e empresas diferentes.

- Se não tivéssemos virtualização, para cada usuário e para cada empresa que desejasse utilizar um servidor, fosse necessário criar um servidor físico totalmente diferente para ela, isso iria tornar o uso de recursos computacionais muito caros.

- Essa mesma idéia, que é compartilhada com a idéia de cloud, já vem da década de 60, permitindo que vários usuários e empresas compartilhem os recursos de hardware, usando um sistema operacional dentro de outro, mas, para o usuário e a empresa, tem a noção de que se trata de um servidor físico, enquanto que, na verdade, a máquina está sendo alugada para esses usuários e empresas, permitindo que haja um sistema operacional rodando um outro sistema operacional totalmente diferente.

- Na década de 60, a pioneira de virtualização foi a IBM, com o CP/CMS, sendo o primeiro sistema operacional a implementar a idéia de virtualização, com o objetivo de utilizar em larga escala, permitindo compartilhar recursos de hardware entre vários usuários diferentes.

## Hypervisor

- Hypervisor vai ser a camada que vamos ter no hardware ou no sistema operacional que vai permitir a criação de máquinas virtuais diferentes.

- Podemos ter um hypervisor instalado diretamente no hardware, sendo para situações específicas, que, normalmente, é bem mais corporativo.

- Um hypervisor instalado no sistema operacional é aquele que podemos baixar a qualquer momento: um VirtualBox, um VMWare ou Hyper-V da Microsoft e, aí, podemos gerar novos sistemas operacionais dentro deles.

Cenário:

- Temos o hardware.

- Temos o sistema operacional embaixo do hypervisor ou ele diretamente no hardware. Mas, será ele quem vai gerenciar todas as máquinas virtuais que vão sendo criadas, disponibilizadas para os usuários, sendo uma máquina virtual totalmente isolada uma da outra.

- A virtual machine monitor (vmm) é uma camada que vai fazer com que o sistema operacional seja enganado. O sistema operacional vai achar que está utilizando o hardware, rede, cpu, mas, na verdade, essas coisas estão sendo virtualizadas para ele para a gente poder alocar somente o que é necessário, o que está compartilhando em termos de recursos.

## Paravirtualização

- Ao longo dos anos, tivemos novas necessidades para poder virtualizar sistemas, a computação foi evoluindo, a capacidade das máquinas também foi evoluindo e nós tivemos um outro conceito sendo adicionado que é a paravirtualização.

- A paravirtualização é uma virtualização mais eficiente. A idéia da paravirtualização é diferente da virtualização convencional, em que deixamos transparente para o sistema convidado que ele está sendo virtualizado e, então, ele trabalha mais em conjunto com o sistema hospedeiro, que é o sistema operacional que está, de fato, rodando no disco.

- Dessa forma, conseguimos mais eficiência, porque não há o overhead inicial, pois, quando subimos uma máquina virutal convencional, ele sequestra da sua máquina CPU e memória.

- Então, a paravirtualização consegue ir utilizando os recursos e, até, ir devolvendo conforme for necessário - ela trabalha em conjunto com o sistema operacional hospedeiro.

- Um exemplo que podemos citar é a WSL do Windows, que é o Linux rodando dentro do Windows. Você inicia o seu Linux dentro do Windows e ele não aloca um monte de CPU e memória; ele vai utilizando o que é necessário, vai devolvendo, vai utilizando e trabalhando em conjunto.

- Então, nós temos alguns tipos de virtualização que vão ser adequados a certas situações, mas, de uma forma geral, conseguimos virtualizar tudo, hoje em dia. Nós podemos virtualizar memória, CPU, rede e a máquina virtual pode imaginar que está utilizando hardware físico, mas, no final das contas, ela não está utilizando.

- É importante entender a idéia de que nós queremos virtualizar, porque desejamos permitir que usuários diferentes compartilhem os recursos da máquina, do servidor.

Cloud:

- Com o surgimento e evolução de cloud, nós temos as máquinas virtuais as quais nós podemos alugar da AWS, Azure, etc. Essa máquina virtual que estamos alugando é uma máquina virtualizada dentro de um servidor que tem recursos muito maiores, mas estamos alugando apenas um pouco de CPU, um pouco de memória, com Windows, Linux, etc., para poder fazer uso. Mas lembrando que estamos em uma máquina virtual que também está rodando junto com outras máquinas virtuais, então, estamos compartilhando aquele servidor com outras empresas e usuários.

- Sem virtualização, dificilmente teríamos cloud. O uso de recursos seria muito mais burocrático e muito mais caro, também. Sempre teríamos que ter um servidor físico para poder trabalhar.

## O que são containers e a diferença para máquinas virtuais

![Containers versus máquinas virtuais](/14dockerekubernetes/imagens/containers_versus_maquinas_virtuais.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Do lado direito, a arquitetura de virtualização, que tem muita relação com hypervisor.

Do lado esquerdo, containers.

Máquinas virtuais precisam de um hypervisor, que vai ser um software que vai estar rodando em cima do sistema operacional, que vai estar gerenciando o hardware.

Cada máquina virtual é um sistema operacional completo.

Se temos um Windows que está virtualizando Linux, cada máquina virtual vai ser um Linux completo, com kernel, com tudo mais, para podermos rodar a nossa aplicação Java ou a nossa aplicação Python:

- O custo disso é muito alto: para poder compartilhar os recursos de hardware para poder rodar aplicações diferentes, é necessário subir uma máquina inteira.

- Então, por conta disso, surgiram os containers.

``
A idéia de um container é ser apenas um processo que vai compartilhar o sistema operacional com os outros processos.
``

Então, nós não temos um sistema operacional novo sendo emulado aqui.

- Uma diferença crucial de container para máquina virtual é que, se tivermos um Windows, nós podemos rodar containers Windows apenas - não podemos rodar containers Linux.

No Windows, é possível rodar um container Linux, porque estamos virtualizando o WSL: é uma máquina virtual - como estamos rodando o Linux, é possível rodar um container Linux.

Mas, tirando essa exceção, o container é feito para rodar no seu sistema operacional hospedeiro. Ele é apenas um processo, que compartilha os recursos.

Em alguns cenários, faz sentido ter máquinas que sejam capazes de emular um sistema operacional inteiro, que vão ter necessidades de indústria e de mercado que vão exigir isso.

Mas, boa parte das aplicações, hoje em dia, só vão rodar de forma isolada dentro de um Linux ou dentro do próprio Windows, pois temos os containers Windows. 

E isso é mais eficiente: 

- Podemos ter várias empresas compartilhando os recursos de hardware com os seus containers totalmente isolados uns dos outros, mas eles estão utilizando o kernel.

Em termos de desvantagens ao utilizar-se máquinas virtuais, temos performance:

- Uma máquina virtual precisa de um hypervisor para funcionar, o que causa uma degradação de desempenho. 

- Uma máquina virtual precisa de um sistema operacional completo: gasta muito mais recursos. 

## Chroot e namespaces

- O comando chroot significa change root.

- userland corresponde aos diretórios e sistema de arquivos comuns no Linux.

Vamos usar o chroot (change root) para fazer com que essa seja a raiz que gente vai trabalhar, como se estivesse em uma partição do Linux.

Comando:

- ps aux: para ver os processos.

- ps aux | grep python

- Conseguimos acessar as informações dos processos no diretório /proc.

## Cgroups

https://github.com/devfullcycle/mba-docker/blob/main/cap%2001%20-%20introdu%C3%A7%C3%A3o%20aos%20containers/04-cgroups.md

Comando:

- systemctl status: árvore de slices. Todo processo vai estar aliado a um cgroup. O slice é uma forma de a gente delimitar isso. Podemos ter um slice que pode herdar de outro slice, acabando em um árvore de slices.

Todo processo do Linux está dentro de um cgroup.

Comando:

- htop: monitorar em tempo real processos e recursos.

O entendimento do conceito de container de que ele é um processo, não tem virtualização por trás, que ele está compartilhando o kernel e ele está em um ambiente isolado rodando na mesma máquina é crucial.

## História do docker, oci e lxc

https://github.com/devfullcycle/mba-docker/blob/main/cap%2002%20-%20Mundo%20de%20Containers%20e%20Docker/01-Hist%C3%B3ria%20do%20Docker%2C%20OCI%20e%20LXC.md

Containers são processos que estão sendo enganados, compartilham os recursos do sistema operacional; são apenas processos, mas estão sendo feitos de bobo, ele está achando que está com o sistema operacional completo, mas ele está em um ambiente que é o namespace e limitado também, sendo limitado pelo cgroup.

Containerd:

- Um serviço que vai ficar rodando em background na máquina e vai gerenciar a execução dos containers.

- Consegue-se saber informações dos containers através de API's.

- Projeto incubado na CNCF.


## Rodando um container sem docker com runc


https://github.com/devfullcycle/mba-docker/blob/main/cap%2002%20-%20Mundo%20de%20Containers%20e%20Docker/02-Rodando%20um%20container%20sem%20Docker%20com%20runc.md


O que é runc:


Userland:

- Ao desejar-se prover um container, é como se estivéssemos provendo um sistema operacional inteiro.

- Vamos utilizar a userland, porque sempre que pegarmos um container e, quando formos trabalhar com Docker, nós vamos baixar uma imagem e vai estar lá a userland lá dentro. Às vezes, essa userland é mais focada em um Alpine ou em outro Linux qualquer, mas sempre teremos as pastas as quais estamos acostumados a ver no Linux.

- runc spec gera um arquivo config.json. Esse arquivo podemos definir como se fosse de como que esse container tem que ser criado, o padrão de mercado, que está na OCI, na Open Container Initiative.

Linux capabilities: permissões do que o container vai ter ou não em cima do Linux.

Com runc, nós podemos fazer várias coisas com containers, poderia executar os containers sem ter que instalar Docker na nossa máquina, mas a gente vai entender os benefícios de utilizar o Docker.

## Quais as vantagens de utilizar o docker

https://github.com/devfullcycle/mba-docker/blob/main/cap%2002%20-%20Mundo%20de%20Containers%20e%20Docker/03-Quais%20as%20vantagens%20de%20usar%20o%20Docker.md

## Docker hub

https://github.com/devfullcycle/mba-docker/blob/main/cap%2002%20-%20Mundo%20de%20Containers%20e%20Docker/04-Docker%20Hub.md

O rootfs, o qual baixamos de algum lugar da internet, pode ser comparado a uma imagem.

A imagem vai ter um monte de arquivos. Quando esse container for criado, ele já é criado naquele userland. Então, podemos ter uma imagem que já vem com o MySQL, PHP, Java, Python, Node.js, etc., já instalado e já podemos sair usando ali, diretamente.

A imagem já contém uma série de ferramentas que já podem ser utilizadas para rodar a aplicação.

- Verificação de segurança: contém selos de auto-verificado. Sabemos que a imagem está vindo diretamente do fornecedor, por exemplo, Oracle: temos mais confiabilidade.

- Builds automáticos: toda vez que subir alguma coisa para o repositório do GitHub, vai ser criado uma imagem no DockerHub de forma automática, taggear, etc.

- Webhooks: no caso de termos alguma ferramenta que tem que ser notificada quando estiver disponível uma nova imagem no DockerHub. Então, os webhooks fazem essa notificação.

- Podemos criar o nosso repositório de imagens na cloud, na máquina local, ou em qualquer outro lugar. Mas a questão é que a Docker está fornecendo um projeto open source com uma série de coisas prontas para poder utilizar. E esse é o motivo de o DockerHub ser tão popular. Inclusive, a ferramenta que popularizou de fato o Docker é o DockerHub, pois, se fosse fornecido apenas os executáveis para poder criar container, não haveria o ferramental adicional que o DockerHub oferece para facilitar o desenvolvimento.

## Arquitetura do docker

https://github.com/devfullcycle/mba-docker/blob/main/cap%2002%20-%20Mundo%20de%20Containers%20e%20Docker/05-Arquitetura%20do%20Docker.md


- Docker engine: é uma arquitetura cliente-servidor, no final das contas, pois temos os comandos sendo executados, que vai, bate no daemon, pedindo alguma informação ou para fazer alguma coisa.

![Arquitetura docker](/14dockerekubernetes/introducaoaoscontainers/README.md)
<p align="left">Fonte: Full Cycle, 2024.</p>




