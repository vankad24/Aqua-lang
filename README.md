# Язык программирования Aqua
## О языке
- Думаете в **Aqua** много воды? Наоборот, никакой воды, только кристально чистый код!
- Язык общего назначения
- Компилируемый
- Строго типизированный
- Большая часть работы ложится на компилятор. Он автоматически определяет большинство типов переменных и возвращаемых значений. Также на компилятор ложится ответственность за безопасность по памяти.
- Реализует ООП: одиночное наследование, полиморфизм и нестрогая инкапсуляция
- Имеет интроспекцию (можно узнать класс, к которому принадлежит объект), но не рефлексию.
- Любое значение является объектом, а любая операция — вызовом метода
- Есть сходства с Java, Python, JS, Kotlin, Rust, Dart, C++.

Хотелось бы видеть:
- Поддержка работы с указателями и памятью, как возможный конкурент C++
- Совместимость с классами C++ и библиотеками .dll
- Многопоточность
- Компиляция в js, web-assembly или другое
- Удобный и простой в использовании пакетный менеджер, как pip в python
- Компиляция в библиотеку (как dll в С++)
- Ведение AEP Aqua Enhancement Proposals
- Генерация документации, как в JavaDoc
- Скомпилированные файлы и библиотеки можно было бы запустить, открылось бы красивое окно с перечнем классов и функций с их сигнатурами, которые в этом файле скомпилированы.
### Целевые применения
- Написание серверов
- Математические вычисления
- Написание ПК и консольных приложений
### Принцип работы с памятью
- По умолчанию всё везде передаётся по ссылкам, кроме int, float, long, char и других, т.к. к ним пременён модификатор `autocopy`.
- В языке всё должно быть инициализировано, никаких null.
- Используется концепция, схожая на владение в Rust
- Существует 3 вида сущьностей: **object, read-only reference, mutable reference**.
```c
Animal a //object
a2 = a  //read-only reference
a3 = &a //mutable reference
a4 = $a //copied object
```
- Одновременно может быть только одна **mutable reference** или много **read-only reference**. Одновременно и того и другого быть не может, чтобы не получилась ситуация, что из одного потока объект изменяют, а из другого читают. При выходе из области видимости они удаляются.
- Через **read-only reference** нельзя вызывать у объекта изменяющие функции. К этим функциям относятся те, что вызывают оператор `=` у какого-то из полей класса. Это свойство учитывается только на этапе компиляции.
- **mutable reference** ведёт себя, как и **object**, позволяет изменять объект
- Можно просто в коде писать фигурные скобки `{ ... }`, чтобы создать область видимости, при выходе из которой объекты удалятся.
## Синтаксис
### Синтаксическте нормы
- Использование ; только при написании в одной строке (как в js)
- Комментарии с помощью `//` и `/* */`
- Табуляция не важна
- Названия классов с большой буквы PascalCase
- Названия функций через camelCase
- Названия переменных через snake_case
- Константы БОЛЬШИМИ буквами
### Общее
- Программа запускается из главного файла, а не из функции main. Запускаемый файл по стандарту должен называться **`main.aq`**. Скомпилированный object файл будет называться **`main.water`**. Скомпилированная статическая библиотека - **`library.snow`**, динамическая библиотека - **`library.ice`**
- В исключениях пишется строка ошибки. Как минимум, в debug режиме
- Инициализация переменной: `a = Animal(24)`, объявление переменной - `Animal a`. Объявлённые переменные должны быть обязательно инициализацированы, иначе будет ошибка компиляции.
- Всё построенно на классах. Даже int является alias от Integer. Поэтому будет работать код `int.class.name`. Хотя неявно компилятор старается использовать примитивный int, где возможно.
- Цикл foreach - `for (var_name : iterable/Number) { ... }` Пример: `for (x : arr) { ... }`, где для `x` автоматически определится тип данных исходя из массива `arr`. Пример с распоковкой: `for (key, value : dict.pairs())`. Конструкция `for (i = 3 : arr.len) { ... }` равносильна `for(int i = 3; i<arr.len; i++){ ... }`. Обычный цикл for тоже есть. 
- Возможность возвращать из функции несколько значений через запятую: `return .4 , 5, "hi"`. Они преобразуются в кортеж `Tuple<double,int,str>`. Для передачи элементов как массив, следует использовать `return [.4, 5, "hi"]`, они преобразуются в `Array<Any>`.
- Swap переменных `a,b = b,a`, как в python
- Групповое, множественное присваивание: `a,b,c = 1,3,2`, `a,b,c = 0`, но `a,b,c = 1,2` - compilation error
### Операторы
- Переопределение операторов синтаксисом: `_+=(int num){ ... }`, `_==(int num) => this.len == num`
- `**` - возведение в степень
- `==` - сравнение по значению (вызов реализованной в классе функции сравнения). По умолчанию поочерёдно вызывает `==` у полей класса.
- `&==` - сравнение по ссылке. Не может быть переопределён.
- `.=` - вызов функции с присваиванием. Пример: `obj.=fun()` равносильно `obj=obj.fun()`
- `...` - произволькое количество аргументов (как в java). По сути, это массив. После аргумента `...` могут идти и другие аргументы, но они должны иметь значение по умолчанию. 
- `:Class` - преобразование типа (cast). Пример: `value:str` будет аналогичным `(String)value` в java
- `#` - распаковывает iterable объекты (как `*` в python). Если в iterable объекте больше элементов, чем нужно, то оставшиеся он проигнорирует. Если меньше - вызовет exception. Пример: `println(#arr)`, `int a, str b = #foo()`, `int a,b,c = #(1,4,0) arr` - индексы элементов, которые нужно взять, `#(2:5)` - равносильно `#(2,3,4)`, `#(:-1)` - до предпоследнего элемента, `#(1:)` - с индекса 1 до конца iterable.
- `.` - по умолчанию - это обращение к полям и методам объекта. Если такого поля в классе нет, то вызывается оператор `_.(str s){ ... }`, где можно реализовать свою логику
- `$` - оператор копирования (deep copy). Нельзя переопределить. Копирует не указатели, а все элементы. Может продлить жизнь объекту. Пример `return &obj` (obj не будет уничтожен при выходе из области видимости и копирования не произойдёт). Ещё пример: `$&obj` - сначала вернёт ссылку, а потом увеличит время жизни ссылки. 
- `!=` - логическое "не равно"
- `=` - оператор присваивания может быть переопределён только для других типов, т.е. `_=()` может принимать все типы, кроме своего. Присваивание к такому же типу означает перезапись ссылки на объект.
- `[:]` - оператор срезов. `:` может быть и несколько: `arr[1:5:-1]`. При пропуске значения `arr[:3]` вызовется оператор `_[](,3)`. Для этого в классе должно быть определено значение по умолчанию: `_[](int start = 0, int end = this.len){ ... }`
- `&&`, `||`, `!` - логические операторы.
- `<<`, `>>`, `~`, `^`, `&`, `|`  - битовые операторы.
- `?` - ?
- `&` - ?
- `*` - ?
- `::` - ?
### Функции
- Синтаксис: название, аргументы, тело функции. Если функция состоит из одной строчки, вместо фигурных скобок можно написать `=>`. Если будет что возвращать, оно вызовет `return`. Перед функцией возвращаемый тип не указывается, он определяется автоматически. Если нужен другой тип, то делаем cast к этому типу. Пример:
```c
sum(int a, int b) => a+b
mul(float a, float b){
	return (a*b):int
}
```
- Поддержка как перегрузки функций, так и значений по умолчанию. Причём в значения по умолчанию можно записывать выражения или вызывать функции. Также, аргумент можно пропускать через `,,`, если указано его начальное значение. Пример:
```c
foo(int<> l, int len = l.size(), int sqrt = Math.sqrt(len)){ ... }
foo(<1,2,3>,,4)
```
- Передача параметров не по порядку, как в python: `println("Hello, there",24, end="!!!")`
- Callback функция, которая передаётся как параметр, будет без проблем вызываться, даже если она принимает меньше параметров (как в js). Пример: в функцию `foo(Function<Event, int, int => ()> fun) ` можно передать как `bar(Event e, int a, int b)`, так и `baz(Event e)`
- Анонимные функции: `foo = (int a, int b = 3)=>{ return a+b }` или `bar = (List l) => l.size()`. Такие же как стрелочные функции в js. Если передаём функцию как параметр, значит известна её сигнатура и можно записать в краткой форме: `foo( e=>e.getX() )` и `foo( =>it.getX() )` или `foo( (a,b) => { int c = a+b; return c } )` и `foo(=> it+it[1])`
- Допустим, есть перегруженная функция `foo(str s)` и `foo(Any any)`. При вызове `foo("hi")` и `foo(5)` вызовется первая и вторая соответственно.
## Встроенная библиотека
### Стандартные функции
- print(arguments..., split=' ') - печатает `arguments` в консоль с разделителем `split`. Вызывает `:str` у аргументов
- println(arguments..., split=' ', end='\n') - как print, только в конце печатает `end`
- help(obj) - печатает документацию, в том числе путь до исходника
- type(obj) - возвращает строку: имя класса объекта `obj.class.name` и, если есть, дженерики в `<>` скобках `obj.class.generics`. Пример: `println(type(dict))` **->** `"Dict<int, str>"`.
- inputLine(msg, end_line="\n") - работает как `input()` в python
- input(msg, split=" ", end_line="\n") - если буфер пуст, то считывает строку, но возвращает только слово до разделителя `split`. Остальное записывает в буфер
- inputClear() - очищает буфер, который мог заполниться функцией `intput()` 
- sizeOf(obj) - размер объекта в байтах
- assert(condition, msg) - функция которая работает только в debug режиме. Если `condition == false` - печатается `msg` и кидается exception
### Классы
- `Any` - класс, от которого неявно все наследуются (как Object в java). Имеет снандартно реализованные функции:
  - `:str=>type(this)` - перевод в строку по умолчанию
  - `const Class class` - переменная, содержащая ссылку на экземпляр своего класса. При приведении типа сравнивается реальный тип и тот, к которому мы приводим.
  - `hash` - ?
  - `==` - сравнение по умолчанию (вызов `==` для каждого из полей)
