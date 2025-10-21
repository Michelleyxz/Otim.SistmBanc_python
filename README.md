Sistema Bancário Orientado a Objetos (POO) em Python

Este projeto é a implementação de um sistema bancário simples, focado em demonstrar conceitos avançados de Programação Orientada a Objetos (POO) em Python, como Abstração, Herança, Encapsulamento, Polimorfismo, e padrões de projeto como Decorators para logging e Iterators para navegação.

Funcionalidades
O sistema oferece as operações básicas de uma conta corrente para clientes Pessoa Física, gerenciadas através de um menu de console interativo.

Usuário (Cliente): Cadastro de novos usuários (Pessoa Física).

Conta Corrente: Criação de contas vinculadas a um usuário.

Operações: Depositar, Sacar e Visualizar Extrato.

Relatório: Geração de um relatório detalhado de transações, com opção de filtro por tipo (Saque ou Depósito).

Listagem: Listagem de todos os usuários e contas cadastradas.

Arquitetura e Conceitos POOO código é estruturado para seguir rigorosamente os princípios de POO:Conceito POOClasses/Padrões AplicadosDescriçãoAbstraçãoCliente, Conta, TransacaoDefine a estrutura básica e os métodos que as classes concretas devem implementar (ex: Transacao.registrar()).HerançaPessoaFisica, ContaCorrente, Saque, DepositoPermite reutilização de código e cria hierarquias claras (ex: PessoaFisica herda de Cliente).EncapsulamentoUso de atributos privados (_saldo, _historico, etc.) e Properties (@property) para controlar o acesso e a modificação de dados internos.PolimorfismoO método realizar_transacao() do Cliente aceita qualquer objeto que herde de Transacao (Saque ou Deposito), chamando o método registrar() específico para cada tipo.Padrão Decorator@log_transacaoUma função decoradora é usada para interceptar a execução de métodos chave (como depositar, sacar e as funções de menu) e registrar um log de auditoria em transacoes.log.Padrão IteratorContaIteradorImplementa o protocolo Iterator (__iter__ e __next__) na classe Conta para permitir iteração eficiente sobre a lista de contas de um cliente.GeneratorsHistorico.gerar_relatorio()Utiliza yield para gerar relatórios de transações de forma eficiente, especialmente útil para grandes volumes de dados.

Como Executar: 
Pré-requisitos:
Python 3.8+

Clone o repositório para sua máquina local:

Bash

git clone [LINK_DO_SEU_REPOSITÓRIO]
cd sistema-bancario-poo
Execute o arquivo principal:

Bash

python seu_arquivo.py










um log de auditoria em transacoes.log.Padrão IteratorContaIteradorImplementa o protocolo Iterator (__iter__ e __next__) na classe Conta para permitir iteração eficiente sobre a lista de contas de um cliente.GeneratorsHistorico.gerar_relatorio()Utiliza yield para gerar relatórios de transações de forma eficiente, especialmente útil para grandes volumes de dados.⚙️ Como ExecutarPré-requisitosPython 3.8+
