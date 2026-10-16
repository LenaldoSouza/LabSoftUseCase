# 🚀🚀 Guia Prático: Design Patterns C# - IMPLEMENTAÇÃO DO PATTERN PROTOTYPE NA CLÍNICA MÉDICA

## 📌 Contexto do Projeto: Sistema Interno da Clínica "Vida & Saúde"

Em sistemas de gestão hospitalar e clínica, é comum a criação recorrente de objetos complexos com configurações padronizadas. Criar esses objetos do zero instanciando todas as propriedades repetidamente pode gerar código redundante e gargalos de processamento.

Nesta atividade, você desenvolverá uma aplicação Console em C# (.NET 9) que aplica o padrão de projeto criacional Prototype. O objetivo é permitir que a clínica clone modelos pré-configurados de exames, prontuários e agendamentos recorrentes sem reconstruí-los manualmente.

---

## 💼 Regra de Negócio: O Problema dos Check-ups e Protocolos Médicos
A clínica "Vida & Saúde" oferece pacotes de exames padronizados (ex: Check-up Cardiológico Geral, Check-up Admissional, Rotina Pediatria). Cada pacote possui um conjunto fixo de procedimentos, orientações prévias e valores base.

1. Agilidade no Atendimento: Em vez de configurar individualmente cada solicitação de exame com suas dezenas de instruções (ex: "Jejum de 8h", "Chegar 15 min antes"), o 2. atendente deve clona um Protótipo de Pacote de Exame já cadastrado.

2. Customização Individual: Após a clonagem, o atendente pode personalizar os dados do paciente, médico solicitante e data/hora sem alterar o pacote modelo original.

3. Cópia Superficial vs. Cópia Profunda (Shallow vs. Deep Copy): As consultas contêm objetos aninhados (como o Paciente e a Lista de Procedimentos). Precisamos garantir3 que alterar a lista de procedimentos de um paciente clonado não altere o modelo base e vice-versa

---

## 🛠️ Pré-requisitos

- Visual Studio 2022 ou VS Code (com .NET 9 SDK instalado)
- Conhecimentos básicos de POO em C# (Classes, Interfaces e Enums

---

## 🎯 Objetivos de Aprendizagem
1. **Compreender o Pattern Prototype: Entender quando e por que clonar objetos existentes.
2. **Diferenciar Cópia Superficial e Profunda: Dominar a diferença técnica entre Shallow Copy e Deep Copy.
3. **Implementar a Interface de Clonagem: Utilizar contratos explícitos para padronizar o método Clonar().
4. **Gerenciar Protótipos em Memória: Criar um registro centralizado (Prototype Registry) para disponibilizar os modelos da clínica

---

## 📋 Atividade Prática: Passo a Passo

---

### 📂 Etapa 1: Definindo o Contrato do Protótipo e Classes de Apoio

#### Subtarefa 1.1: Criar a interface IPrototipo<T>
Crie o arquivo IPrototipo.cs na pasta Interfaces. Utilizaremos uma interface genérica para garantir tipagem forte na clonagem.

```csharp
namespace ClinicaPOO.Interfaces
{
    public interface IPrototipo<T>
    {
        T Clonar();     // Cópia Profunda (Deep Copy)
        T ClonarRaso(); // Cópia Superficial (Shallow Copy)
    }
}

```

#### Subtarefa 1.2: Criar a classe Procedimento
Crie o arquivo Procedimento.cs na pasta Models. Representa um item do pacote de exames.

```csharp
namespace ClinicaPOO.Models
{
    public class Procedimento : IPrototipo<Procedimento>
    {
        public string Nome { get; set; } = string.Empty;
        public decimal Valor { get; set; }

        public Procedimento(string nome, decimal valor)
        {
            Nome = nome;
            Valor = valor;
        }

        // Cópia para tipos por valor (structs/strings imutáveis)
        public Procedimento Clonar()
        {
            return (Procedimento)MemberwiseClone();
        }

        public Procedimento ClonarRaso()
        {
            return Clonar();
        }
    }
}
```

### 📂 Etapa 2: Implementando a Classe Complexa Prototipável (PacoteExame)

O PacoteExame contém referências para uma lista de procedimentos. Aqui aplicaremos o conceito de Deep Copy.


#### Subtarefa 2.1: Criar a classe PacoteExamel
Crie o arquivo PacoteExame.cs na pasta Models.

```csharp
using ClinicaPOO.Interfaces;

namespace ClinicaPOO.Models
{
    public class PacoteExame : IPrototipo<PacoteExame>
    {
        public string NomePacote { get; set; } = string.Empty;
        public string OrientaçõesJejum { get; set; } = string.Empty;
        public string NomePaciente { get; set; } = string.Empty;
        public List<Procedimento> Procedimentos { get; set; } = new List<Procedimento>();

        public PacoteExame() { }

        public PacoteExame(string nomePacote, string orientacoesJejum)
        {
            NomePacote = nomePacote;
            OrientaçõesJejum = orientacoesJejum;
        }

        public void AdicionarProcedimento(Procedimento proc)
        {
            Procedimentos.Add(proc);
        }

        // Cópia Superficial: Copia referências. A lista apontará para a MESMA memória do modelo original.
        public PacoteExame ClonarRaso()
        {
            return (PacoteExame)MemberwiseClone();
        }

        // Cópia Profunda: Duplica o objeto E sua lista de procedimentos interna.
        public PacoteExame Clonar()
        {
            var clone = (PacoteExame)MemberwiseClone();
            
            // Instancia uma nova lista e clona elemento por elemento
            clone.Procedimentos = new List<Procedimento>();
            foreach (var proc in this.Procedimentos)
            {
                clone.Procedimentos.Add(proc.Clonar());
            }

            return clone;
        }

        public void ExibirResumo()
        {
            Console.WriteLine($"\n[PACOTE]: {NomePacote} | Paciente: {(string.IsNullOrEmpty(NomePaciente) ? "NÃO ATRIBUÍDO" : NomePaciente)}");
            Console.WriteLine($"  Orientação: {OrientaçõesJejum}");
            Console.WriteLine("  Procedimentos inclusos:");
            decimal total = 0;
            foreach (var p in Procedimentos)
            {
                Console.WriteLine($"   - {p.Nome}: R$ {p.Valor:F2}");
                total += p.Valor;
            }
            Console.WriteLine($"  Total: R$ {total:F2}");
        }
    }
}
```