- `Iterable` - абстрактный класс. Обязывает реализовать оператор `[]` и имеет длинну `len`. Благодаря ему будет работать цикл for each
  - next() - как в python
  - map(function/class) - как в python. `class` должен иметь конструктор, который принимает такой же тип, как элементы в iterable.
  - filter(function) - возвращает объект того же типа как в python
  - sort(comparator) - возвращает массив, как в python
  - max(comparator)
  - min(comparator)
  - any(function) - как в python
  - all(function) - как в python
  - none(function) - как в kotlin
  - forEach(function) - как в js
  - reduce(function, start_value=this[0]) - как reduce в functools в python
  - contains(value) - return true если значение есть в iterable.
  - where(function) - Возвращает первый элемент, удовлетворяющий условию, иначе - возвращает последний элемент iterable. function возвращает тип bool. 
- `Array` - стандартный неизменяемый массив. Реализует iterable. Конструктор может принимать iterable.
  - Стандартный синтаксис `arr = Array<int>(3)`
  - Короткий синтаксис `arr = int[3]`
  - `int[] arr` - объявление без инициализации
  - Инициализация `arr = int[]` равносильна `arr = Array<int>()`, то есть длина будет 0.
  - Объявление `int[][] arr` равносильно `Array<<int>> arr`. Это настоящий двумерный массив, в отличие от DimArray.
  - Может создаваться синтаксисом `arr = [1,2,4]`
  - Конструктор из строки: `arr = Array<int>("[1.2,2 , 4]")` - приведёт всё к int
  - Иммеет длину `const int len`
  - Имеет красивый вывод в консоль `[1,2,4]`
  - Встроенные функции, такие как indexOf, binarySearch и т.д.
  - Поддержка отрицательных индексов
  - Срезы синтаксисом `[from:to]`. Необязательно указывать оба параметра. То есть `s[,5]` равносильно `s[0,5]`. Поддерживает отрицательные индексы.
  - Синтаксис `arr = [5]` или `[] arr` недопустим. Следует писать `Array arr`
  - Для инициализации массива объектами используется конструктор без параметров. Для эффективности создаёт один такой объект, а далее копирует до конца массива.
  - При вызове конструктора по умолчанию `arr = Array<int>()` создаётся массив длинны 0. Его можно переприсвоить `arr = $arr2`, const len изменится.
