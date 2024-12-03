# Introdução aos containers

## O que é virtualização

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


