# Exercícios – Testes Unitários com JUnit

## Objetivo

Os exercícios a seguir têm como objetivo praticar a elaboração de testes unitários com **JUnit**, relacionando a implementação dos testes às técnicas estudadas anteriormente.

Ao desenvolver cada atividade, analise o comportamento da classe e defina, de forma justificada, um conjunto de casos de teste adequado.

Considere, sempre que aplicável:

- particionamento de equivalência;
- análise de valor limite;
- entradas válidas e inválidas;
- exceções;
- mudanças de estado;
- cobertura estrutural;
- testes parametrizados.

A escolha das anotações, assertivas e demais recursos do JUnit deve ser feita de acordo com as necessidades identificadas em cada problema.

---

# Exercício 1 – Calculadora de Desconto

Considere a classe abaixo:

```java
public class CalculadoraDesconto {

    public double calcular(double valorCompra, double percentual) {

        if (valorCompra < 0) {
            throw new IllegalArgumentException(
                    "Valor da compra não pode ser negativo");
        }

        if (percentual < 0 || percentual > 100) {
            throw new IllegalArgumentException(
                    "Percentual deve estar entre 0 e 100");
        }

        return valorCompra - (valorCompra * percentual / 100);
    }
}
```

## Atividade

Desenvolva testes unitários com JUnit para verificar o comportamento do método `calcular()`.

Antes da implementação:

1. analise o domínio de entrada do método;
2. identifique as classes de equivalência relevantes;
3. identifique os valores-limite;
4. defina um conjunto de casos de teste que considere adequado.

Em seguida:

- implemente os casos de teste;
- execute a suíte de testes;
- analise a cobertura obtida;
- avalie se o conjunto criado é suficiente para verificar o comportamento esperado do método.

Justifique brevemente as escolhas realizadas.

---

# Exercício 2 – Conta Bancária

Considere a classe abaixo:

```java
public class ContaBancaria {

    private double saldo;

    public ContaBancaria(double saldoInicial) {
        if (saldoInicial < 0) {
            throw new IllegalArgumentException(
                    "Saldo inicial não pode ser negativo");
        }

        saldo = saldoInicial;
    }

    public void depositar(double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException(
                    "Valor do depósito deve ser positivo");
        }

        saldo += valor;
    }

    public void sacar(double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException(
                    "Valor do saque deve ser positivo");
        }

        if (valor > saldo) {
            throw new IllegalArgumentException(
                    "Saldo insuficiente");
        }

        saldo -= valor;
    }

    public double getSaldo() {
        return saldo;
    }
}
```

## Atividade

Desenvolva uma classe de testes unitários para verificar o comportamento de `ContaBancaria`.

Antes da implementação:

1. analise as regras presentes no construtor e nos métodos `depositar()` e `sacar()`;
2. identifique situações válidas, inválidas e de limite;
3. determine quais mudanças de estado devem ser verificadas;
4. defina um conjunto de casos de teste que considere adequado.

Durante a implementação, avalie quando é interessante utilizar recursos de preparação de testes, testes parametrizados e verificação de exceções.

Após executar a suíte:

- analise a cobertura obtida;
- verifique se os principais comportamentos e regras da classe foram exercitados;
- justifique eventuais casos adicionais necessários.

---

# Exercício 3 – Sistema de Reserva de Hotel

Considere a classe abaixo:

```java
public class ReservaHotel {

    private int numeroDiarias;
    private double valorDiaria;
    private boolean cafeDaManha;
    private boolean cancelada;

    public ReservaHotel(int numeroDiarias, double valorDiaria) {

        if (numeroDiarias <= 0) {
            throw new IllegalArgumentException(
                    "Número de diárias deve ser maior que zero");
        }

        if (valorDiaria <= 0) {
            throw new IllegalArgumentException(
                    "Valor da diária deve ser maior que zero");
        }

        this.numeroDiarias = numeroDiarias;
        this.valorDiaria = valorDiaria;
        this.cancelada = false;
    }

    public void adicionarCafeDaManha() {

        if (cancelada) {
            throw new IllegalStateException(
                    "Não é possível alterar uma reserva cancelada");
        }

        cafeDaManha = true;
    }

    public double calcularValorTotal() {

        if (cancelada) {
            return 0;
        }

        double total = numeroDiarias * valorDiaria;

        if (cafeDaManha) {
            total += numeroDiarias * 30;
        }

        if (numeroDiarias >= 7) {
            total = total * 0.90;
        }

        return total;
    }

    public void cancelar() {

        if (cancelada) {
            throw new IllegalStateException(
                    "Reserva já está cancelada");
        }

        cancelada = true;
    }

    public boolean isCancelada() {
        return cancelada;
    }
}
```

## Atividade

Desenvolva testes unitários com JUnit para verificar o comportamento da classe `ReservaHotel`.

Antes da implementação:

1. identifique as regras de negócio presentes na classe;
2. determine as classes de equivalência e os valores-limite relevantes;
3. identifique os estados possíveis da reserva e as operações permitidas em cada estado;
4. defina um conjunto de casos de teste capaz de exercitar esses comportamentos.

Na implementação, procure organizar os testes de forma clara e evitar repetições desnecessárias.

Ao final:

- execute a suíte de testes;
- analise a cobertura obtida;
- identifique eventuais comportamentos não exercitados;
- avalie se a cobertura alcançada representa, de fato, um conjunto de testes adequado.

Justifique as decisões tomadas na definição dos casos de teste.

---

# Entrega

Para cada exercício, entregue:

1. a classe de teste implementada;
2. uma breve descrição dos critérios utilizados para definir os casos de teste;
3. o resultado da execução da suíte;
4. a cobertura obtida;
5. uma breve avaliação sobre a suficiência do conjunto de testes criado.
