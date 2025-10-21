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

git clone [gh repo clone Michelleyxz/Otim.SistmBanc_python]
cd sistema-bancario-poo
Execute o arquivo principal:

Bash

python seu_arquivo.py


import textwrap
from abc import ABC, abstractmethod
from datetime import datetime
from typing import List, Optional, Generator, Any
from functools import wraps


# -------------------------
# Decorador de log
# Registra chamadas de funções em um arquivo 'transacoes.log'
# -------------------------
def log_transacao(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        resultado = None
        try:
            resultado = func(*args, **kwargs)
            status = "SUCESSO"
            return resultado
        except Exception as e:
            status = f"FALHA: {e}"
            raise
        finally:
            # montar uma linha de log com data/hora, função, args mínimos
            timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            nome_func = func.__name__
            # tenta extrair CPF se estiver nos kwargs ou no primeiro arg (lista clientes)
            extra = ""
            try:
                if "cpf" in kwargs:
                    extra = f" cpf={kwargs['cpf']}"
                elif len(args) > 0 and isinstance(args[0], Cliente):
                    extra = f" cpf={args[0].cpf}"
                elif len(args) > 1 and isinstance(args[1], str):
                    # Para funções como sacar(valor) em Conta
                    extra = f" valor={args[1]}"
            except Exception:
                extra = ""
            with open("transacoes.log", "a", encoding="utf-8") as f:
                f.write(f"{timestamp} | {nome_func} | {status}{extra}\n")
    return wrapper


# -------------------------
# Iterador personalizado para contas
# Permite iterar sobre uma lista de contas com protocolo iterator
# -------------------------
class ContaIterador:
    def __init__(self, contas: List["Conta"]):
        self._contas = contas
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self) -> "Conta":
        if self._index < len(self._contas):
            conta = self._contas[self._index]
            self._index += 1
            return conta
        raise StopIteration


# -------------------------
# Cliente e PessoaFisica
# -------------------------
class Cliente:
    def __init__(self, nome: str, endereco: str, cpf: str, data_nascimento: str):
        self.nome = nome
        self.endereco = endereco
        self.cpf = cpf
        self.data_nascimento = data_nascimento
        self.contas: List["Conta"] = []

    def adicionar_conta(self, conta: "Conta"):
        self.contas.append(conta)

    def recuperar_conta_principal(self) -> Optional["Conta"]:
        return self.contas[0] if self.contas else None

    @log_transacao
    def realizar_transacao(self, conta: "Conta", transacao: "Transacao"):
        # limita transações do dia a 10 por conta (exemplo)
        qtd_hoje = len(conta.historico.transacoes_do_dia())
        if qtd_hoje >= 10:
            print("\n@@@ Você excedeu o número de transações permitidas para hoje! @@@")
            return False

        sucesso = transacao.registrar(conta)
        return sucesso


class PessoaFisica(Cliente):
    def __init__(self, nome: str, data_nascimento: str, cpf: str, endereco: str):
        super().__init__(nome=nome, endereco=endereco, cpf=cpf, data_nascimento=data_nascimento)

    def __repr__(self) -> str:
        return f"<{self.__class__.__name__}: ('{self.nome}', '{self.cpf}')>"


# -------------------------
# Abstração Transacao
# -------------------------
class Transacao(ABC):
    def __init__(self, valor: float):
        self._valor = float(valor)
        self._data = datetime.now()

    @property
    def valor(self) -> float:
        return self._valor

    @property
    def data(self) -> datetime:
        return self._data

    @abstractmethod
    def registrar(self, conta: "Conta") -> bool:
        ...

    def __str__(self):
        return f"{self.__class__.__name__}: R$ {self.valor:.2f}"


# -------------------------
# Saque e Deposito
# -------------------------
class Saque(Transacao):
    def registrar(self, conta: "Conta") -> bool:
        sucesso = conta.sacar(self.valor)
        if sucesso:
            conta.historico.adicionar_transacao(self)
        return sucesso


