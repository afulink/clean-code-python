# clean-code-python

[![Build Status](https://travis-ci.com/zedr/clean-code-python.svg?branch=master)](https://travis-ci.com/zedr/clean-code-python)
[![](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/download/releases/3.8.3/)

## Índice
  1. [Introdução](#introdução)
  2. [Variáveis](#variáveis)
  3. [Funções](#funções)
  4. [Objects and Data Structures](#objetos-e-estruturas-de-dados)
  5. [Classes](#classes)
     1. [S: Princípio da Responsabilidade Única (SRP)](#princípio-da-responsabilidade-Única-srp)
     2. [O: Princípio do Aberto/Fechado (OCP)](#princípio-do-abertofechado-ocp)
     3. [L: Princípio de Substituição de Liskov (LSP)](#princípio-de-substituição-de-liskov-lsp)
     4. [I: Princípio da Segregação de Interface (ISP)](#princípio-da-segregação-de-interface-isp)
     5. [D: Princípio da Inversão de Dependência (DIP)](#princípio-da-inversão-de-dependência-dip)
  6. [Não se repita (DRY)](#não-se-repita-dry)

## Introdução

Principios da engenharia de software, do livro de Robert C. Martin
[*Código Limpo*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adaptados para Python. Isto não é um style guide. É um guia para desenvolver software legível, reutilizavel e refatorável em Python.

Nem todos principios contidos aqui tem de ser seguidos estritamente, e muito menos irão ser universalmente aprovados. Estes são apenas guias e nada mais, mas que foram codificados durante muito anos por experiências coletivas dos autores de *Código Limpo*.

Inspriado em [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)

Versão Python3.7+

## **Variáveis**
### Use nomes significantes e pronunciáveis em suas variáveis

**Ruim:**
```python
import datetime


ymdstr = datetime.date.today().strftime("%y-%m-%d")
```

**Bom**:
```python
import datetime


current_date: str = datetime.date.today().strftime("%y-%m-%d")
```
**[⬆ back to top](#índice)**

### Use o mesmo vocabulário para o mesmo tipo de variável

**Ruim:**
Usamos três nomes diferentes para a mesma entidade:
```python
def get_user_info(): pass
def get_client_data(): pass
def get_customer_record(): pass
```

**Bom**:
Se a entidade for a mesma, você deve ser consistente ao se referir a ela em suas funções:
```python
def get_user_info(): pass
def get_user_data(): pass
def get_user_record(): pass
```

**Melhor ainda**:
Python é (também) uma linguagem de programação orientada a objetos. Se fizer sentido, empacote as funções junto com a implementação concreta da entidade em seu código, como atributos de instância, métodos ou métodos de propriedade:

```python
from typing import Union, Dict, Text


class Record:
    pass


class User:
    info : str

    @property
    def data(self) -> Dict[Text, Text]:
        return {}

    def get_record(self) -> Union[Record, None]:
        return Record()
```

**[⬆ back to top](#índice)**

### Use nomes fáceis de pesquisar
Nós vamos ler mais código do que escrever, por isso é importante que o código que escrevemos seja legível e fácil de achar. Ao *não* nomear variáveis, prejudicamos nossos leitores.
Torne seus nomes fáceis de procurar.

**Ruim:**
```python
import time


# Para que é o número 86400?
time.sleep(86400)
```

**Bom**:
```python
import time


# Declare-os no namespace global do módulo.
SECONDS_IN_A_DAY = 60 * 60 * 24
time.sleep(SECONDS_IN_A_DAY)
```
**[⬆ back to top](#índice)**

### Use variáveis explicativas
**Ruim:**
```python
import re


address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$"

matches = re.match(city_zip_code_regex, address)
if matches:
    print(f"{matches[1]}: {matches[2]}")
```

**Nada mal**: É melhor, mas ainda dependemos muito do regex.

```python
import re


address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$"
matches = re.match(city_zip_code_regex, address)

if matches:
    city, zip_code = matches.groups()
    print(f"{city}: {zip_code}")
```

**Bom**: Diminua a dependência de regex nomeando as variáveis em subgrupo
```python
import re


address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(?P<city>.+?)\s*(?P<zip_code>\d{5})?$"

matches = re.match(city_zip_code_regex, address)
if matches:
    print(f"{matches['city']}, {matches['zip_code']}")
```
**[⬆ back to top](#índice)**

### Evite mapear mentalmente
Não force o leitor do seu código a traduzir o que a variável significa.
Explicito é melhor que implito.

**Ruim:**
```python
seq = ("Austin", "New York", "San Francisco")

for item in seq:
    #do_stuff()
    #do_some_other_stuff()

    # Espere, `item` de novo?
    print(item)
```

**Bom**:
```python
locations = ("Austin", "New York", "San Francisco")

for location in locations:
    #do_stuff()
    #do_some_other_stuff()
    # ...
    print(location)
```
**[⬆ back to top](#índice)**


### Não adicione contextos desnecessários

Se o nome da sua classe/objeto expressa algo, não repita isso no nome da variável.

**Ruim:**

```python
class Car:
    car_make: str
    car_model: str
    car_color: str
```

**Bom**:

```python
class Car:
    make: str
    model: str
    color: str
```

**[⬆ back to top](#índice)**

### Use argumentos padrões ao invés de encadear condicionais

**Muito ruim**

Porque escrever:

```python
import hashlib


def create_micro_brewery(name):
    name = "Hipster Brew Co." if name is None else name
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

... quando você pode especificar um argumento padrão em vez disso? Isso também deixa claro que
você está esperando uma string como argumento.

**Bom**:

```python
from typing import Text
import hashlib


def create_micro_brewery(name: Text = "Hipster Brew Co."):
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

**[⬆ back to top](#índice)**
## **Funções**
### Argumentos de funções (2 ou menos, idealmente)
Limitar a quantidade de parametros de uma função é incrivelmente importantante porque isso torna sua função fácil de testar. Ter mais de três de leva em uma explosão onde você tem que testar vários casos diferentes, com argumentos separados.

Um ou dois argumentos é o caso ideal, e três deve ser evitado se possível. Algo além disso deve ser deixado de lado. Usualmente, se você tem mais de dois argumentos, suas funções estão tentando fazer coisas demais. Nos casos que não estão, na maior parte do tempo um objeto irá ser o suficiente como argumento.

**Ruim:**
```python
def create_menu(title, body, button_text, cancellable):
    pass
```

**Java-esque**:
```python
class Menu:
    def __init__(self, config: dict):
        self.title = config["title"]
        self.body = config["body"]
        # ...

menu = Menu(
    {
        "title": "My Menu",
        "body": "Something about my menu",
        "button_text": "OK",
        "cancellable": False
    }
)
```

**Muito bom**
```python
from typing import Text


class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: Text
    body: Text
    button_text: Text
    cancellable: bool = False


def create_menu(config: MenuConfig) -> None:
    title = config.title
    body = config.body
    # ...


config = MenuConfig()
config.title = "My delicious menu"
config.body = "A description of the various items on the menu"
config.button_text = "Order now!"
# O atributo de instância substitui o atributo de classe padrão.
config.cancellable = True

create_menu(config)
```

**Chique**
```python
from typing import NamedTuple


class MenuConfig(NamedTuple):
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False


def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = config
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

**Ainda mais chique**
```python
from typing import Text
from dataclasses import astuple, dataclass


@dataclass
class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: Text
    body: Text
    button_text: Text
    cancellable: bool = False

def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = astuple(config)
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

**Ainda mais chique, versões Python3.8+**
```python
from typing import TypedDict, Text


class MenuConfig(TypedDict):
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: Text
    body: Text
    button_text: Text
    cancellable: bool


def create_menu(config: MenuConfig):
    title = config["title"]
    # ...


create_menu(
    # Você precisa informar todos os parâmetros
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!",
        cancellable=True
    )
)
```
**[⬆ back to top](#índice)**

### Funções devem fazer somente uma coisa
Esta é, de longe, a regra mais importante da engenharia de software. Quando as funções fazem mais de uma coisa, elas são mais difíceis de compor, testar e pensar sobre. Quando você consegue isolar a função para apenas uma ação, elas podem ser refatoradas sem muita dificuldade e seu código será fácilmente lido. Se você não tirar mais nada deste guia além disso, você estará à frente de muitos programadores.

**Ruim:**
```python
from typing import List


class Client:
    active: bool


def email(client: Client) -> None:
    pass


def email_clients(clients: List[Client]) -> None:
    """Filter active clients and send them an email.
    """
    for client in clients:
        if client.active:
            email(client)
```

**Bom**:
```python
from typing import List


class Client:
    active: bool


def email(client: Client) -> None:
    pass


def get_active_clients(clients: List[Client]) -> List[Client]:
    """Filter active clients.
    """
    return [client for client in clients if client.active]


def email_clients(clients: List[Client]) -> None:
    """Send an email to a given list of clients.
    """
    for client in get_active_clients(clients):
        email(client)
```

Você vê uma oportunidade para usar geradores agora?

**Melhor ainda**
```python
from typing import Generator, Iterator


class Client:
    active: bool


def email(client: Client):
    pass


def active_clients(clients: Iterator[Client]) -> Generator[Client, None, None]:
    """Only active clients"""
    return (client for client in clients if client.active)


def email_client(clients: Iterator[Client]) -> None:
    """Send an email to a given list of clients.
    """
    for client in active_clients(clients):
        email(client)
```


**[⬆ back to top](#índice)**

### Nomes das funções devem dizer o que elas fazem

**Ruim:**

```python
class Email:
    def handle(self) -> None:
        pass

message = Email()
# O que isso quer dizer?
message.handle()
```

**Bom:**

```python
class Email:
    def send(self) -> None:
        """Send this message"""

message = Email()
message.send()
```

**[⬆ back to top](#índice)**

### Funções devem estar em apenas um nível de abstração

Quando você tem mais de um nível de abstração possívelmente sua função está fazendo coisa demais. Dividir suas funções desencadeia em código reusável e fácil de testar.

**Ruim:**

```python
# type: ignore

def parse_better_js_alternative(code: str) -> None:
    regexes = [
        # ...
    ]

    statements = code.split('\n')
    tokens = []
    for regex in regexes:
        for statement in statements:
            pass

    ast = []
    for token in tokens:
        pass

    for node in ast:
        pass
```

**Bom:**

```python
from typing import Tuple, List, Text, Dict


REGEXES: Tuple = (
   # ...
)


def parse_better_js_alternative(code: Text) -> None:
    tokens: List = tokenize(code)
    syntax_tree: List = parse(tokens)

    for node in syntax_tree:
        pass


def tokenize(code: Text) -> List:
    statements = code.split()
    tokens: List[Dict] = []
    for regex in REGEXES:
        for statement in statements:
            pass

    return tokens


def parse(tokens: List) -> List:
    syntax_tree: List[Dict] = []
    for token in tokens:
        pass

    return syntax_tree
```

**[⬆ back to top](#índice)**

### Não use sinalizadores como parâmetros de função

Os sinalizadores informam ao usuário que esta função faz mais de uma coisa. Funções
deve fazer uma coisa. Divida suas funções se elas estiverem seguindo um código diferente
caminhos baseados em verdadeiro ou falso.

**Ruim:**

```python
from typing import Text
from tempfile import gettempdir
from pathlib import Path


def create_file(name: Text, temp: bool) -> None:
    if temp:
        (Path(gettempdir()) / name).touch()
    else:
        Path(name).touch()
```

**Bom:**

```python
from typing import Text
from tempfile import gettempdir
from pathlib import Path


def create_file(name: Text) -> None:
    Path(name).touch()


def create_temp_file(name: Text) -> None:
    (Path(gettempdir()) / name).touch()
```

**[⬆ back to top](#índice)**

### Evite efeitos colaterais

Uma função produz um efeito colateral se fizer qualquer coisa além de assumir um valor ao invés de retornar outro valor ou valores. Por exemplo, um efeito colateral pode ser a escrita
a um arquivo, modificando alguma variável global ou transferindo acidentalmente todo o seu dinheiro
para um estranho.

No entanto, você precisa ter efeitos colaterais em um programa de vez em quando - por exemplo, como
no exemplo anterior, você pode precisar gravar em um arquivo. Nestes casos, você
deve centralizar e indicar onde você está incorporando efeitos colaterais. Não tem
várias funções e classes que gravam em um arquivo específico - em vez disso, têm um
(e apenas um) serviço que o faz.

O ponto principal é evitar armadilhas comuns, como o compartilhamento de estado entre objetos
sem qualquer estrutura, usando tipos de dados mutáveis ​​que podem ser gravados por qualquer coisa ou usando uma instância de uma classe, e não centralizando onde ocorrem seus efeitos colaterais.
Se você puder fazer isso, ficará mais feliz do que a grande maioria dos outros programadores.

**Ruim:**

```python
# type: ignore

# Este é um nome de nível de módulo..
# É uma boa prática defini-los como valores imutáveis, como uma string.
# No entanto...
fullname = "Ryan McDermott"

def split_into_first_and_last_name() -> None:
    # O uso da palavra-chave global aqui está mudando o significado da
    # seguinte linha. Esta função agora está alterando o nível do módulo
    # estado e introduzindo um efeito colateral!
    global fullname
    fullname = fullname.split()

split_into_first_and_last_name()

# MyPy irá detectar o problema,  'Incompatible types in
# assignment: (expression has type "List[str]", variable has type "str")'
print(fullname)  # ["Ryan", "McDermott"]

# OK. Funcionou da primeira vez, mas o que acontecerá se chamarmos de
# funcionar de novo?
```

**Bom:**
```python
from typing import List, AnyStr


def split_into_first_and_last_name(name: AnyStr) -> List[AnyStr]:
    return name.split()

fullname = "Ryan McDermott"
name, surname = split_into_first_and_last_name(fullname)

print(name, surname)  # => Ryan McDermott
```

**Muito bom**
```python
from typing import Text
from dataclasses import dataclass


@dataclass
class Person:
    name: Text

    @property
    def name_as_first_and_last(self) -> list:
        return self.name.split()


# A razão pela qual criamos instâncias de classes é para gerenciar o estado!
person = Person("Ryan McDermott")
print(person.name)  # => "Ryan McDermott"
print(person.name_as_first_and_last)  # => ["Ryan", "McDermott"]
```

**[⬆ back to top](#índice)**

## **Objetos e Estruturas de Dados**

*Coming soon*

**[⬆ back to top](#índice)**

## **Classes**

### **Princípio da Responsabilidade Única (SRP)**
### **Princípio do Aberto/Fechado (OCP)**
### **Princípio de Substituição de Liskov (LSP)**
### **Princípio da Segregação de Interface (ISP)**
### **Princípio da Inversão de Dependência (DIP)**

*Coming soon*

**[⬆ back to top](#índice)**

## **Não se repita (DRY)**

Mais informações sobre o príncipio [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

Como programador, você deve evitar código duplicado. A duplicação é ruim porque isso significa
que há mais de um lugar para alterar algo, se precisar mudar alguma lógica

Imagine que você é dono de um restaurante e você toma conta do seu estoque: todos os seus tomates, cebolas, alhos, temperos, etc. Se você tem multiplas listas onde guarda estas informações, então você terá que atualizar todas elas quando servir um prato que tenha tomates. Se você tivesse apenas uma lista, teria apenas um lugar para atualizar!

Frequentemente, você possui código duplicado porque você tem duas ou mais coisas levemente diferentes, que possuem muito em comum, mas suas diferenças lhe forçam a ter mais duas ou três funções que fazem muito das mesmas coisas. Remover código duplicado significa criar uma abstração que seja capaz de lidar com este conjunto de coisas diferentes com apenas uma função/módulo/classe.

Conseguir a abstração correta é crítico, por isso que você deveria seguir os princípios SOLID descritos na seção Classes. Abstrações ruins podem ser piores do que código duplicado, então tome cuidado! Dito isto, se você puder fazer uma boa abstração, faça-a! Não repita a si mesmo, caso contrário você se pegará atualizando muitos lugares toda vez que precisar mudar qualquer coisinha.

**Ruim:**

```python
from typing import List, Text, Dict
from dataclasses import dataclass

@dataclass
class Developer:
    def __init__(self, experience: float, github_link: Text) -> None:
        self._experience = experience
        self._github_link = github_link
        
    @property
    def experience(self) -> float:
        return self._experience
    
    @property
    def github_link(self) -> Text:
        return self._github_link
    
@dataclass
class Manager:
    def __init__(self, experience: float, github_link: Text) -> None:
        self._experience = experience
        self._github_link = github_link
        
    @property
    def experience(self) -> float:
        return self._experience
    
    @property
    def github_link(self) -> Text:
        return self._github_link
    

def get_developer_list(developers: List[Developer]) -> List[Dict]:
    developers_list = []
    for developer in developers:
        developers_list.append({
        'experience' : developer.experience,
        'github_link' : developer.github_link
            })
    return developers_list

def get_manager_list(managers: List[Manager]) -> List[Dict]:
    managers_list = []
    for manager in managers:
        managers_list.append({
        'experience' : manager.experience,
        'github_link' : manager.github_link
            })
    return managers_list

## create list objects of developers
company_developers = [
    Developer(experience=2.5, github_link='https://github.com/1'),
    Developer(experience=1.5, github_link='https://github.com/2')
]
company_developers_list = get_developer_list(developers=company_developers)

## create list objects of managers
company_managers = [
    Manager(experience=4.5, github_link='https://github.com/3'),
    Manager(experience=5.7, github_link='https://github.com/4')
]
company_managers_list = get_manager_list(managers=company_managers)
```

**Bom:**

```python
from typing import List, Text, Dict
from dataclasses import dataclass

@dataclass
class Employee:
    def __init__(self, experience: float, github_link: Text) -> None:
        self._experience = experience
        self._github_link = github_link
        
    @property
    def experience(self) -> float:
        return self._experience
    
    @property
    def github_link(self) -> Text:
        return self._github_link
    


def get_employee_list(employees: List[Employee]) -> List[Dict]:
    employees_list = []
    for employee in employees:
        employees_list.append({
        'experience' : employee.experience,
        'github_link' : employee.github_link
            })
    return employees_list

## create list objects of developers
company_developers = [
    Employee(experience=2.5, github_link='https://github.com/1'),
    Employee(experience=1.5, github_link='https://github.com/2')
]
company_developers_list = get_employee_list(employees=company_developers)

## create list objects of managers
company_managers = [
    Employee(experience=4.5, github_link='https://github.com/3'),
    Employee(experience=5.7, github_link='https://github.com/4')
]
company_managers_list = get_employee_list(employees=company_managers)
```



**[⬆ back to top](#índice)**