- `List` - расширяемый массив (как ArrayList в java)
  - Стандартный синтаксис `list = List<int>(5)`
  - Короткий синтаксис `list = int<5>` или `list = int<>`
  - `int<> list` - объявление без инициализации
  - Может создаваться синтаксисом `List([1,2,4])`
  - В консоль выводится в виде `<1,2,3>`
  - Синтаксис `list = <>` или `<> list` недопустим. Следует писать `List list`
- `DimArray` - (dimensional array) ведёт себя как многомерный массив, но внутри является одномерным. Неизменяемый.
  - Имеет поле `shape` (размеры измерений). Функция `reshape()` изменяет размерность
  - Оператор `[]` возвращает `DimArray`, поэтому, чтобы получить значение, нужно использовать функцию `value()`
  - Имеет горизонтальные и вертикальные срезы, поддержкой шага, как в numpy
  - Статистические функции, такие как `mean`, `median`, `mode`
  - `groupBy(column, axis)` - группирует данные (как в pandas). Возвращает объект `DimArray`
  - Работа с матрицами: `determinant()`, `transpose()`
- `LinkedList` - массив с быстрым удалением за счёт переписывания ссылок.
- `Dict` - словарь
  - Стандартный синтаксис `dict = Dict<int,float>()`
  - Короткий синтаксис `dict = {int : float}`
  - `{int : float} dict` - объявление без инициализации
  - Может создаваться синтаксисом `{1:24, 2:12}`
  - Обращение может происходить как через `[]` - `dict["hi"]`, так и через `.` - `dict.hi`, т.к. переопределён оператор `.`
  - Имеет поля `keys` и `values`
  - Есть функция `pairs()`, как в python