class Deposito(Transacao):
    def registrar(self, conta: "Conta") -> bool:
        sucesso = conta.depositar(self.valor)
        if sucesso:
            conta.historico.adicionar_transacao(self)
        return sucesso


# -------------------------
# Historico com gerador de relatório (Continuação da sua classe)
# -------------------------
class Historico:
    def __init__(self):
        self._transacoes: List[dict] = []

    @property
    def transacoes(self) -> List[dict]:
        return self._transacoes

    def adicionar_transacao(self, transacao: Transacao):
        self._transacoes.append({
            "tipo": transacao.__class__.__name__,
            "valor": transacao.valor,
            "data": transacao.data.strftime("%Y-%m-%d %H:%M:%S"),
        })

    def transacoes_do_dia(self) -> List[dict]:
        data_atual = datetime.now().date()
        return [
            transacao for transacao in self._transacoes
            if datetime.strptime(transacao["data"], "%Y-%m-%d %H:%M:%S").date() == data_atual
        ]

    # Implementação do gerador para relatórios
    def gerar_relatorio(self, tipo_transacao: Optional[str] = None) -> Generator[str, Any, None]:
        for transacao in self._transacoes:
            if tipo_transacao is None or transacao["tipo"].lower() == tipo_transacao.lower():
                yield f"{transacao['data']} | {transacao['tipo']}: R$ {transacao['valor']:.2f}"

# -------------------------
# Contas (Abstração e Herança)
# -------------------------
class Conta(ABC):
    def __init__(self, numero: int, cliente: Cliente):
        self._saldo = 0.0
        self._numero = numero
        self._agencia = "0001"
        self._cliente = cliente
        self._historico = Historico()

    @property
    def saldo(self) -> float:
        return self._saldo

    @property
    def numero(self) -> int:
        return self._numero

    @property
    def agencia(self) -> str:
        return self._agencia

    @property
    def cliente(self) -> Cliente:
        return self._cliente

    @property
    def historico(self) -> Historico:
        return self._historico

    @classmethod
    def nova_conta(cls, cliente: Cliente, numero: int):
        return cls(numero, cliente)

    @abstractmethod
    def sacar(self, valor: float) -> bool:
        ...

    @abstractmethod
    def depositar(self, valor: float) -> bool:
        ...

    def __str__(self):
        return f"""\
            Agência:\t{self.agencia}
            C/C:\t\t{self.numero}
            Titular:\t{self.cliente.nome}
        """

    def __iter__(self):
        return ContaIterador(self.cliente.contas)


class ContaCorrente(Conta):
    def __init__(self, numero: int, cliente: Cliente, limite: float = 500.0, limite_saques: int = 3):
        super().__init__(numero, cliente)
        self._limite = limite
        self._limite_saques = limite_saques

    @log_transacao
    def sacar(self, valor: float) -> bool:
        numero_saques = len([
            transacao for transacao in self.historico.transacoes
            if transacao["tipo"] == Saque.__name__ and datetime.strptime(transacao["data"], "%Y-%m-%d %H:%M:%S").date() == datetime.now().date()
        ])

        excedeu_saldo = valor > self.saldo
        excedeu_limite = valor > self._limite
        excedeu_saques = numero_saques >= self._limite_saques

        if excedeu_saldo:
            print("\n@@@ Operação falhou! Você não tem saldo suficiente. @@@")
        elif excedeu_limite:
            print("\n@@@ Operação falhou! O valor do saque excede o limite por transação. @@@")
        elif excedeu_saques:
            print("\n@@@ Operação falhou! Número máximo de saques diários excedido. @@@")
        elif valor > 0:
            self._saldo -= valor
            print("\n=== Saque realizado com sucesso! ===")
            return True
        else:
            print("\n@@@ Operação falhou! O valor informado é inválido. @@@")
        return False

    @log_transacao
    def depositar(self, valor: float) -> bool:
        if valor > 0:
            self._saldo += valor
            print("\n=== Depósito realizado com sucesso! ===")
            return True
        else:
            print("\n@@@ Operação falhou! O valor informado é inválido. @@@")
            return False

    def __str__(self):
        return textwrap.dedent(f"""\
            Agência:\t{self.agencia}
            C/C:\t\t{self.numero}
            Titular:\t{self.cliente.nome}
            Limite:\t\tR$ {self._limite:.2f}
        """)