---

### 📂 Etapa 3: Gerenciador de Protótipos (Prototype Registry)

Para evitar a instanciação direta no código principal, usaremos um registro para armazenar os protótipos pré-configurados da clínica

#### Subtarefa 3.1: Criar a classe GerenciadorProtótipos
Crie o arquivo GerenciadorPrototipos.cs na pasta Services

```csharp
using ClinicaPOO.Models;

namespace ClinicaPOO.Services
{
    public class GerenciadorPrototipos
    {
        private readonly Dictionary<string, PacoteExame> _prototipos = new();

        public void CadastrarPrototipo(string chave, PacoteExame prototipo)
        {
            _prototipos[chave] = prototipo;
        }

        public PacoteExame ObterClone(string chave)
        {
            if (_prototipos.TryGetValue(chave, out var prototipo))
            {
                // Retorna uma Cópia Profunda por padrão para segurança das alterações
                return prototipo.Clonar();
            }

            throw new KeyNotFoundException($"O protótipo '{chave}' não foi encontrado no registro.");
        }
    }
}

```

---

### 📂 Etapa 4: Testando e Comparando Shallow Copy vs Deep Copy (Program.cs)

No Program.cs, simularemos o fluxo de atendimento da clínica e testaremos o comportamento das cópias em memória.

#### Subtarefa 4.1: Código do Program.cs`

```csharp
using ClinicaPOO.Models;
using ClinicaPOO.Services;

Console.WriteLine("=== SISTEMA DA CLÍNICA VIDA & SAÚDE - PATTERN PROTOTYPE ===\n");

var gerenciador = new GerenciadorPrototipos();

// 1. CONFIGURAÇÃO DOS PROTÓTIPOS MODELO (Feito na inicialização do sistema)
var checkupCardio = new PacoteExame("Check-up Cardiológico", "Jejum de 8h e evitar cafeína");
checkupCardio.AdicionarProcedimento(new Procedimento("Eletrocardiograma", 150.00m));
checkupCardio.AdicionarProcedimento(new Procedimento("Ecocardiograma", 350.00m));
checkupCardio.AdicionarProcedimento(new Procedimento("Teste Ergométrico", 200.00m));

gerenciador.CadastrarPrototipo("CARDIO_PADRAO", checkupCardio);

Console.WriteLine("--- Exibindo o Modelo Base ---");
checkupCardio.ExibirResumo();

// 2. ATENDIMENTO PACIENTE 1 (Uso de Deep Copy - Recomendado)
Console.WriteLine("\n------------------------------------------------");
Console.WriteLine("--> Atendimento 1: Clonando modelo via Deep Copy");
var atendimento1 = gerenciador.ObterClone("CARDIO_PADRAO");
atendimento1.NomePaciente = "Carlos Eduardo";

// Personalizando: Adicionando um procedimento extra só para este paciente
atendimento1.AdicionarProcedimento(new Procedimento("Holter 24h", 250.00m));

atendimento1.ExibirResumo();

// 3. DEMONSTRAÇÃO DO PERIGO DA CÓPIA SUPERFICIAL (Shallow Copy)
Console.WriteLine("\n------------------------------------------------");
Console.WriteLine("--> Atendimento 2: Exemplo de Clonagem Rasa (Shallow Copy)");

var modeloBase = gerenciador.ObterClone("CARDIO_PADRAO"); // Base original
var atendimentoErro = modeloBase.ClonarRaso();           // Clone Raso
atendimentoErro.NomePaciente = "Ana Maria";

// Alterando a lista no clone raso
atendimentoErro.Procedimentos.Add(new Procedimento("Glicemia em Jejum", 30.00m));

Console.WriteLine("\n[Alerta]: Procedimento adicionado na cópia rasa do cliente:");
atendimentoErro.ExibirResumo();

Console.WriteLine("\n[VERIFICAÇÃO]: Modelo Base após alteração no clone raso:");
modeloBase.ExibirResumo(); // A lista do modelo base FOI ALTERADA indevidamente!

// 4. VERIFICAÇÃO DO PACIENTE 1 (Deep Copy manteve-se isolado)
Console.WriteLine("\n------------------------------------------------");
Console.WriteLine("[VERIFICAÇÃO]: Atendimento 1 (Deep Copy) permaneceu isolado:");
atendimento1.ExibirResumo();

```

---




## 🚀 Desafios Extras para Aprendizado

1. **Protótipo de Prontuário Médico:** Crie uma nova classe ProntuarioTemplate que contenha perguntas e medições padrão de uma consulta inicial. Implemente a clonagem dessa estrutura para uso em triagens.
2. **Carga Dinâmica de Protótipos:** Adicione um método no GerenciadorPrototipos que permita listar os nomes de todos os modelos disponíveis cadastrados.e.
3. **Clonagem por Construtor de Cópia***: Tente reimplementar a classe PacoteExame utilizando o padrão de Copy Constructor (um construtor que aceita o próprio objeto como parâmetro public PacoteExame(PacoteExame outro)) em vez de utilizar o método MemberwiseClone(). Compare as abordagens.