## **Primitivos**

### INDENTATION
O certo é usar ESPAÇO e não TABULAÇÃO!t kkk

```Python

def sum(
        a, b): 
    print(a + b)

c = sum(
  a, b)

c = reduce(a, b, c
           d, e)

c = (a
	 + b
	 + c
	 + d)

```

### OPERATORS
```Python
# Não tem operador ternário
# O equivalente:
"True" if 10 > 9 else "False"
```
### BOOLEANS
	and or True False bool()
- True e False são 1 e 0 e **somente** isso.
- Valores vazios são falsos.
- **and** e **or** fazem cast para boolean em comparações entre *ints*.
- É POSSÍVEL COMPARAR VÁRIAS COISAS DE UMA VEZ: 
	``a < b < c``

### EQUALITY
	= !== == is
```Python
a = [1, 2, 3] # a aponta para 10
b = a  # b aponta para a
b is a # b e a apontam para o mesmo objeto
b == a # b e a tem o mesmo valor
b = [1, 2, 3]
b is a # b e a NÃO apontam para o mesmo objeto
```

### STRINGS
```Python
f"Similar aos template literals do JS {seuLindo}"

len(string) # aqui é uma função

user_input = input("Diz alguma coisa: ")

```

# **Variáveis e Coleções**
### VARIABLES
Não existe declaração de variáveis:
```Python
a

print("howdy")
```

### LISTS
```Python
li[start:end:step]

li = [1, 2, 3, 4, 5]
li[1:2]  # => [2]
li[1::2] # => [2, 4]

new_list = li[:] # copia a lista

del li[3] # => [1, 2, 3, 5] - remove um elemento específico

li.append[6] # => [1, 2, 3, 4, 5, 6]
li.pop()   # => [1, 2, 3, 4, 5]
li.remove[3] # => [1, 2, 4, 5] - remove a primeira instância de um valor

li.insert[3, 2] # => [1, 2, 3, 4, 5] - insere um valor em um índice

li2 = [6, 7]
li3 = li + li2 # => [1,2,3,4,5,6,7]

li.extend(li2) # agora li = [1,2,3,4,5,6,7]

4 in li # => True
```

#### Tuples
    Listas imutáveis

```Python
my_tup = (1, 2, 3) # parênteses

# Em tuplas de 1 elemento, usar a vírugla é obrigatório
type((1))  # => <class 'int'>
type((1,)) # => <class 'tuple'>
type(())   # => <class 'tuple'> ¯\_(ツ)_/¯

a, b, c = (1, 2, 3) # a is 1, b is 2, c is 3 (funciona com listas)

a, b = b, a # inverte os valores
```

### DICTIONARIES
	AKA Objetos
```Python
# As chaves em Python devem ser de um tipo imutável

my_dict = {(1, 2, 3): "hey"}

my_dict = {"one": 1, "two": 2, "three": 3}

# O equivalente de Object.keys() é
list(my_dict.keys())
list(my_dict.values())

"one" in my_dict

my_dict.get("four") # => None
"four" in my_dict # => KeyError

del my_dict["three"]

```
#### Sets
	Não permite duplicatas
```Python
new_set = {1, 1, 2, 3, 4, 4} # => {1, 2, 3, 4}

new_set.add(5) # => {1, 2, 3, 4, 5}

other_set = {3, 4}

# Operações de União e Intersessão
new_set & other_set # => {3,4}
new_set | other_set # => {1,2,3,4,5}
new_set - other_set # => {1,2,5}
new_set => other_set # verifica se um set é superSet do outro (herança), nesse caso, True
```
# **Controle de Fluxo e Iteráveis**
## ITERATIONS
```Python
# O FOR loop itera através de coleções:
list = ["cachorro", "elefante", "baleia"]
for el in list:
	    print("{} is a mammal".format(animal))

# range(start, finish, step)
for i in range(4)
	print(i)

# enumerate(index, value)
for i, value in enumerate(list)
    print(i, value)

# while
while x < 4:
    print(x)
    x += 1

```

## TRY
```Python
try:
	raise IndexError("Algo não deu certo")
except IndexError as e:
	pass
except (TypeError, NameError):
	pass
else:
	print("All good!"
finally:
	print("cleanup")
```

## CREATING FILES
```Python
contents = "teste"

with open("misc/file.txt", "w") as file: # w é uma flag para escrita

  file.write(str(contents))

with open("misc/file.txt", "r") as file: # r é pra escrita

  content_file = file.read()

  

print(content_file)
```

## FUNCTIONS
```Python
def sum(x, y):
	print(f"x: {x}, y: {y})
	return x + y

# A chamada de função pode ser:

sum(5,5)
sum(y=10,x=8)

def sum_all(*args):
    sum = 0
    for i in args:
        sum += i
    return sum
```

### Scopes
```Python
x = 1

def set_x(num):
	global x
	x = num

set_x(10) # x is now 10

def average():
	total = 0
	count = 0
	def incr(num):
		total += num
		count += 1
		return total / count
	return incr
```
### Lambda
```Python
(lambda x: x > 2)(3) # => True
(lambda x, y: x**2 + y**2)(2, 1) # => 5
```

### Map
```Python
og_list = [1,2,3,4,5]

list = list(map((lambda x: x**2), og_list))
list = list(filter((lambda x: x>3), og_list))
```
# **Módulos**

## IMPORT
```Python
import math # importa o namespace
print(math.sqrt(16))

from math import ceil
print(math.ceil(3.14))

from math import * # importa as funções para o namespace atual
print(sqrt(16))
```
# Classes
```Python
Class Human:

	species = "H. Sapiens"
	upper_class = "mammal"

	def __init__(self, name):
		self.name = name
		self._age = 0

	# todos os métodos da instância recebem SELF como primeiro argumento (o this)
	def say(self, msg):
		print(f"{self.name}: {message}")

	def sing(self):
		print("testando... 1... 2... 3")

	@classmethod
    def get_species(cls):

          return cls.species

    @staticmethod

    def laugh():

          print("hahaha")

    @property

    def age(self):

          return self._age

    @age.setter

    def age(self, age):

          self._age = age

a = Human(name="Adolfo")

  

a.say("Howdy")

a.age = 27

print(a.age)
```
### Inheritance
```Python
class Kid(Human)

	def __init__(self, name, sex):
		self.sex  = sex
		self._age = 0
		super().__init__(name)

	def say(self, msg):
		print(f"{self.name}: dadada")
	def poop(self)
		print(f"bebe fez coco")
	
```