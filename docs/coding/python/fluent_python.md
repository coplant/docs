---
  tags:
    - Python
    - Advanced
    - Book
---

Название: Python – к вершинам мастерства. Лаконичное и эффективное программирование  
Автор: Лусиану Рамальо  
Год: 2022  
[PDF](https://github.com/coplant/docs/blob/main/book/Ramalio_L_Fluent_Python.pdf)

---
## Глава 1. Модель данных в языке Python

При работе с фреймворком, разработчики большую часть времени тратят на реализацию методов, вызываемых фреймворком.
Это относится к определению новых классов, использующих модель данных.
Интерпретатор Python вызывает специальные "магические" или "dunder" методы для выполнения основных операций над объектами, часто вызываемые определенным синтаксисом.

### Специальные методы

Пример класса `FrenchDeck` демонстрирует, как реализация всего двух специальных методов, `__getitem__` и `__len__`,
позволяет колоде вести себя как стандартная последовательность --
получить количество карт, первую или последнюю (_или случайную `random.choice(deck)`_) карту. 
Так же благодаря `__getitem__` колода поддерживает срезы, итерирование, сортировку. 

!!! note ""  
    Итерирование часто подразумевается неявно. Если в коллекции отсутствует метод `__contains__`, то оператор `in` производит последовательный просмотр.

```py linenums="1"
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])


class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]
```

???+ note annotate  "`namedtuple` vs `dataclass`"
    Мы используем класс `namedtuple` для построения классов,
    содержащих только атрибуты и никаких методов, как, например, запись базы данных.
    
    `Card = collections.namedtuple('Card', ['rank', 'suit'])`
    
    Однако для этой задачи может подойти использование `dataclass`. 
    Как и кортежи, они могут быть неизменяемыми (`frozen=True`),
    но более эффективными по памяти.

    === "namedtuple"
    
        ```python linenums="1"
        from typing import NamedTuple
        
        class UserNamedTuple(NamedTuple):
            pk: int
            name: str
        ```
        { .annotate }
    
    
    === "dataclass"
    
        ```python linenums="1"
        from dataclasses import dataclass
        
        @dataclass(frozen=True, slots=True)
        class UserDataClass:
            pk: int
            name: str
        ```
        { .annotate }

Специальные методы в Python предназначены для вызова интерпретатором, и, как правило, вызываются неявно.
Они позволяют объектам вести себя как встроенные типы и хорошо интегрироваться с возможностями языка Python. 

### Эмуляция числовых, строковых и булевых типов

Реализуем класс для представления двумерных векторов с методами
`__repr__`, `__abs__`, `__add__` и `__mul__`.

```python linenums="1"
import math


class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f'Vector({self.x!r}, {self.y!r})'

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x, y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
```

В примере реализованы операторы `+` и `*`, демонстрируя методы `__add__` и `__mul__`.
Оба метода создают и возвращают новый экземпляр класса `Vector`, не изменяя исходные операнды.
Такое поведение ожидается от инфиксных операторов: они создают новые объекты, не затрагивая исходные данные.

!!! note "" 
    Важное требование к объекту Python -- обеспечить полезные строковые представления себя:
    одно – для отладки и протоколирования, другое – для показа пользователям. 
    Именно для этой цели предназначены специальные методы `__repr__` и `__str__`.

Метод `__repr__` используется функцией `repr` для получения строкового представления объекта.
Если его не реализовать, объект класса `Vector` будет представлен как `<Vector object at 0x10e100070>`.

В методе `__repr__` использовано `!r` в f-строке для получения стандартного представления атрибутов.
Это важно, так как позволяет отличать `Vector(1, 2)` от `Vector('1', '2')`.

В отличие от `__repr__`, метод `__str__` вызывается конструктором `str()` и неявно
используется в функции `print`. Он должен возвращать строку, пригодную для показа пользователям.

!!! tip 
    Программисты, привыкшие к методу `toString` в других языках,
    часто реализуют `__str__` вместо `__repr__`.
    Однако, если нужно выбрать один из них, предпочтение стоит отдать `__repr__`.

По умолчанию любой экземпляр пользовательского класса считается истинным, но положение меняется,
если реализован хотя бы один из методов `__bool__` или `__len__`. 
Функция `bool(x)`, по существу, вызывает `x.__bool__()` и использует полученный результат. 
Если метод `__bool__` не реализован, то Python пытается вызвать `x.__len__()` и при получении нуля функция `bool` возвращает `False`.
В противном случае `bool` возвращает `True`.

### API коллекций. Абстрактные базовые классы

??? note "API коллекций"
    Все классы на этой диаграмме являются _абстрактными базовыми классами_, ABC.

    ```mermaid
    classDiagram
        class Iterable {
            __iter__
        }
        
        class Sized {
            __len__
        }
        
        class Container {
            __contains__
        }
        
        class Reversible {
            __reversed__
        }
        
        class Collection {
        }
        
        class Sequence {
            __getitem__
            __contains__
            __iter__
            __reversed__
            index
            count
        }
        
        class Mapping {
            __getitem__
            __contains__
            __eq__
            __ne__
            get
            items
            keys
            values
        }
        
        class Set {
            isdisjoint
            __le__
            __lt__
            __gt__
            __ge__
            __eq__
            __ne__
            __and__
            __or__
            __sub__
            __xor__
        }
        
        Reversible --|> Iterable 
    
        Sequence --|> Reversible
        Sequence --|> Collection
        
        Mapping --|> Collection
        Set --|> Collection
         
        Collection --|> Iterable 
        Collection --|> Sized 
        Collection --|> Container 
    ```
У каждого из классов `Iterable`, `Sized`, `Container` есть всего один специальный метод.
Абстрактный базовый класс `Collection` унифицирует все три основных интерфейса, которые должна реализовать любая коллекция:

- `Iterable` -- для поддержки `for`, распаковки и других видов итерирования;
- `Sized` -- для поддержки `len`;
- `Container` -- для поддержки `in`.

Python не требует наследования какому-то из этих ABC.
Любой класс, реализующий метод `__len__`, удовлетворяет требованиям интерфейса `Sized`.

Три важнейшие специализации `Collection`:

- `Sequence` -- формализует интерфейс встроенных классов, в частности `list` и `str`;
- `Mapping` -- реализован классами `dict`, `collections.defaultdict` и др.;
- `Set` -- интерфейс встроенных типов `set` и `frozenset`.

!!! note ""
    Только `Sequence` реализует интерфейс `Reversible`.

!!! tip
    Начиная с `Python 3.7`, тип `dict` считается «упорядоченным», что означает сохранение порядка вставки ключей. 
    Однако переупорядочить ключи словаря `dict` по собственному желанию невозможно.

### Сводка специальных методов

Имена специальных методов (операторы не включены)  

| Категория                                              | Имена методов                                                              |
|--------------------------------------------------------|----------------------------------------------------------------------------|
| Представление в виде строк и байтов                    | `__repr__`, `__str__`, `__format__`, `__bytes__`, `__fspath__`             |
| Преобразование в число                                 | `__bool__`, `__complex__`, `__int__`, `__float__`, `__hash__`, `__index__` |
| Эмуляция коллекций                                     | `__len__`, `__getitem__`, `__setitem__`, `__delitem__`, `__contains__`     |
| Итерирование                                           | `__iter__`, `__aiter__`, `__next__`, `__anext__`, `__reversed__`           |
| Выполнение объектов, допускающих вызов, или сопрограмм | `__call__`, `__await__`                                                    |
| Управление контекстом                                  | `__enter__`, `__exit__`, `__aenter__`, `__aexit__`  `                      |
| Создание и уничтожение объектов                        | `__new__`, `__init__`, `_del__`                                            |
| Управление атрибутами                                  | `__getattr__`, `__getattribute__`, `__setattr__`, `__delattr__`, `__dir__` |
| Дескрипторы атрибутов                                  | `__get__`, `__set__`, `__delete__`, `__set_name__`                         |
| Абстрактные базовые классы                             | `__instancecheck__`, `__subclasscheck__`                                   |
| Метапрограммирование классов                           | `__prepare__`, `_init_subclass__`, `__class_getitem__`, `__mro_entries__`  |



Имена специальных методов для операторов 

| Категория                                        | Символы                                                        | Имена методов                                                                                                        |
|--------------------------------------------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| Унарные числовые операторы                       | `-` `+` `abs()`                                                | `__neg__` `__pos__` `__abs__`                                                                                        |
| Операторы сравнения                              | `<` `<=` `==` `!` `>` `>=`                                     | `__lt__` `__le__` `__eq__` `__ne__` `__gt__` `__ge__`                                                                |
| Арифметические операторы                         | `+` `-` `*` `/` `//` `%` `@` `divmod()` `round()` `**` `pow()` | `__add__` `__sub__` `__mul__` `__truediv__` `__floordiv__` `__mod__` `__matmul__` `__divmod__` `__round__` `__pow__` |
| Инверсные арифметические операторы               | (арифметические операторы с переставленными операндами)        | `__radd__` `__rsub__` `__rmul__` `__rtruediv__` `__rfloordiv__` `__rmod__` `__rmatmul__` `__rdivmod__` `__rpow__`    |
| Арифметические операторы составного присваивания | `+=` `-=` `*=` `/=` `//=` `%=` `@=` `**=`                      | `__iadd__` `__isub__` `__imul__` `__itruediv__` `__ifloordiv__` `__imod__` `__imatmul__` `__ipow__`                  | 
| Поразрядные операторы                            | `&` `\|` `^` `<<` `>>` ```~```                                 | `__and__` `__or__` `__xor__` `__lshift__` `__rshift__` `__invert__`                                                  |
| Инверсные поразрядные операторы                  | (поразрядные операторы с переставленными операндами)           | `__rand__` `__ror__` `__rxor__` `__rlshift__` `__rrshift__`                                                          |
| Поразрядные операторы составного присваивания    | `&=` `\|=` `^=` `<<=` `>>=`                                    | `__iand__` `__ior__` `__ixor__` `__ilshift__` `__irshift__`                                                          |

!!! note ""
    Python вызывает инверсный метод от имени второго операнда, если нельзя использовать соответственный метод от имени
    первого операнда.  
    Операторы составного присваивания – сокращенный способ вызвать инфиксный оператор с последующим присваиванием переменной, например `a += b`.