- `Set` - словарь
  - Стандартный синтаксис `set = Set<int>()`
  - Короткий синтаксис `set = int{}`
  - Инициализация синтаксисом `set = {1,2,3}`
- `Tuple` - кортеж. Используется для храниния разных типов в одной структуре
  - Стандартный синтаксис `tuple = Tuple<int, str, float>()`
  - Короткий синтаксис `tuple = (1, "hello", 4.1)`
  - Есть оператор сравнения, длина len, не относится к Iterable.
  - Обращение к элеметам через `[]`. Индекс должен быть константным. Пример: `num = tuple[0]`. Тип указывать не нужно, т.к. он известен у tuple.
  - Можно менять элементы: `tuple[0] = 24`
  - Именование кортежа: `tuple = (one,t,th)(1,2,3)` или `tuple = (one,t,th) Tuple<int, str, float>()`, чтобы потом обращаться как `tuple.one`
  - Часто используется с оператором `#`. `a,b,c = #tuple` - распакует кортеж и создаст переменные с нужными типами.
- `Counter` - наследуется от Dict
  - `count(obj)` - функция создаст во внутреннем словаре пару `obj:1`, а если `obj` уже есть, то увеличит счётчик
  - В конструктор можно передать объект iterable, где для каждого элемента вызовется функция `count(obj)`