# -------------------------
# Funções Utilitárias e Menu
# -------------------------
def filtrar_cliente(clientes: List[Cliente], cpf: str) -> Optional[Cliente]:
    clientes_filtrados = [cliente for cliente in clientes if cliente.cpf == cpf]
    return clientes_filtrados[0] if clientes_filtrados else None


def buscar_conta_cliente(cliente: Cliente, numero_conta: int) -> Optional[Conta]:
    contas_filtradas = [conta for conta in cliente.contas if conta.numero == numero_conta]
    return contas_filtradas[0] if contas_filtradas else None


def menu():
    menu_str = """\n
    ============= MENU =============
    [d]\tDepositar
    [s]\tSacar
    [e]\tExtrato
    [r]\tRelatório (Transações)
    [nc]\tNova conta
    [lc]\tListar contas
    [nu]\tNovo usuário
    [lu]\tListar usuários
    [q]\tSair
    => """
    return input(textwrap.dedent(menu_str))

@log_transacao
def depositar(clientes: List[Cliente]):
    cpf = input("Informe o CPF do cliente: ")
    cliente = filtrar_cliente(clientes, cpf)

    if not cliente:
        print("\n@@@ Cliente não encontrado! @@@")
        return

    valor = float(input("Informe o valor do depósito: "))
    conta = cliente.recuperar_conta_principal()

    if not conta:
        print("\n@@@ Cliente não possui conta! @@@")
        return

    transacao = Deposito(valor)
    cliente.realizar_transacao(conta, transacao)

@log_transacao
def sacar(clientes: List[Cliente]):
    cpf = input("Informe o CPF do cliente: ")
    cliente = filtrar_cliente(clientes, cpf)

    if not cliente:
        print("\n@@@ Cliente não encontrado! @@@")
        return

    valor = float(input("Informe o valor do saque: "))
    conta = cliente.recuperar_conta_principal()

    if not conta:
        print("\n@@@ Cliente não possui conta! @@@")
        return

    transacao = Saque(valor)
    cliente.realizar_transacao(conta, transacao)

@log_transacao
def exibir_extrato(clientes: List[Cliente]):
    cpf = input("Informe o CPF do cliente: ")
    cliente = filtrar_cliente(clientes, cpf)

    if not cliente:
        print("\n@@@ Cliente não encontrado! @@@")
        return

    conta = cliente.recuperar_conta_principal()
    if not conta:
        print("\n@@@ Cliente não possui conta! @@@")
        return

    print("\n=============== EXTRATO ===============")
    
    # Usando o gerador para listar transações
    transacoes = conta.historico.gerar_relatorio()
    extrato = "Não foram realizadas movimentações."
    if any(transacoes): # Verifica se há transações
        print("Histórico de Transações:")
        for linha in conta.historico.gerar_relatorio():
            print(linha)
    else:
        print(extrato)

    print(f"\nSaldo:\t\tR$ {conta.saldo:.2f}")
    print("=======================================")

@log_transacao
def criar_cliente(clientes: List[Cliente]):
    cpf = input("Informe o CPF (somente números): ")
    cliente = filtrar_cliente(clientes, cpf)

    if cliente:
        print("\n@@@ Já existe cliente com esse CPF! @@@")
        return

    nome = input("Informe o nome completo: ")
    data_nascimento = input("Informe a data de nascimento (dd-mm-aaaa): ")
    endereco = input("Informe o endereço (logradouro, nro - bairro - cidade/sigla estado): ")

    cliente = PessoaFisica(nome=nome, data_nascimento=data_nascimento, cpf=cpf, endereco=endereco)
    clientes.append(cliente)

    print("\n=== Cliente criado com sucesso! ===")

