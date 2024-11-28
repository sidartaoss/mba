# Design Patterns

## Rodrigo Branas

## SOLID

## SRP - Single Responsibility Principle

Devemos separar coisas que mudam por motivos diferentes.

## DIP - Dependency Inversion Principle

Componentes de alto nível não devem depender de componentes de baixo nível, eles devem depender de abstrações.

## OCP - Open/Closed Principle

Fechado para modificação e aberto para extensão

Crie pontos de extensão, evitando mexer no que já está funcionando e evitando fragilizar o código (Padrões Strategy, Decorator)

## LSP - Liskov Substituion Principle

Coerência da hierarquia, respeitar a invariância dos objetos, não implementar métodos que não precisa, não retornar valores incompatíveis com a superclasse.

## ISP - Interface Segregation Principle

Decompor em mais interfaces, menores, com menos funcionalidades, para que tenha a capacidade de estender só o que precisa.

## Patterns

## DTO - Data Transfer Object

Objeto que só tem propriedades, sendo utilizado para transporte entre camadas da aplicação.

## Repository

Realizar a persistência de aggregates (clusters de objetos de domínio como entities e value objects), separando essa responsabilidade da aplicação.

## Adapter

Converte a interface de uma classe em outra esperada pelo cliente, permitindo que classes incompatíveis trabalhem juntas.

## Strategy

Criar comportamento intercambiável.

## Dynamic Factory

Criar uma instância com base em uma string.