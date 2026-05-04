

# 🛠️ Apresentação: Pure Fabrication (Fabricação Pura)
## Instituto Federal de Educação e Ciência e Tecnologia de Mato Grosso
## Alunos: Nagafe de Oliveira Martins, Guilherme P. Silva
## Disciplina: Projeto de Software
## Professor: Orlando


## 📌 1. O que é Pure Fabrication?
O **Pure Fabrication** (Fabricação Pura) é um padrão de atribuição de responsabilidades do GRASP(General Responsibility Assignment Software Patterns) focado na criação de classes técnicas altamente especializadas. 

Em vez de sobrecarregar as entidades centrais do sistema com múltiplas tarefas, o desenvolvedor "fabrica" (cria) novas classes cujo único propósito é agrupar comportamentos sistêmicos, de infraestrutura ou lógicas complexas de coordenação. Como essas classes não representam conceitos do "mundo real" do negócio, elas são chamadas de Fabricações Puras.

## ⚖️ 2. O Grande Dilema: O Embate com o "Information Expert"
Na teoria GRASP, a regra de ouro é o **Information Expert**: "A classe que tem a informação deve executar a ação".

**O Problema:** Se aplicarmos o *Expert* cegamente, destruímos a arquitetura. Se a classe `Pedido` possui os dados da venda, ela deveria se salvar no banco de dados. Mas fazer isso fere o Princípio da Responsabilidade Única (SRP), misturando regra de negócio com infraestrutura (SQL, conexões).

**A Solução:** Quando o *Information Expert* falha em manter o design limpo, nós usamos a "válvula de escape": o **Pure Fabrication**.

---

## 💻 3. Como é aplicado na prática? (Exemplos em Java)

A aplicação ocorre através da **delegação**. Veja a diferença estrutural entre um código engessado e um código que utiliza o padrão:


```
// 🚨 ERRO: Violação do Princípio da Responsabilidade Única (SRP)
public class Pedido {
    private String cliente;
    private double valorTotal;

    public void aplicarDesconto(double percentual) {
        this.valorTotal -= (this.valorTotal * percentual); // ✅ Coeso ao negócio
    }

    // 🚨 Misturando domínio com infraestrutura
    // Alta acoplagem com o banco e serviço de e-mail!
    public void salvarNoBancoDeDados() {
        System.out.println("Conectando ao banco MySQL...");
        System.out.println("Executando INSERT INTO pedidos...");
    }
}
```
### ✅ O Design Limpo (Com Pure Fabrication)
A classe de domínio fica isolada, e inventamos classes novas apenas para lidar com a infraestrutura.

**O Domínio:**
```
// 🟢 Domínio limpo e focado. Não sabe o que é Banco de Dados ou Internet.
public class Pedido {
    private String cliente;
    private double valorTotal;

    public void aplicarDesconto(double percentual) {
        this.valorTotal -= (this.valorTotal * percentual);
    }
}
```


**As Fabricações Puras (Pure Fabrications):**
```
// 🟢 PURE FABRICATION 1: Repositório (Lidando com Persistência)
public class PedidoRepository {
    public void salvar(Pedido pedido) {
        System.out.println("[Banco de Dados] Abrindo conexão e salvando pedido...");
    }
}
```
```
// 🟢 PURE FABRICATION 2: Serviço (Lidando com Integrações)
public class EmailNotificationService {
    private String servidorSmtp = "smtp.servidor.com";

    public void enviarRecibo(Pedido pedido, String emailDestino) {
        System.out.println("[E-mail] Autenticando no servidor " + servidorSmtp);
        System.out.println("[E-mail] Enviando recibo para: " + emailDestino);
    }
}
```

---

## 🔗 4. Conexão com outros padrões

O Pure Fabrication é a engrenagem que faz outros princípios funcionarem na prática:
* **SRP do SOLID (Responsabilidade Única):** Garante que a classe `Pedido` mude apenas se a regra de negócio mudar, e a classe `EmailNotificationService` mude apenas se o servidor SMTP mudar.
* **Apoio à Alta Coesão:** Garante que cada classe faça apenas uma coisa e faça bem feito.
* **Garantia de Baixo Acoplamento:** Evita que as regras de negócio fiquem engessadas a bibliotecas externas, frameworks ou APIs.

## 🚫 5. Restrições (Quando NÃO usar)
Este padrão **não deve** ser usado quando:
1. **Causar um Modelo de Domínio Anêmico:** Se você fabricar classes (Serviços) para calcular regras de negócio básicas que deveriam estar dentro das próprias entidades, suas classes principais se tornarão meros recipientes de *getters/setters*.

2. **O *Information Expert* for suficiente:** Se a tarefa envolve apenas manipular o estado interno da própria classe sem depender de recursos externos (ex: calcular o total do carrinho), deixe o método na própria classe.


