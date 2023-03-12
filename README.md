# Язык программирования Aqua
## О языке
- Язык компилируемый
- Поддержка работы с указателями и памятью, как возможный конкурент C++
### Целевые применения (схожи с python)
- Написание серверов
- Математические вычисления
- Написание ПК и консольных приложений


## Синтаксис
### Общее
- Использование ; только при написании в одной строке (как в js)
- Комментарии с помощью `//` и `/* */`
- Табуляция не важна
- Названия классов с большой буквы
- Создание переменной: `a = Animal(24)`
- Всё построенно на классах. Даже int является alias от Integer. Поэтому будет работать код `int.class.name`
- Цикл foreach - `for( var_name : iterable/int ){ ... }` Пример: `for(x : arr){ ... }`, где для `x` автоматически определится тип данных исходя из массива `arr`. Пример с распоковкой: `for (key, value : dict.pairs())`. Конструкция `for(i = 3 : arr.length){ ... }` равносильна `for(int i = 3; i<arr.length; i++){ ... }`.
- Возможность возвращать из функции несколько значений через запятую: `return value, 5, "hi"`. Они преобразуются в массив Arr: `[value, 5, "hi"]`.
- В исключениях пишется строка ошибки. Как минимум, в debug режиме
- Swap переменных `a,b = b,a` как в python
### Операторы
- Переопределение операторов синтаксисом `int _+_(int num){ ... }`
- `**` - возведение в степень
- `==` - сравнение по значению (вызов реализованной в классе функции сравнения)
- `&==` - сравнение по ссылке
- `.=` - вызов функции с присваиванием. Пример: `obj.=fun()` равносильно `obj=obj.fun()`
- `...` - произволькое количество аргументов (как в java). По сути, это массив. Может быть не только последним аргументом
- `:Class` - преобразование типа (cast). Пример: `value:str` будет аналогичным `(String)value` в java
- `#` - распаковывает iterable объекты (как `*` в python). Если в iterable объекте больше элементов, чем нужно, то оставшиеся он проигнорирует. Если меньше - вызовет exception. Пример: `println(#arr)`, `int a,b,c = #(1,4,0) arr` - индексы элементов, которые нужно взять
- `->` - проверяет, содержится ли элемент в контейнере (как `in` в python). Пример: `if (x -> arr)`
- `?` - проверяет на null. `if(?value)` равносильно `if(value != null)`
- `&` - ?
- `*` - ?
- `!!` - ?
- `::` - ?
### Функции
- Объявление функций как в kotlin: fun, название, аргументы. Возвращаемое значение определится автоматически, если оно не указано. Пример:
```c
fun sum(int a, int b) = a+b
fun int mul(float a, float b){
	return a*b
}
```
- Поддержка как перегрузки функций, так и значений по умолчанию. Причём в значения по умолчанию можно записывать выражения или вызывать функции. Пример:
```c
fun foo(List l, int len = l.length(), int sqrt = Math.sqrt(len)){ ... }
```
- Передача параметров не по порядку как в python: `println("Hello, there",24, end="!!!")`
- Callback функция, которая передаётся как параметр, будет без проблем вызываться, даже если она принимает меньше параметров (как в js). Пример: в функцию `foo(Function<void(Event, int, int)> fun) ` можно передать как `void bar(Event e, int a, int b)`, так и `void baz(Event e)`
- Анонимные функции: `foo = (int a, int b = 3)=>{ return a+b }` или `bar = (List l) => l.size()`. Такие же как стрелочные функции в js. Если передаём функцию как параметр, значит известна её сигнатура и можно записать в краткой форме: `foo(=>{ args[0] + args.value2 })`
## Встроенная библиотека
### Стандартные функции
- print(arguments..., split=' ') - печатает `arguments` в консоль с разделителем `split`
- println(arguments..., split=' ', end='\n') - как print, только в конце печатает `end`
- help(obj) - печатает документацию, в том числе путь до исходника
- type(obj) - возвращает имя класса объекта, то есть `obj.class.name`
- inputLine(msg, end_line="\n") - работает как `input()` в python
- input(msg,split=" ", end_line="\n") - если буфер пуст, то считывает строку, но возвращает только слово до разделителя `split`. Остальное записывает в буфер
- inputClear() - очищает буфер, который мог заполниться функцией `intput()` 
### Классы
- iterable - абстрактный класс
  - next(iterable) - как в python
  - map(iterable, function) - как в python
  - filter(iterable, function) - как в python
  - sort(iterable, comparator) - как в python
  - any(iterable, function) - как в python
  - all(iterable, function) - как в python
  - none(iterable, function) - как в kotlin
  - forEach(iterable, function) - как в js
  - reduce(iterable, function, start_value) - как reduce в functools в python
