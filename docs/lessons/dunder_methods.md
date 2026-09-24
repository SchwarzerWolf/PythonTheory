# Дандер-методы

## Опрос: на каком уровне вы знакомы с ООП?

1. Я уже всё знаю про дандер-методы
   → [вам сюда](#vy-velikolepny)
2. Я знаю, что такое классы, но про дандер-методы не слышал(а)
   → [к дандер-методам](#dunder-metody)
3. Я никогда не сталкивался(ась) с ООП
   → [начать с основ](#osnovy-oop)

---

## Основы ООП {: #osnovy-oop }

### Зачем вообще нужны классы

Исторически класс — следующая ступень в развитии кода после функций. Если
раньше программы были маленькими и простыми и было достаточно группировать
код по смыслу, то с ростом сложности систем такой подход перестал работать.

Поэтому появилось ООП — концепция, которая объединяет данные (атрибуты) и
функции, которые с этими данными работают (методы).

Возьмём пример: есть данные о молекуле и функция, которая с ними работает:

```python
molecule = {"smiles": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C", "name": "Caffeine"}


def is_valid(mol_dict: dict) -> bool:
    return Chem.MolFromSmiles(mol_dict["smiles"]) is not None


is_valid(molecule)
```

Оно работает, и это прекрасно. Но данные и обрабатывающие их код живут отдельно друг от
друга — ничто не гарантирует, что в `mol_dict` вообще есть ключ `"smiles"`.
Опечатка в ключе всплывёт не сразу, а где-то в середине пайплайна, в рантайме.

**Как понять, что перед вами кандидат на класс:**

- вы таскаете один и тот же словарь (или несколько переменных) в несколько
  функций, и в начале каждой — проверка «а точно ли там есть нужный ключ»;
- у данных есть операции, которые имеют смысл только для них — `is_valid()`,
  `molecular_weight` — это про конкретную молекулу, а не про данные общего
  назначения;
- есть поля, которые не хочется менять в рантайме (например, `smiles` после
  создания молекулы);
- одни и те же данные меняются согласованно в нескольких местах кода —
  дешевле держать эту логику в одном месте (методе), чем размазывать по
  функциям;
- вы работаете с другими людьми независимо друг от друга и хотите
  зафиксировать **контракт** — что можно ожидать от этих данных;
- система становится большой, и хочется добавить ей структуры.

### Определяем класс
Класс — это шаблон. Определяется он так:

```python
class Molecule:
    def __init__(self, smiles: str, name: str) -> None:
        self.smiles = smiles
        self.name = name
```

Разберём по частям:

- `class Molecule:` — объявление класса, шаблона для будущих объектов.
- `def __init__(...)` — конструктор: код, который выполняется при создании
  объекта.
- `self` — ссылка на создаваемый объект. Через `self` объект сохраняет свои данные.
- `self.smiles = smiles` — **атрибут**: данные, которые хранит каждый объект.

Несколько правил определения класса:

- имя класса — `PascalCase` (`Molecule`, а не `molecule` или `molecule_class`);
- у каждого атрибута в `__init__` — аннотация типа (`smiles: str`), чтобы
  редактор и коллеги сразу видели, что ожидается;
- в `__init__` попадают только данные, без которых объект не имеет смысла —
  если параметр можно не передавать, у него должно быть значение по
  умолчанию, а не отдельный метод-заглушка;
- если атрибут не должен меняться снаружи объекта, его нужно отметить нижним подчеркиванием 
(`_smiles`), но об этом подробнее на других занятиях.

### Создаем объекты

В нашем случае объект (экземпляр) — конкретная молекула, созданная по определенному выше шаблону:

```python
caffeine = Molecule(smiles="CN1C=NC2=C1C(=O)N(C(=O)N2C)C", name="Caffeine")
aspirin = Molecule(smiles="CC(=O)OC1=CC=CC=C1C(=O)O", name="Aspirin")

print(caffeine.name)
print(aspirin.name)
# Caffeine
# Aspirin
```

`caffeine` и `aspirin` — два независимых объекта. У каждого своя память под
атрибуты: изменение `caffeine.name` никак не затронет `aspirin`.

### Методы

Метод — функция, определённая внутри класса. В зависимости от того, какой
доступ к данным ему нужен, метод бывает одного из трёх видов:

1. **метод экземпляра** — получает `self`, работает с данными конкретного
   объекта;
2. **метод класса** (`@classmethod`) — получает `cls` вместо `self`, не
   привязан к конкретному объекту;
3. **статический метод** (`@staticmethod`) — не получает ни `self`, ни `cls`,
   это просто функция, логически связанная с классом.

```python
class Molecule:
    def __init__(self, smiles: str, name: str) -> None:
        self.smiles = smiles
        self.name = name

    def is_valid(self) -> bool:
        return Chem.MolFromSmiles(self.smiles) is not None

    @classmethod
    def from_lookup(cls, name: str, smiles_db: dict[str, str]) -> "Molecule":
        return cls(smiles=smiles_db[name], name=name)

    @staticmethod
    def looks_like_smiles(text: str) -> bool:
        return Chem.MolFromSmiles(text) is not None
```

```python
caffeine = Molecule("CN1C=NC2=C1C(=O)N(C(=O)N2C)C", "Caffeine")
if not caffeine.is_valid():
    print("Molecule is invalid, nothing to do...")

known_smiles = {"Caffeine": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"}
caffeine2 = Molecule.from_lookup("Caffeine", known_smiles)
# classmethod — альтернативный конструктор, вызывается на классе, не на объекте

Molecule.looks_like_smiles("not a smiles")
# False — staticmethod, вызывается без объекта вообще
```

### Проблема: вроде понятно, как с этим работать, но хочется красиво {: #dunder-metody }

Например, хотелось бы пройтись по атомам молекулы вот так:

```python
for atom in mol:
    print(atom)
```

А потом делать обычный `print` и получать осмысленный результат, а не
служебный мусор:

```python
print(mol)
# <__main__.Molecule object at 0x7f8a3c0a5490>
```

Или сравнивать две молекулы вот таким кодом:

```python
print(mol1 == mol2)
```

---

## Решение: дандер-методы 

### Что это такое

Тут на помощь приходят дандер-методы — их также называют магическими
методами (magic methods).

**Дандер** — от *double underscore*: методы вида `__init__`, `__str__`,
`__eq__`. Вы уже пользовались одним из них — `__init__` вызывается каждый раз
при создании объекта.

Ключевая идея: у Python есть набор встроенных операций — `print()`, `len()`,
`==`, `+`, `for ... in ...` — и для наших объектов и классов он не
угадывает, как их выполнять, а **явно обращается к дандер-методу**. Другими
словами, `len(obj)` — это не магия, а синтаксический сахар над
`obj.__len__()`, а точнее над `type(obj).__len__(obj)` — Python ищет метод не
у самого объекта, а у его класса.

Если дандер-метод не переопределён, срабатывает поведение по умолчанию. Для
`__eq__` это сравнение **по адресу в памяти** (как `is`), а не по содержимому:

```python
Molecule("CCO", "Ethanol") == Molecule("CCO", "Ethanol")
# False — это два разных объекта, хотя формула одна и та же
```
Переопределять абсолютно каждый метод бессмысленно — обычно мы переопределяем
то, что делает синтаксис красивым. А варианты у нас есть следующие: 

| Выражение              | Что реально вызывает Python |
|-------------------------|------------------------------|
| `Molecule(...)`          | `__init__`                   |
| `print(obj)`, `str(obj)` | `__str__`                    |
| `repr(obj)`, вывод в консоли/списке | `__repr__`        |
| `obj1 == obj2`           | `__eq__`                      |
| `obj1 < obj2`            | `__lt__`                      |
| `len(obj)`               | `__len__`                     |
| `obj1 + obj2`            | `__add__`                     |
| `for x in obj`           | `__iter__`                    |
| `obj[i]`                 | `__getitem__`                 |
| `x in obj`                | `__contains__`                |
| `obj(...)`                | `__call__`                    |


### Определяем дандер-методы на примере `Molecule`

Разберём это на уже знакомом классе `Molecule`. Обратите внимание на
`__eq__`: он сравнивает не сырые строки SMILES, а **канонический** SMILES —
одна и та же молекула может быть записана разными строками, и без
канонизации такие записи считались бы разными молекулами.

```python
from rdkit import Chem
from rdkit.Chem import Descriptors


class Molecule:
    def __init__(self, smiles: str, name: str) -> None:
        self.smiles = smiles
        self.name = name
        self._mol = Chem.MolFromSmiles(smiles)

    def __repr__(self) -> str:
        # "техническое" представление — для отладки, консоли, логов
        return f"Molecule(name={self.name!r}, smiles={self.smiles!r})"

    def __str__(self) -> str:
        # "человеческое" представление — то, что видит пользователь в print()
        # если метод не определен, то вызывается __repr__
        return f"{self.name} ({self.smiles})"

    def __eq__(self, other: object) -> bool:
        # сравнение по каноническому SMILES, а не по адресу в памяти
        if not isinstance(other, Molecule):
            return NotImplemented
        return self.canonical_smiles == other.canonical_smiles

    def __iter__(self):
        # for atom in mol — перебираем атомы молекулы одним за другим
        return iter(atom.GetSymbol() for atom in self._mol.GetAtoms())

    @property
    def canonical_smiles(self) -> str:
        # не дандер: обычное свойство, чтобы не хранить лишний атрибут
        return Chem.MolToSmiles(self._mol)

    @property
    def molecular_weight(self) -> float:
        # не дандер: обычное свойство, чтобы не хранить лишний атрибут
        return Descriptors.MolWt(self._mol)
```

И что теперь можно делать:

```python
caffeine = Molecule("CN1C=NC2=C1C(=O)N(C(=O)N2C)C", "Caffeine")
aspirin = Molecule("CC(=O)OC1=CC=CC=C1C(=O)O", "Aspirin")

print(caffeine)
# Caffeine (CN1C=NC2=C1C(=O)N(C(=O)N2C)C)     — вызвался __str__

repr(caffeine)
# Molecule(name='Caffeine', smiles='CN1C=...')  — вызвался __repr__

caffeine == Molecule("Cn1cnc2c1c(=O)n(C)c(=O)n2C", "Caffeine (другая запись SMILES)")
# True — строки SMILES разные, но канонический SMILES совпадает

for atom in caffeine:
    print(atom)
# C
# N
# C
# ...              — вызвался __iter__, перебор по атомам молекулы
```

## Вы великолепны! {: #vy-velikolepny }

Вся теория на сегодня пройдена — вы разобрались и с классами, и с
дандер-методами. Если хочется закрепить на практике — вот задача без молекул,
чтобы потренироваться на новом материале. Ответ есть под спойлером —
заглядывайте туда после своей попытки, а не вместо неё.

### Задание: `Vector2D`

`Vector2D` — классический учебный пример для перегрузки операторов. Для тренировки 
реализуйте `Vector2D(x, y)` с арифметикой:

```python
v1 = Vector2D(1, 2)
v2 = Vector2D(3, 4)

print(v1 + v2)
print(v1 * 2)
print(v1 == Vector2D(1, 2))
print(len(v1))
```

??? "Ответ"
    ```python
    class Vector2D:
        def __init__(self, x: float, y: float) -> None:
            self.x = x
            self.y = y

        def __repr__(self) -> str:
            return f"Vector2D({self.x}, {self.y})"

        def __eq__(self, other: object) -> bool:
            if not isinstance(other, Vector2D):
                return NotImplemented
            return (self.x, self.y) == (other.x, other.y)

        def __add__(self, other: "Vector2D") -> "Vector2D":
            return Vector2D(self.x + other.x, self.y + other.y)

        def __mul__(self, scalar: float) -> "Vector2D":
            return Vector2D(self.x * scalar, self.y * scalar)
    ```

    Обратите внимание: `v1 * 2` работает, а `2 * v1` — нет, потому что у `int`
    нет метода для умножения на `Vector2D`. Чтобы это заработало, нужен ещё
    `__rmul__` — необязательный бонус, если хочется углубиться.

    `__len__` не реализован: Python требует, чтобы он возвращал `int`,
    а геометрическая длина вектора — `√(x² + y²)`,
    то есть `float`. Поэтому переопределять его таким образом невозможно 