- `Json` - тот самый json (наследуется от Dict<str,Json>)
  - Имеет конструктор, принимающий str
  - `pretty()` - возвращает json строку в приятном виде
  - Внутри имеет поля типов `str` - значение в виде строки, `Array<Json>` - значение  массив. При обращении через `[]` обращается во внутренний словарь и возвращает `Json` объект.
  - Получение данных из Json: , `json.pages[1].views:int`, либо благодаря `autocast` написать `int views = json.pages[1].views`
  - `str()` - функция возвращает значение узла: `json.pages[1].title.str()`. Нельзя использовать `:str`, т.к. оно используется при печати в консоль
  - `array()` - возвращает `Array<Json>.map(=>it.str())`.
  - `jsonArray()` - функция возвращает массив `Array<Json>`.
  - `isNull()` - возвращает `str == "null"
  - `isEmpty()` - возвращает `this.isEmpty() && str.isEmpty() && array.isEmpty()`
  - Запись в Json: `json.page.number = 5` - если ключа `number` в словаре не было, создаст его. `json.pages = [1,"hello"]` - автоматически конвертирует массив в `Array<Json>`. 
  - Немного о реализации: cast вызывается только для внутренней str, оператор `_[](int i)` возвращает элемент из внутреннего массива, а оператор `.` или `_[](str s)` возвращают Json
  - Применён `autocast` для int, long, double, float, но не для str. Примеры: `int views = json.pages[1].views`, `int rate = json.pages[1].rate:float`
  - Если в входной строке json будет значение `{key:[]}`, то массив будет иметь длину 0, а если `{key:{}}`, то объект будет иметь пустой массив и пустую строку.
- `HashDict`
- `HashList`
- `INumber` - интерфейс, от которого наследуются все численные типы.
  - Все числа имеют константы MAX и MIN.
  - Функции по типу `isSafeAdd(n1,n2)` которая возвращает false, если при сложении переменная переполнится.
- `IFloat` - интерфейс, от которого наследуются float и double.
- `IIntger`- интерфейс, от которого наследуются все целочисленные типы: short, int, long
- `SystemInt` - имеет размер разрядности системы: 4 или 8 байт. Alias - `sint`. Беззнаковое число (Unsigned)
- `BigLong` - Long занимает 64 бита, а BigLong 128. Есть alias `blong`
- `BigNumber` - класс с реализацией длинной арифметики
  - В конструктор принимает int - количество байт, которое занимает число
- `LongNumber` - класс с реализацией длинной арифметики
  - В отличие от предыдущего, может хранить число неограниченного размера
- `Class` - все классы неявно будут экземплярами этого класса
  - `name` - имя класса
  - `package` - имя пакета
  - `children` - массив классов-наследников
  - `size` - минимальный размер объекта в байтах (размер указателя 4 или 8 байт)
  - `generics` - массив с типами дженериков. Он может быть и пустой.
  - Через точку можно обратиться к static переменным, либо к функциям класса. Для их вызова функций первым параметром обязательно нужно будет передать объект этого же класса (передать this).
  - Оператор `class.new()` вызовет конструктор.
- `Function` - все функции неявно будут экземплярами этого класса. Сигнатура описывается как: `аргументы => возвращаемое_значение`. Если функция ничего не возвращает, ничего не пишется. Пример: `Function<int, Any => >`, `Function< => >` - функция, которая ничего не принимает и ничего не возвращает.
  - `name` - имя функции
  - `type` - тип возвращаемого значения
  - `arguments` - массив из типов принимаемых аргументов
  - Оператор `()` исполняет эту функцию
  - Сигнатура описывается как `Function<float, float => int> f` - принимает два float, возвращает int, `Function<()=>void> f`- ничего не принимает, возвращает что угодно или ничего не возвращает.
- `String` - является массивом из `char`. Является Iterable
  - Стандартный синтаксис `string = String()`
  - Короткий синтаксис `string = ""` или `string = ''`
  - Нет разницы между `'` и `"`. Чтобы создать символ, нужно использовать явный тип: `char c = "!"`
  - Встраивание переменных в строку через `$`. Пример: `println("Hello, $i")` или можно заключать выражения в скобки `$( )`,`${ }`: `println("Hello, $(i+1)")`. `"${=a}"` преобразуется в `"a="+a`.
  - Может быть изменена отдельная буква: `s[2] = 'h'`
  - Поддержка отрицательных индексов