- Arr - стандартный неизменяемый массив. Реализует iterable
  - Иммеет длину `length`
  - Может создаваться синтаксисом `[1,2,4]`
  - Имеет красивый вывод в консоль
  - Встроенные функции, такие как indexOf, binarySearch и т.д.
  - Поддержка отрицательных индексов
  - Срезы
- List - расширяемый массив (как ArrayList в java)
  - Может создаваться синтаксисом `List([1,2,4])`
- Dict - словарь
  - Может создаваться синтаксисом `{1:24,2:12}`
- Set - словарь
- Counter - наследуется от Dict
  - `count(obj)` - функция создаст во внутреннем словаре пару `obj:1`, а если `obj` уже есть, то увеличит счётчик
- HashDict
- HashList
- Number - класс с реализацией длинной арифметики
  - В конструктор принимает int - количество байт, которое занимает число
- LongNumber - класс с реализацией длинной арифметики
  - В отличие от предыдущего, может хранить число неограниченного размера
- Class - все классы неявно будут экземплярами этого класса
  - `name` - имя класса
  - `package` - имя пакета
  - `children` - массив классов-наследников
  - `size` - размер в байтах
- Function - все функции неявно будут экземплярами этого класса
  - `name` - имя функции
  - `type` - тип возвращаемого значения
  - `arguments` - массив из типов принимаемых аргументов
  - оператор `()` исполняет эту функцию
- String
  - Встраивание переменных в строку через `$`. Пример: `println("Hello, $i")` или `println("Hello, $(i+1)")`
- Regex - относится к примитивным типам
  - Конструктор, котоый принимает строку `Regex("a[bcd]+")`
  - Имеет литерал `/my regex/` как в js
## ООП
- К private и protected полям можно обращаться через `_` как в python
- Модификаторы доступа можно писать как перед функцией / переменной (`public int main(){ ... }`), так и оборачивать всё в блок (`public{ int main(){ ... } }`)
- Дженерики как в java
- При инициализации можно реализовывать интерфейс или абстрактный класс как в java. Но таким же образом можно переопределять методы уже готового класса. Сначала переопределяется класс, потом вызывается конструктор. Пример:
```c
arr = Arr<int>(5){
	indefOf(int i){ ... }
}
```
## Ключевые слова
- `this` - ссылка на текущий объект. Может использоваться для вызова конструктора this(args) как в java 
- `thisclass` - ссылка на текущий класс. Доступно и в static. Может использоваться для вызова конструктора thisclass(args). При наследовании станет ссылкой на текущий класс, а не на класс родителя.
- `args` - ссылка на текущие аргументы функции. Если в функцию передан аргумент `value`, можно обратиться к нему через `args.value`
- `alias` - переименование функций и пакетов. Пример: `alias текущее_имя новое_имя`. По умолчанию неявно применён `alias Integer int`, `alias Float float`, `alias String str` и т.д.
- `import` - подключение пакетов. Поддержка `as` и `from` как в python, а так же `safely`. По умолчанию `import` добаляет всё в текущее пространство имён. То есть после `import Math` можно вызывать `sin(.5)`, но если использовать `import safely Math` нужно писать `Math.sin(.5)`.
- `namespace` - как namespace в C++, только обращение к нему через `.`
- `null` - спец. значение
- `break/continue` - выход/пропуск цикла. Можно создавать метки выхода `break start` (как в java). Также можно выйти сразу из всех вложенных циклов через `break all`
## Принцип с ссылками
- по умолчанию всё везде передаётся по ссылкам, кроме int, float и т.д.
- сборщик мусора, ручное удаление или владение ссылкой?

## Примеры кода
### Вывести таблицу умножения NxM
```c
n = input("Введите N"):int
m = input("Введите M"):int
arr = Arr<int>(n,m)
for (i: arr.length){
	for (j: arr[0].length){
		arr[i][j] = (i+1)*(j+1)
	}
}
	
println("Таблица $n x $m")
for (line : arr)
	println(#line, split = "\t")
```

# Не забыть

break all;
распаковка объектов (1, 0) : (arr)?
println(value:str + v2:str)
in как в python
переопределять при создании объекта

std::move
switch case как в kotlin
Класс Any, с методами (:str и hash) от которого неявно все наследуются
`var` - автоопределение типа. Создание переменной типа String: `var s = "hello"`. Его можно и опустить.
&& или and?
оставить сокращение Arr?
надо ли fun?
стоит ли использовать var?
`extends`, `implements` - наследование как в java или через : ?
оставить ли подобный стиль int[] a = Arr(4)?
какие примеры кода ещё написать?