**Exemplo 1**
```
// Entidade Anêmica (só tem dados, não tem comportamento)
public class Carrinho {
    public List<Double> precosDosItens = new ArrayList<>();
}

// 🚨 Fabricação Pura Desnecessária! 
// Roubou a regra de negócio que pertencia ao Carrinho.
public class CalculadoraDeCarrinhoService {
    public double calcularTotal(Carrinho carrinho) {
        double total = 0;
        for(Double preco : carrinho.precosDosItens) {
            total += preco;
        }
        return total;
    }
}
```

```
public class Carrinho {
    private List<Double> precosDosItens = new ArrayList<>();

    // O Carrinho é o Especialista da Informação aqui. Não precisamos de Serviços.
    public double calcularTotal() {
        double total = 0;
        for(Double preco : this.precosDosItens) {
            total += preco;
        }
        return total;
    }
}
```
**Exemplo 2**

```
public class Cliente {
    public String nome;
    public String sobrenome;
}

// 🚨 Fabricação Pura Desnecessária!
// Não há complexidade técnica de infraestrutura que justifique criar essa classe.
public class FormatadorDeNomeDeCliente {
    public String gerarNomeCompleto(Cliente cliente) {
        return cliente.nome + " " + cliente.sobrenome;
    }
}
```

```
public class Cliente {
    private String nome;
    private String sobrenome;

    // A própria classe resolve o problema sem ferir regras de arquitetura.
    public String getNomeCompleto() {
        return this.nome + " " + this.sobrenome;
    }
}
```
---

## 🔎 6. Como identificar um Pure Fabrication em um projeto real?

Para exemplificar, vamos analisar a estrutura de código de um projeto real em Java. Acesse o diretório base do projeto **OrcamentoPessoal** no link abaixo:

🔗 **[Repositório: OrcamentoPessoal - Pacote Base](https://github.com/GuilhermePereiraS/OrcamentoPessoal/tree/main/src/main/java/br/com/porquinho)**

Ao navegar pelas pastas desse projeto (ou de qualquer outro sistema corporativo), você pode identificar rapidamente quais arquivos são **Pure Fabrications** aplicando este *checklist* de 4 passos:

 1. O Teste do "Mundo Real" (Vocabulário)
Em um sistema de controle financeiro, entidades como `Despesa`, `Receita`, `Categoria` e `Usuario` representam o domínio. Elas "existem" na vida real. 
Se você abrir uma pasta e esbarrar em um arquivo como `DespesaRepository` ou `AutenticacaoService`, pergunte-se: *"Isso existe no mundo real das finanças?"*. Como a resposta é **não**, você acabou de encontrar uma Invenção Pura.

 2. A Regra dos Sufixos
Por convenção da engenharia de software, desenvolvedores nomeiam essas classes artificiais com sufixos técnicos para facilitar a identificação. Procure por arquivos que terminam com:
* `...Repository` ou `...DAO` (Fabricados para acessar o banco de dados)
* `...Service` (Fabricados para orquestrar lógicas complexas e integrações)
* `...Controller` (Fabricados para receber requisições de telas ou APIs)
* `...Mapper` (Fabricados para converter formatos de dados)

 3. Olhe os "Imports" (Infraestrutura)
Se você abrir o arquivo e olhar o topo dele, uma classe de domínio terá importações simples (como `java.util.List` ou `java.time.LocalDate`). 
Já um *Pure Fabrication* costuma ter importações pesadas de infraestrutura e frameworks, como bibliotecas de SQL (`java.sql.*`), anotações do Spring Boot (`org.springframework.*`) ou bibliotecas de rede.

 4. Análise de Atributos (Estado vs Dependências)
* **A Classe de Domínio:** Guarda o *estado* do negócio. Seus atributos são informações reais (ex: `String descricao`, `double valor`).
* **A Classe Fabricada:** Guarda *dependências*. Seus atributos geralmente são ferramentas ou outras classes fabricadas (ex: `DatabaseConnection db`, `EmailSender sender`).

## 📚 7. Fontes e Referências

* **Refactoring Guru - O que é um Padrão de Projeto?**
  [https://refactoring.guru/pt-br/design-patterns/what-is-pattern](https://refactoring.guru/pt-br/design-patterns/what-is-pattern)
* **Medium - Pure Fabrication in Software Design (por Safonov B2C):**
  [https://medium.com/@safonovb2c/pure-fabrication-in-software-design-bd4c7ce34bf7](https://medium.com/@safonovb2c/pure-fabrication-in-software-design-bd4c7ce34bf7)

### ❌ A armadilha do Information Expert (Sem Pure Fabrication)
A classe tenta fazer tudo sozinha, apenas porque "possui os dados".