- `Regex` - регулярные выражения
  - Конструктор, котрый принимает строку `Regex("a[bcd]+")`
  - Имеет короткий синтаксис `/my regex/` (как в js)
- `Pointer` - указатель. Абстрактный класс.
  - ???Статический метод `Pointer.from(T obj)`, чтобы получить указатель из объекта
  - Оператор `_[](int i)` вызывает код, равносильный `*(ptr+i*sizeof(Type))` и возвращает ссылку на объект.
  - Внутри хранит начальный адрес памяти. В debug режиме хранится и длина выделенной памяти, проверяется выход за пределы.
  - Указатель можно получить, воспользовавшись экземпляром класса `Allocator`. 
  - Внутри классов можно вызвать `allocate()` для получения указателя. Этот метод неявно вызовется у стандартного аллокатора или у иного, если он был указан при инициализации.
  - Метод `isNull()`
- ???`ObjectPointer` - указатель на другой указатель.
  - Память по этому адресу не удаляется, поскольку она принадлежит другому объекту и именно он должен очистить её.
  - Не является наследником `Pointer`
  - Имеет конструктор `ObjectPointer(Pointer)`
- `Time` - класс для работы с временем
  - `millis()` - возвращает текущее время в миллисекундах
  - `nanos()` - возвращает текущее время в наносекундах
  - `measureTime(function)` - возвращает время выполнения function в наносекундах
- `Date` - класс для работы с датами
- `ComplexNumber` - класс для работы с комплексными числами
- `Allocator` - стандартный аллокатор, который по умолчанию выделяет память для объектов
  - При инициализации перед любым конструктором может быть написан класс-наследник аллокатора. Экземпляр этого класса будет использоваться для выделения памяти. Пример: `list = (LinearAllocator) new int<>`
  - Имеет методы `alloc()`, `realloc()`, `free(Pointer)`
- `Database` - абстрактный класс для управления бд (как библиотека room в java)
  - Нужно переопределить методы, чтобы составлялись нужные sql запросы.
  - Подключение к БД нужно реализовывать в функции `connect()`
- `Void` - специальный класс. Обозначает, что функция ничего не возвращает. Есть `alias Void void`
  - `type(void)` вернёт "Void"
  - `println(println())` напечатает `\n void` т.к. функция `println` возвращает `void`
  - `sizeOf(void)` будет 0. То есть может играть роль некой заглушки.