@log_transacao
def criar_conta(numero_conta: int, clientes: List[Cliente], contas: List[Conta]):
    cpf = input("Informe o CPF do cliente: ")
    cliente = filtrar_cliente(clientes, cpf)

    if not cliente:
        print("\n@@@ Cliente não encontrado, fluxo de criação de conta encerrado! @@@")
        return

    conta = ContaCorrente.nova_conta(cliente=cliente, numero=numero_conta)
    contas.append(conta)
    cliente.adicionar_conta(conta)

    print("\n=== Conta criada com sucesso! ===")

@log_transacao
def listar_contas(contas: List[Conta]):
    if not contas:
        print("\n@@@ Não existem contas cadastradas. @@@")
        return
    
    print("\n=============== LISTA DE CONTAS ===============")
    # Usando o iterador personalizado
    for conta in ContaIterador(contas):
        print("=" * 40)
        print(textwrap.dedent(str(conta)))
    print("===============================================")

def listar_usuarios(clientes: List[Cliente]):
    if not clientes:
        print("\n@@@ Não existem usuários cadastrados. @@@")
        return
    
    print("\n=============== LISTA DE USUÁRIOS ===============")
    for cliente in clientes:
        print("=" * 40)
        print(f"Nome:\t\t{cliente.nome}")
        print(f"CPF:\t\t{cliente.cpf}")
        print(f"Data Nasc.:\t{cliente.data_nascimento}")
        print(f"Endereço:\t{cliente.endereco}")
    print("=================================================")

@log_transacao
def exibir_relatorio(clientes: List[Cliente]):
    cpf = input("Informe o CPF do cliente: ")
    cliente = filtrar_cliente(clientes, cpf)

    if not cliente:
        print("\n@@@ Cliente não encontrado! @@@")
        return

    conta = cliente.recuperar_conta_principal()
    if not conta:
        print("\n@@@ Cliente não possui conta! @@@")
        return

    print("\n=========== RELATÓRIO DE TRANSAÇÕES ===========")
    tipo = input("Filtrar por tipo (Saque/Deposito ou ENTER para todos): ")
    
    gerador_relatorio = conta.historico.gerar_relatorio(tipo)
    
    if any(conta.historico.gerar_relatorio(tipo)): # Verifica se há transações com o filtro
        for linha in conta.historico.gerar_relatorio(tipo):
            print(linha)
    else:
        print("Não há transações com o filtro especificado.")
    print("===============================================")

# -------------------------
# Função Principal
# -------------------------
def main():
    clientes: List[Cliente] = []
    contas: List[Conta] = []
    numero_conta = 1

    while True:
        opcao = menu()

        if opcao == "d":
            depositar(clientes)

        elif opcao == "s":
            sacar(clientes)

        elif opcao == "e":
            exibir_extrato(clientes)

        elif opcao == "r":
            exibir_relatorio(clientes)
            
        elif opcao == "nu":
            criar_cliente(clientes)
        
        elif opcao == "lu":
            listar_usuarios(clientes)

        elif opcao == "nc":
            criar_conta(numero_conta, clientes, contas)
            if any(contas):
                numero_conta += 1

        elif opcao == "lc":
            listar_contas(contas)

        elif opcao == "q":
            print("\nObrigado por usar nosso sistema. Até a próxima!")
            break

        else:
            print("\n@@@ Operação inválida, por favor selecione novamente a operação desejada. @@@")


if __name__ == "__main__":
    main()







um log de auditoria em transacoes.log.Padrão IteratorContaIteradorImplementa o protocolo Iterator (__iter__ e __next__) na classe Conta para permitir iteração eficiente sobre a lista de contas de um cliente.GeneratorsHistorico.gerar_relatorio()Utiliza yield para gerar relatórios de transações de forma eficiente, especialmente útil para grandes volumes de dados.⚙️ Como ExecutarPré-requisitosPython 3.8+