## ООП
- Возможно только еденичное наследование и множественная реализация интерфейсов. Можно наследовать интерфейс от интерфейса.
- Синтаксис наследования через двоеточие: `class MyArray : Array, Iterable{ ... }`
- По умолчанию все поля и методы public
- К private и protected полям можно обращаться через `_`, как в python
- Модификаторы доступа можно писать как перед функцией / переменной (`public int main(){ ... }`), так и оборачивать всё в блок (`public{ int main(){ ... } }`)
- Дженерики. Пишутся перед функцией или после имени класса. Их можно сокращать: `Array<Array<int>>` равносильно `Array<<int>>`. Можно присваивать: `<T>foo(Class cls){ T = cls }`
- При инициализации можно реализовывать интерфейс или абстрактный класс, как в java. Но таким же образом можно переопределять методы уже готового класса. Сначала переопределяется класс, потом вызывается конструктор. Пример:
```c
arr = int[5]{
    indefOf(int i){ ... }
}
```
- Возможность создавать функции и классы внутри функций. При выходе из области видимости они бы удалялись.
- Для всех полей класса должен быть написан тип, а инициализироваться в конструкторе они должны через `new`
- Все поля обязательно инициализируются в конструкторе, если для них не было установлено начальное значение. Для тех, которые не были инициализированы неявно вызывается конструктор по умолчанию. Для int и других численных типов задастся значение 0.
- В классе можно создавать переменные того-же класса и напрямую обращаться к private полям. Снаружи так можно делать только через `_`
- Хоть всё является объектами, там, где возможно, компилятор будет заменять Integer на примитивный int для эффективности. Класс занимает больше места, чем просто int. Пример: `arr = int[5]` - используется примитивный тип int. `Any num = 5` - объектный тип int, поскольку ему нужно помнить, что он относится к классу int.
- В классах неявно создаётся конструктор по умолчанию без параметров. Он вызывает конструктор по умолчанию для всех полей. Можно его переопределить, но нельзя поставить private, т.к. он используется для инициализации массивов.
- `static class` означает, что класс не имеет конструктора и это просто набор полей и функций. Как namespace в C++, только обращение через `.`. Пример такого класса - `Math`
- Именованные конструкторы. Пример: `MyArray.zeros(int len){ ... }`, который заполняет массив нулями. Именованные конструкторы можно использовать и с **new**: `arr = new.zeros(5)`. Конструкторы `MyArray(int len)` и `MyArray.zeros(int len)` не будут конфликтовать.
- Наследник может переопределять даже private методы, но для этого нужно написать `_` перед функцией
- Статичный метод может быть вызван даже у абстрактного класса.
- Во внутренней реализации у каждого объкта хранится указатель на его класс. Можно попробовать хитро хранить указатель на объект и класс в одном указателе.
## Ключевые слова
- `this` - ссылка на текущий объект. Может использоваться для вызова конструктора this(args), как в java. При использовании в лямбда функции будет являтся ссылкой на объект, от которого вызвана функция. Пример: `arr.forEach(=> println(this.len+it) )`, где `this` является ссылкой на `arr`
- `parent` - ссылка на класс-родитель. Может использоваться для вызова конструктора parent(args), как `super` в java. После вызова конструктора `parent` необходимо инициализировать не все поля, а только те, которые не инициализацировал класс-родитель.
- `thisclass` - ссылка на текущий класс. Доступно и в static. Может использоваться для вызова конструктора thisclass(args). При наследовании станет ссылкой на текущий класс, а не на класс родителя.
- `it` - Кортеж на аргументы функции. Т.к. это tuple, имеет конкретный тип при обращении через `[]`. Если обращаться без скобок - `it` становится ссылкой на первый аргумент функции. Равносильно `it[0]`. Чаще всего используется при передаче анонимной функции с пропущенным описанием параметров.
- `alias` - переименование функций и пакетов. Действует только в текущей области видимости. Пример: `alias текущее_имя новое_имя`. По умолчанию неявно применён `alias Integer int`, `alias Float float`, `alias String str`, `alias UnsignedInt uint` и т.д.
- `import` - подключение пакетов. Поддержка `as` (как в python), а так же `safely`. По умолчанию `import` добаляет всё в текущее пространство имён. То есть после `import Math` можно вызывать `sin(.5)`, но если использовать `import safely Math` нужно писать `Math.sin(.5)`. Если нет конфликта имён, можно писать только название пакета. Если же есть, нужно писать путь пакета `import aqua.Math` или `import lib.Math`. Можно и импортировать весь пакет: `import aqua.utils.*`
- `break/continue` - выход/пропуск цикла. Можно создавать метки выхода `break start` (как в java). Также можно выйти сразу из всех вложенных циклов через `break all`
- `autocast` - пишется перед определением оператора cast `:Class`, либо перед конструктором. Означает, что при несовпадении типов, будет неявно вызван cast к этому типу, либо конструктор. `autocast` определён для всех численных типов между друг другом. Пример: `int a = Math.sqrt(7)` автоматически приведёт double к int. `int num = mystr:float` - сначала приведёт число к float, затем к int.
- `elif` - `else if`, как в python
- `new` - вызов конструктора для переменных с уже известным типом. Так же используется для описания сигнатуры класса. `Class<new(str)> cls` - означает, что этот класс имеет конструктор, который принимает `str`. Пример:
```c
Arr<int> arr
arr = new(5)
```
- `autocopy` - пишется перед определением класса. Меняет поведение `=`: данные будут копироваться, а не создаваться ссылка. Оператор `&` всё так же будет создавать ссылку. Классы `Integer`, `Long`, `Float` и т.д. помечены этим словом.
- `native` - пометка для функций, что код реализован на низком уровне (для исходников). 
- `debug` - этим словом помечаются переменные или блок кода `debug{ ... }`, которые должны выполняться только в режиме **debug**. Например, проверка выхода за длинну массива проверяется только в debug. Обычные переменные не могут изменяться переменными `debug`. Например, есть `debug int a`. Мы не можем использовать `a` для обычных переменных: `int b = a + 5`, но можем написать `debug int c = a + b`.
- `test` - является пометкой для кода, который должен быть запущен в режиме тестов.
## Литералы
- Типовых литералов нет. Тип для чисел определяется автоматически. По умолчанию выбирает int, затем, если не хватает, берёт long и BigNumber. Если нужна переменная byte, то нужно явно указывать: `byte b = 10` или `char c = "~"`. Если переменная слишком большая для типа, она обрежется: `byte b = 1000`
- `_` - между цифрами для визуального разделения, как в java: `1_000_000`
- `e` - `million = 1e6`
- `0b` - binary
- `0` - octal
- `0x` - hex
- `\u` - для символов: `\u00F7`

## Примеры кода
### Hello world
```c
println("Hello, world!")
```
### Вывести таблицу умножения NxM
```c
n = input("Введите N"):int
m = input("Введите M"):int
arr = int[n][m]
for (i : arr.len) {
	for (j : arr[0].len) {
		arr[i][j] = (i+1)*(j+1)
	}
}

println("Таблица $n x $m")
for (line : arr)
	println(#line, split = "\t")
```
### Вывести сумму и произведение введённых чисел
```c
import Math

arr = inputLine("Введите ряд чисел").split().map(double)

s = arr.sum()
p = prod(arr)
println("Sum: $s")
println("Product: $p")
```
### Числа Фибоначчи
```c
n = inputLine("Введите N"):int
fib(int n) => n < 2 ? 1 : fib(n-1)+fib(n-2)
for (i : 9)
    print(fib(i)," ")
//1 1 2 3 5 8 13 21 34
```
### Найти самый часто повторяющийся элемент массива. Если их несколько, вывести все.
```c
import Counter
arr = inputLine("Введите ряд чисел").split().map(int)
c = Counter(arr)
max_value = c.values.max()
println( #c.filter(=>this[it]==max_value) )
```
### Найти факториал
```c
import Math
println(factorial(1000))
```
### Переписанный пример использования классов с сайта языка Dart (dart.dev)
```c
abstract class Item {
  void use()
}

class Chest<T> : Item {
  List<T> contents

  Chest(List<T> list) => contents = $list

  use() => print('${this.class.name} has ${contents.len} items.')
}

class Sword : Item {
  const damage = 5

  use() => print('${this.class.name} dealt $damage damage.')
}

class DiamondSword : Sword {
  const damage = 50
}

main() {
  chest = Chest<Item>([
    DiamondSword(),
    Sword(),
  ])

  chest.use()

  for (item : chest.contents) {
    item.use()
  }
}
```
# Не забыть
in, is как в python
Нужны ли скобки в if?
Вместо `namespace` использовать `static class`?
`extends`, `implements` - наследование как в java или через : ?
Для всех объектов сделать class - полем или функцией?
срезы массива? вертикальные стоил ли?
какие примеры кода ещё написать?

std::move
switch case как в kotlin
GraalVM, Truffle, LLVM, JIT компиляция
`args` - ссылка на текущие аргументы функции. Если в функцию передан аргумент `value`, можно обратиться к нему через `args.value`. Является словарём типа Any
`->` - проверяет, содержится ли элемент в контейнере (как в python). Пример: `if (x -> arr)`
ключевое слово required, compile, stackalloc, heapalloc
array zeros, ones, fill(index), raw
Ссылки в объектах и массивах
with
Макросы
