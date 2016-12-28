# Функции и записи <sup>[1](#1)</sup>

## Цели главы

Эта глава знакомит с двумя элементами языка PureScript: функциями и записями<sup>[1](#1)</sup>. Кроме того, мы увидим как структурировать программы на PureScript и как использовать типы для помощи в разработке программ.

Мы создадим простую программу для управления записной книжкой контактов. Этот код внесет некоторые новые идеи из ситаксиса PureScript.

В качестве интерфейса к нашему приложению мы будем использовать непосредственно оболочку интерпретатора PSCi в интерактивном режиме, но ничего не мешает написать интерфейс на JavaScript. На самом деле, мы это и сделаем в следующих главах, добавив валидацию форм и возможность сохранения и загрузки.

## Настройки проекта

Исходники для этой главы находятся в файле `src/Data/AddressBook.purs`. Этот файл начинается с объявления модуля и списка импортов:

```haskell
module Data.AddressBook where

import Prelude

import Control.Plus (empty)
import Data.List (List(..), filter, head)
import Data.Maybe (Maybe)
```

Здесь мы импортируем несколько модулей:

- Модуль `Control.Plus` , который определяет значение `empty`.
- Модуль `Data.List`, который содержится в пакете `purescript-lists` и может быть установлен с использованием Bower. Он содержит несколько функций, которые нам будут нужны для работы со связанными списками.
- Модуль `Data.Maybe`, который определяет типы данных и функции для работы с необязательными значениями<sup>[2](#2)</sup>. 

Обратите внимание что импортируемые имена явно перечислены в скобках. Это хорошая, хотя и необязательная, практика, которая помогает при чтении кода и разрешении конфликтов импортов.

Предпогая что вы склонировали себе репозиторий кода из книги, проект для этой главы может быть построен при помощи программы Pulp, используя следующие команды:

```text
$ cd chapter3
$ bower update
$ pulp build
```

## Простые типы

PureScript определяет три встроенных типа, которые соответствуют примитивным типам JavaScript: числа, строки и булевые значения. Они определены в модуле `Prim`, который неявно импортируется любым модулем. Эти типы называются `Number`, `String` и `Boolean` соответственно. Их можно увидеть в PSCi при помощи команды `:type`, выведя тип для некоторых простых значений:

```text
$ pulp psci

> :type 1.0
Number

> :type "test"
String

> :type true
Boolean
```

PureScript определяет также несколько других встроенных типов: целые, символы, массивы, записи и функции.

Целые отличаются от чисел с плавающей точкой типа `Number` отстутствием десятичной точки:

```text
> :type 1
Int
```

Символьные значения заключены в одиночные кавычки, в отличие от строковых значений, которые заключаются в двойные кавычки:

```text
> :type 'a'
Char
```

Массивы соответствуют массивам JavaScript, но вопреки JavaScript, все элементы массива в PureScript должны иметь один и тот же тип: 

```text
> :type [1, 2, 3]
Array Int

> :type [true, false]
Array Boolean

> :type [1, false]
Could not match type Int with Boolean.
```

Ошибка в последнем примере это ошибка от тайпчекера, который пытался _унифицировать_ (то есть сделать равными) типы двух элементов.

Записи соответствуют объектам JavaScript, и значения имеют тот же синтаксис как и значения объектов в JavaScript:

```text
> let author = { name: "Phil", interests: ["Functional Programming", "JavaScript"] }

> :type author
{ name :: String
, interests :: Array String
}
```

Этот тип обозначает что указанный объект имеет два _поля_, имя `name`, которое имеет тип `String` и интересы `interests`, которое имеет тип `Array String`,  то есть массив `String`.

Поля записей могут адресоваться через точку, сопровождающуюся именем поля:

```text
> author.name
"Phil"

> author.interests
["Functional Programming","JavaScript"]
```

Функции PureScript соответствуют функциям JavaScript. Стандартные библиотеки PureScript предоставляют массу примеров функций и мы еще увидим их в этой главе:

```text
> import Prelude
> :type flip
forall a b c. (a -> b -> c) -> b -> a -> c

> :type const
forall a b. a -> b -> a
```

Функции могут быть определены на верхнем уровне файла при помощи указания аргументов перед знаком равенства:

```haskell
add :: Int -> Int -> Int
add x y = x + y
```

Либо можно определять функции на вложенных уровнях файла с исходным кодом, используя обратную косую черту, после которой следует список имен параметров. Для того чтобы переключить PSCi в режим многострочного ввода, надо выбрать "режим копирования" при помощи команды `:paste`. В этом режиме объявления завершаются последовательностью _Control-D_: 

```text
> :paste
… let
…   add :: Int -> Int -> Int
…   add = \x y -> x + y
… ^D
```

Определив эту функцию в PSCi мы можем _применить_ ее к ее аргументам, указав аргументы через пробел после имени функции:

```text
> add 10 20
30
```

## Квантифицированные типы

В предыдущем разделе мы видели типы некоторых функций, определенных в модуле Prelude. Например, функция `flip` имеет следующий тип:

```text
> :type flip
forall a b c. (a -> b -> c) -> b -> a -> c
```

The keyword `forall` here indicates that `flip` has a _universally quantified type_. It means that we can substitute any types for `a`, `b` and `c`, and `flip` will work with those types.

Ключевое слово `forall` здесь обозначает что `flip` имеет _универсально квантифицированный тип_<sup>[3](#3)</sup>. Это означает что вы можете подставить любой тип вместо `a`, `b` или `c` и `flip` будет работать с этими типами.

Например мы можем выбрать тип `Int` для `a`, `String` для `b` и `String` для `c`. В таком случае мы можем _специализировать_ тип функции `flip` как

```text
(Int -> String -> String) -> String -> Int -> String
```

Мы не обязаны указывать в коде что мы хотим специализировать квантифицированный тип - это получается автоматически. Мы можем использовать `flip` как будто у него и так есть такой тип:

```text
> flip (\n s -> show n <> s) "Ten" 10

"10Ten"
```

В то время как мы можем выбрать любой тип для `a`, `b` и `c` мы должны быть последовательны. Тип функции, который мы передаем во `flip` должен соответствовать  типам остальных параметров. Именно поэтому мы передали строку `"Ten"` в качестве второго параметра и число `10` в качестве третьего. Переставить местами аргументы не получилось бы:

```text
> flip (\n s -> show n <> s) 10 "Ten"

Could not match type Int with type String
```

## Замечания об отступах

PureScript - язык, чувствительный к отступам, также как и Haskell, но не JavaScript. Это означает что пробелы/табуляции в вашем коде несут смысл и используются для группирования блоков кода, аналогично фигурным скобкам в JavaScript.

Если объявление занимает больше одной строки, то то все строки кроме первой должны быть предварены отступом, большим чем отступ первой строки.

Таким образом, данный код корректен:

```haskell
add x y z = x +
  y + z
```

А этот некорректен:

```haskell
add x y z = x +
y + z
```

Во втором случае компилятор попытается разобрать _два_ объявления, по одному на каждой строке.

В общем случае, все объявления, определяемые в одном блоке, должны быть предварены отступом одинаковой глубины. Например, в PSCi, объявления внутри выражения `let` должны быть выровнены отступами одинаково. Данный код корректен:

```text
> :paste
… let x = 1
…     y = 2
… ^D
```

А этот нет:

```text
> :paste
… let x = 1
…      y = 2
… ^D
```
Некоторые ключевые слова PureScript (такие как `where`, `of` или `let`) вводят новый блок кода, в котором объявления должны быть предварены отступами глубже:

```haskell
example x y z = foo + bar
  where
    foo = x * y
    bar = y * z
```

Обратите внимание что объявления для `foo` и `bar` отступлены дальше чем объявление `example`.

Единственное исключение из этого правила это ключевое слово `where` в объявлении заголовка модуля `module` в начале файла.

## Определяем наши типы

Хорошим первым шагом чтобы разобраться с новой проблемой в PureScript является написание определений типов для значений с которыми предстоит работать. Сначала, давайте определим тип для записей в нашей адресной книге:

```haskell
type Entry =
  { firstName :: String
  , lastName  :: String
  , address   :: Address
  }
```

Это определение определяет _тип-синоним_, который называется `Entry` - тип `Entry` эквивалентен типу справа от знака равенства: тип записи с тремя полями - `firstName`, `lastName` и `address`. Два поля для имени будут иметь тип `String`, а поле `address` будет иметь тип `Address`, определенный следующим образом:

```haskell
type Address =
  { street :: String
  , city   :: String
  , state  :: String
  }
```

Обратите внимание что записи могут содержать другие записи.

Теперь давайте определим третий тип-синоним для нашей структуры данных адресной книги, который будет представлять собой просто связанный список записей:

```haskell
type AddressBook = List Entry
```

Обратите внимание что `List Entry` это не тоже самое что `Array Entry`, который представляет собой _массив_ записей.

## Конструкторы типов и родá

`List` это пример _конструктора типов_. Значения не имеют тип `List` напрямую, а имеют тип `List a` для некоторого типа `a`. Таким образом, `List` принимает _аргумент типа_ `a` и _конструирует_ новый тип `List a`.

Обратите внимание что как и в случае применения функций, конструкторы типов применяются к другим типам просто составлением: тип `List Entry` это конструктор типов `List`, _примененный_ к типу `Entry` - он представляет собой список записей.

Если мы попробуем некорректно определить значение типа `List` (используя оператор аннотации типа `::`), то мы увидим новый тип ошибки:

```text
> import Data.List
> Nil :: List
In a type-annotated expression x :: t, the type t must have kind *
```

Это ошибка _рóда_. Прямо как значения различаются по их _типам_, типы различаются по их _родам_, и точно также как неправильно типизированные значения приводят к ошибкам _типов_, неправильно "порожденные" типы приводят к ошибкам _родов_.

Есть специальный род `*`, который представляет собой род всех типов, которые имеют значения, например `Number` и `String`.

Также существуют родá для конструкторов типов. Например род `* -> *` представляет функцию из типа в тип, ровно такую как `List`. Поэтому ошибка выше возникла из-за того что значения имеют тип с родом `*`, в то время как `List` имеет род `* -> *`. 

Для того чтобы определить род типа можно использовать команду `:kind` в PSCi. Например:

```text
> :kind Number
*

> import Data.List
> :kind List
* -> *

> :kind List String
*
```

_Система родóв_ в PureScript поддерживает и другие интересные родá, которые мы еще увидим позже в этой книге.

## Вывод записей из адресной книги

Let's write our first function, which will render an address book entry as a string. We start by giving the function a type. This is optional, but good practice, since it acts as a form of documentation. In fact, the PureScript compiler will give a warning if a top-level declaration does not contain a type annotation. A type declaration separates the name of a function from its type with the `::` symbol:

Давайте напишем нашу первую функцию, которая будет отображать запись из адресной книги как строку. Начнем с того что дадим нашей функции тип. Это вообще необязательно, но считается хорошим тоном, поскольку служит в каком-то смысле документацией. Объявление типа разделяет имя функции от ее типа символом `::`:

```haskell
showEntry :: Entry -> String
```

This type signature says that `showEntry` is a function, which takes an `Entry` as an argument and returns a `String`. Here is the code for `showEntry`:

Эта сигнатура типа сообщает что `showEntry` является функцией, которая принимает параметр типа `Entry` и возвращает `String`. Вот собственно код для `showEntry`:

```haskell
showEntry entry = entry.lastName <> ", " <>
                  entry.firstName <> ": " <>
                  showAddress entry.address
```

This function concatenates the three fields of the `Entry` record into a single string, using the `showAddress` function to turn the record inside the `address` field into a `String`. `showAddress` is defined similarly:


Этот код объединяет три поля записи `Entry` в единую строку, используя функцию `showAddress` для того чтобы превратить вложенную запись в поле `address` тоже в `String`. Функция `showAddress` определена подобным же образом:

```haskell
showAddress :: Address -> String
showAddress addr = addr.street <> ", " <>
                   addr.city <> ", " <>
                   addr.state
```

A function definition begins with the name of the function, followed by a list of argument names. The result of the function is specified after the equals sign. Fields are accessed with a dot, followed by the field name. In PureScript, string concatenation uses the diamond operator (`<>`), instead of the plus operator like in Javascript.

Определение функции начинается с имени функции, сопровождающемся списоком аргументов. Результат функции указывается после знака равенства. Поля записи адресуются через точку, после которой следует имя поля. Для объединения строк в PureScript используется оператор "ромб" (`<>`) вместо плюса, который применяется в JavaScript.

## Test Early, Test Often
## Тестируйте рано, тестируйте часто

The PSCi interactive mode allows for rapid prototyping with immediate feedback, so let's use it to verify that our first few functions behave as expected.

Интерактивный режим PSCi позволяет быстрое прототипирование с мгновенной обратной связью. Давайте посмотрим как его можно использовать чтобы убедиться что функции ведут себя как ожидается.


First, build the code you've written:

Сначала, скомпилируйте написанный код:

```text
$ pulp build
```

Next, load PSCi, and use the `import` command to import your new module:
Затем запустите PSCi и используйте команду `import` для того чтобы загрузить ваш новый модуль:

```text
$ pulp psci

> import Data.AddressBook
```

We can create an entry by using a record literal, which looks just like an anonymous object in JavaScript. Bind it to a name with a `let` expression:

Мы можем создать запись, используя запись-константу, которая выглядит совершенно как анонимный объект в JavaScript. Дайте ей имя при помощи выражения `let`:

```text
> let address = { street: "123 Fake St.", city: "Faketown", state: "CA" }
```

Now, try applying our function to the example:
Теперь давайте применим нашу функцию к этим данным:

```text
> showAddress address

"123 Fake St., Faketown, CA"
```

Let's also test `showEntry` by creating an address book entry record containing our example address:

Заодно проверим `showEntry` создав запись в адресной книге, содержащую наш пример адреса: 

```text
> let entry = { firstName: "John", lastName: "Smith", address: address }
> showEntry entry

"Smith, John: 123 Fake St., Faketown, CA"
```

## Creating Address Books
## Создаем адресные книги

Now let's write some utility functions for working with address books. We will need a value which represents an empty address book: an empty list.

Теперь давайте напишем несколько функций для работы с адресными книгами. Нам понадобится значение, представляющее пустую адресную книгу: пустой список.

```haskell
emptyBook :: AddressBook
emptyBook = empty
```

We will also need a function for inserting a value into an existing address book. We will call this function `insertEntry`. Start by giving its type:

Также понадобится функция для вставки значения в существующую адресную книгу. Назовем эту функцию `insertEntry`. Начнем с определения типа для нее:

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
```

This type signature says that `insertEntry` takes an `Entry` as its first argument, and an `AddressBook` as a second argument, and returns a new `AddressBook`.

Эта сигнатура говорит что `insertEntry` принимает параметр типа `Entry` в качестве первого аргумента и типа `AddressBook` в качестве второго, а возвращает значение типа `AddressBook`.

We don't modify the existing `AddressBook` directly. Instead, we return a new `AddressBook` which contains the same data. As such, `AddressBook` is an example of an _immutable data structure_. This is an important idea in PureScript - mutation is a side-effect of code, and inhibits our ability to reason effectively about its behavior, so we prefer pure functions and immutable data where possible.

Мы не модифицируем существующую структуру `AddressBook` напрямую. Вместо этого мы возвращаем _новую структуру `AddressBook`, которая содержит те же данные. Таким образом, `AddressBook` это пример _неизменяемой структуры данных_. И это одна из важнеших концепций PureScript - изменение данных это побочный эффект от исполнения программы и мешает нам анализировать ее поведение, поэтому мы предпочитаем чистые функции (без побочных эффектов) и неизменяемые структуры данных.

To implement `insertEntry`, we can use the `Cons` function from `Data.List`. To see its type, open PSCi and use the `:type` command:

Чтобы реализовать `insertEntry` мы можем использовать функцию `Cons` из модуля `Data.List`. Чтобы посмотреть ее тип, запустим PSCi и воспользуемся опять `:type`:

```text
$ pulp psci

> import Data.List
> :type Cons

forall a. a -> List a -> List a
```

This type signature says that `Cons` takes a value of some type `a`, and a list of elements of type `a`, and returns a new list with entries of the same type. Let's specialize this with `a` as our `Entry` type:

Эта сигнатура говорит что `Cons` принимает значение какого-то типа `a` плюс список элементов типа `a`, и возвращает новый список элементов этого типа. Давайте специализируем эту функцию, подразумевая что `a`  это наш типа `Entry`:

```haskell
Entry -> List Entry -> List Entry
```

But `List Entry` is the same as `AddressBook`, so this is equivalent to

Но `List Entry` это тот же `AddressBook`, так что это эквивалентно

```haskell
Entry -> AddressBook -> AddressBook
```

In our case, we already have the appropriate inputs: an `Entry`, and a `AddressBook`, so can apply `Cons` and get a new `AddressBook`, which is exactly what we wanted!

В нашем случае мы уже имеем все необходимые входные типы: и `Entry` и `AddressBook`, так что мы можем просто применить `Cons` и получить новый `AddressBook`, то есть ровно то что нам и надо!

Here is our implementation of `insertEntry`:

Вот наша реализация `insertEntry`:

```haskell
insertEntry entry book = Cons entry book
```

This brings the two arguments `entry` and `book` into scope, on the left hand side of the equals symbol, and then applies the `Cons` function to create the result.

Она вводит два параметра `entry` и `book` в область видимости, помещая их слева от знака равенства и затем применяет к ним  `Cons` для того чтобы получить результат.

## Curried Functions
## Каррированные функции 

Functions in PureScript take exactly one argument. While it looks like the `insertEntry` function takes two arguments, it is in fact an example of a _curried function_.

Функции в PureScript принимают только один аргумент. Кажется что `insertEntry` принимает два, но на самом деле это пример так называемой _каррированной функции_.

The `->` operator in the type of `insertEntry` associates to the right, which means that the compiler parses the type as

Оператор `->` в типе `insertEntry` ассоциируется направо, что означает что компилятор воспринимает это как

```haskell
Entry -> (AddressBook -> AddressBook)
```

That is, `insertEntry` is a function which returns a function! It takes a single argument, an `Entry`, and returns a new function, which in turn takes a single `AddressBook` argument and returns a new `AddressBook`.

Таким образом, `insertEntry` это функция, которая возвращает функцию! Она принимает единственный аргумент типа `Entry` и возвращает функцию, которая в свою очередь, принимает единственный аргумент типа `AddressBook` и возвращает новый `AddressBook`.

This means that we can _partially apply_ `insertEntry` by specifying only its first argument, for example. In PSCi, we can see the result type:

Это означает что мы можем _частично применить_ `insertEntry`, например указав только ее первый аргумент. Это можно увидеть в PSCi:

```text
> :type insertEntry entry

AddressBook -> AddressBook
```

As expected, the return type was a function. We can apply the resulting function to a second argument:

Как и ожидалось, тип возвращаемого значения это функция. Мы можем применить ее ко второму аргументу:

```text
> :type (insertEntry entry) emptyBook
AddressBook
```

Note though that the parentheses here are unnecessary - the following is equivalent:

Обратите внимание что скобки тут необязательны, данная запись совершенно эквивалентна:

```text
> :type insertEntry example emptyBook
AddressBook
```

This is because function application associates to the left, and this explains why we can just specify function arguments one after the other, separated by whitespace.

Это происходит потому что примение функции левоассоциативно и это объясняет почему мы можем просто указывать аргументы функции один за одним, разделяя их пробелом.

Note that in the rest of the book, I will talk about things like "functions of two arguments". However, it is to be understood that this means a curried function, taking a first argument and returning another function.

Заметим, что далее в книге я буду говорить о вещах типа "функции от двух аргументов". Но на самом деле это надо воспринимать как каррированную функцию, принимающую первый аргумент и возвращающую другую функцию, принимающую второй аргумент.

Now consider the definition of `insertEntry`:

Посмотрите на определение `insertEntry`:

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry entry book = Cons entry book
```

If we explicitly parenthesize the right-hand side, we get `(Cons entry) book`. That is, `insertEntry entry` is a function whose argument is just passed along to the `(Cons entry)` function. But if two functions have the same result for every input, then they are the same function! So we can remove the argument `book` from both sides:

Если мы явным образом заключим в скобки правую часть, то мы получим `(Cons entry) book`. Таким образом, `insertEntry entry` это функция, чей аргумент просто передается дальше в функцию `(Cons entry)`. Но если две функции возвращают один и тот же результат для любого значения, то значит это одна и та же функция! Поэтому мы можем удалить аргумент `book` с обоих сторон:

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry entry = Cons entry
```

But now, by the same argument, we can remove `entry` from both sides:

Но теперь, пользуясь той же логикой, мы можем удалить `entry` с обоих сторон:

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry = Cons
```

This process is called _eta conversion_, and can be used (along with some other techniques) to rewrite functions in _point-free form_, which means functions defined without reference to their arguments.

Этот процесс называется _эта преобразованием_<sup>[4](#4)</sup> и может быть использован (вкупе с некоторыми другими методами) для записи функций в _бесточечной форме_<sup>[5](#5)</sup>, что означает отсутствие указания аргументов.

In the case of `insertEntry`, _eta conversion_ has resulted in a very clear definition of our function - "`insertEntry` is just cons on lists". However, it is arguable whether point-free form is better in general.

В случае с `insertEntry`, _эта преобразование_ привело к очень ясному определению нашей функции - "`insertEntry` это просто cons на списках". Однако, преимущества записи в бесточечной форме в общем случае спорны. 

## Querying the Address Book
## Запросы к адресной книге

The last function we need to implement for our minimal address book application will look up a person by name and return the correct `Entry`. This will be a nice application of building programs by composing small functions - a key idea from functional programming.

Последняя функция, которую надо определить для нашего минимального приложения, будет запрашивать адресную книгу по имени персоны и возвращать необходимую запись `Entry`. Это будет прекрасное приложение метода построения программы из композиции маленьких функций - ключевой идеи функционального программирования.

We can first filter the address book, keeping only those entries with the correct first and last names. Then we can simply return the head (i.e. first) element of the resulting list.

Сначала мы можем профильтровать адресную книгу, сохраняя только те записи у которых нужные нам имя и фамилия. И потом просто вернем голову (то есть первый элемент) списка.

With this high-level specification of our approach, we can calculate the type of our function. First open PSCi, and find the types of the `filter` and `head` functions:

Базируюясь на таком дизайне высокого уровня, мы можем вычислить необходимый нам тип желаемой функции. Запустим PSCi и посмотрим на тип функций `filter` и `head`:

```text
$ pulp psci

> import Data.List
> :type filter

forall a. (a -> Boolean) -> List a -> List a

> :type head

forall a. List a -> Maybe a
```

Let's pick apart these two types to understand their meaning.

Давайте разберем эти типы чтобы понять что они означают.

`filter` is a curried function of two arguments. Its first argument is a function, which takes a list element and returns a `Boolean` value as a result. Its second argument is a list of elements, and the return value is another list.

`filter` это каррированая функция о двух аргументах. Первый аргумент это функция, которая принимает список и возвращает значение типа `Boolean`. Второй это список элементов, а возвращаемое значение это опять список.

`head` takes a list as its argument, and returns a type we haven't seen before: `Maybe a`. `Maybe a` represents an optional value of type `a`, and provides a type-safe alternative to using `null` to indicate a missing value in languages like Javascript. We will see it again in more detail in later chapters.

`head` принимает список в качестве первого аргумента и возврашает тип, который мы до сих пор еще не видели: `Maybe a`. Этот тип представляет собой необязательное значение типа `a`, и позволяет реализовать типо-безопасную альтернативу значению `null`, чтобы отразить в программе несуществующее значение, как в JavaScript. Мы его еще увидим в дальнейших главах.

The universally quantified types of `filter` and `head` can be _specialized_ by the PureScript compiler, to the following types:

Универсально квантифицированные типы для `filter` и `head` могут быть _специализированы_ компилятором PureScript в следующие типы:

```haskell
filter :: (Entry -> Boolean) -> AddressBook -> AddressBook

head :: AddressBook -> Maybe Entry
```

We know that we will need to pass the first and last names that we want to search for, as arguments to our function.

Мы знаем что мы должны передать искомые имя и фамилию как аргументы в нашу функцию.

We also know that we will need a function to pass to `filter`. Let's call this function `filterEntry`. `filterEntry` will have type `Entry -> Boolean`. The application `filter filterEntry` will then have type `AddressBook -> AddressBook`. If we pass the result of this function to the `head` function, we get our result of type `Maybe Entry`.

Мы также знаем что нам нужна функция, которую мы передадим в `filter`. Давайте назовем ее `filterEntry`. `filterEntry` будет иметь тип `Entry -> Boolean`. Таким образом `filter filterEntry` будет иметь тип `AddressBook -> AddressBook`. Если мы передадим результат этой функции в  `head`, то мы получим искомый результат `Maybe Entry`.

Putting these facts together, a reasonable type signature for our function, which we will call `findEntry`, is:

Объединяя эти факты, разумная сигнатура нашей функции, котору мы назовем `findEntry`, будет такой:

```haskell
findEntry :: String -> String -> AddressBook -> Maybe Entry
```

This type signature says that `findEntry` takes two strings, the first and last names, and a `AddressBook`, and returns an optional `Entry`. The optional result will contain a value only if the name is found in the address book.

Эта сигнатура говорит, что `findEntry` принимает две строки, имя и фамилию, а также `AddressBook`, и возвращает необязательное значение `Entry`. Необязательное значение будет содержать реальное значение только если имя найдено в адресной книге.

And here is the definition of `findEntry`:

И вот определение `findEntry`:

```haskell
findEntry firstName lastName book = head $ filter filterEntry book
  where
    filterEntry :: Entry -> Boolean
    filterEntry entry = entry.firstName == firstName && entry.lastName == lastName
```

Let's go over this code step by step.

Давайте пройдемся по этому коду шаг за шагом.

`findEntry` brings three names into scope: `firstName`, and `lastName`, both representing strings, and `book`, an `AddressBook`.

`findEntry` вводит в область видимости три идентификатора: `firstName`, `lastName`, представленные строками и `book`, представленный типом `AddressBook`.

The right hand side of the definition combines the `filter` and `head` functions: first, the list of entries is filtered, and the `head` function is applied to the result.

Правая часть определения комбинирует функции `filter` и `head`: сначала фильтруется список записей, а потом к результату применяется `head`.

The predicate function `filterEntry` is defined as an auxiliary declaration inside a `where` clause. This way, the `filterEntry` function is available inside the definition of our function, but not outside it. Also, it can depend on the arguments to the enclosing function, which is essential here because `filterEntry` uses the `firstName` and `lastName` arguments to filter the specified `Entry`.

Функция-предикат `filterEntry` определена как вспомогательное объявление внутри объявления `where`. Таким образом, функция `filterEntry` доступна внутри определения нашей функции, но не снаружи ее области видимости. К тому же, она может зависеть от аргументов внешней функции, что в данном случае для нас существенно, поскольку `filterEntry` использует аргументы `firstName` и `lastName` чтобы отфильтровать необходимое значение `Entry`.

Note that, just like for top-level declarations, it was not necessary to specify a type signature for `filterEntry`. However, doing so is recommended as a form of documentation.

Обратите внимание что, как и в случае с нашими внешними объявлениями, не обязательно указывать сигнатуру типов для `filterEntry`. Однако это считается хорошим тоном для документирования.

## Infix Function Application
## Инфиксное применение функций

In the code for `findEntry` above, we used a different form of function application: the `head` function was applied to the expression `filter filterEntry book` by using the infix `$` symbol.

В предыдущем коде для `findEntry` мы использовали другую форму применения функций к аргументам: функция `head` была применена к выражению `filter filterEntry book` при помощи инфиксного символа `$`.

This is equivalent to the usual application `head (filter filterEntry book)`

Это эквивалентно обычному применению `head (filter filterEntry book)`

`($)` is just an alias for a regular function called `apply`, which is defined in the Prelude. It is defined as follows:

`($)` это просто синоним для обычной функции `apply`, которая определена в модуле Prelude. Определение выглядит следующим образом:

```haskell
apply :: forall a b. (a -> b) -> a -> b
apply f x = f x

infixr 0 apply as $
```

So `apply` takes a function and a value and applies the function to the value. The `infixr` keyword is used to define `($)` as an alias for `apply`.

Таким образом, `apply` берет функцию и значение и применяет функцию к значению. Ключевое слово `infixr` использовано для определения `($)` как синонима `apply`.

But why would we want to use `$` instead of regular function application? The reason is that `$` is a right-associative, low precedence operator. This means that `$` allows us to remove sets of parentheses for deeply-nested applications.

Но почему мы захотели бы использовать `$` вместо обычного применения функции? Причина в том, что `$` это правоассоциативный оператор с низким приоритетом. Что означает что `$` избавляет нас от набора скобок вокруг глубоко вложенных применений функций.

For example, the following nested function application, which finds the street in the address of an employee's boss:

Например, следующая вложенное применение функций, которое находит улицу в адресе руководителя сотрудника:

```haskell
street (address (boss employee))
```

becomes (arguably) easier to read when expressed using `$`:
становится (предположительно) более легкочитаемой после записи с использованием `$`:

```haskell
street $ address $ boss employee
```

## Function Composition
## Композиция функций

Just like we were able to simplify the `insertEntry` function by using eta conversion, we can simplify the definition of `findEntry` by reasoning about its arguments.

Ровно также как мы смогли упростить функцию `insertEntry` используя эта-преобразование, мы можем упростить определение `findEntry` рассуждая о ее аргументах.

Note that the `book` argument is passed to the `filter filterEntry` function, and the result of this application is passed to `head`. In other words, `book` is passed to the _composition_ of the functions `filter filterEntry` and `head`.

Обратите внимание, что аргумент `book` передается в функцию `filter filterEntry` и результат этого применения функции передается в `head`. Другими словами, `book` передается в _композицию_ функций `filter filterEntry` и `head`.

In PureScript, the function composition operators are `<<<` and `>>>`. The first is "backwards composition", and the second is "forwards composition".

В PureScript операторами для композиции функций являются `<<<` и `>>>`. Первый это "обратная композиция", а второй - "прямая композиция".

We can rewrite the right-hand side of `findEntry` using either operator. Using backwards-composition, the right-hand side would be

Мы можем переписать правую часть определения `findEntry` используя любой из этих операторов. Используя обратную композицию, правая часть будет выглядеть так:

```
(head <<< filter filterEntry) book
```

In this form, we can apply the eta conversion trick from earlier, to arrive at the final form of `findEntry`:

В этой форме использования мы можем применить трюк эта-преобразования как и раньше, чтобы придти к окончательной форме `findEntry`:

```haskell
findEntry firstName lastName = head <<< filter filterEntry
  where
    ...
```

An equally valid right-hand side would be:

Не менее корректная другая форма правой части определения:

```haskell
filter filterEntry >>> head
```

Either way, this gives a clear definition of the `findEntry` function: "`findEntry` is the composition of a filtering function and the `head` function".

В любом случае это дает нам ясное определение функции `findEntry`: "`findEntry` это композиция фильтрующей функции и функции `head`".

I will let you make your own decision which definition is easier to understand, but it is often useful to think of functions as building blocks in this way - each function executing a single task, and solutions assembled using function composition.

Я позволю вам самостоятельно принять решение какое определение легче понять, но в любом случае, часто выгодно думать о функциях как о строительных блоках в таком ключе - каждая функция выполняет одну простую задачу, а решение собирается вместе при помощи композиции функций.

## Tests, Tests, Tests ...
## Тесты, тесты, тесты ...

Now that we have the core of a working application, let's try it out using PSCi.

Теперь, когда у нас есть ядро работающего приложения, давайте попробуем его с применением PSCi.

```text
$ pulp psci

> import Data.AddressBook
```

Let's first try looking up an entry in the empty address book (we obviously expect this to return an empty result):

Давайте сначала попробуем поискать запись в пустой адресной книге (очевидно что мы ожидаем получить пустой результат):

```text
> findEntry "John" "Smith" emptyBook

No type class instance was found for

    Data.Show.Show { firstName :: String
                   , lastName :: String
                   , address :: { street :: String
                                , city :: String
                                , state :: String
                                }
                   }
```

An error! Not to worry, this error simply means that PSCi doesn't know how to print a value of type `Entry` as a String.

Ошибка! Не беспокойтесь, эта ошибка просто означает что PSCi не имеет понятия как распечатать значение типа `Entry` в виде строки.

The return type of `findEntry` is `Maybe Entry`, which we can convert to a `String` by hand.

Возвращаемое значение `findEntry` это `Maybe Entry`, которое мы можем преобразовать в `String` вручную.

Our `showEntry` function expects an argument of type `Entry`, but we have a value of type `Maybe Entry`. Remember that this means that the function returns an optional value of type `Entry`. What we need to do is apply the `showEntry` function if the optional value is present, and propagate the missing value if not.

Наша функция `showEntry` ожидает аргумент типа `Entry`, а у нас вместо этого типа `Maybe Entry`. Как вы помните, что это означает что функция возвращает необязательное значение `Entry`. Все что нам надо сделать это применить `showEntry` если необязательное значение присутствует и пробросить отсутствующее значение дальше если нет.

Fortunately, the Prelude module provides a way to do this. The `map` operator can be used to lift a function over an appropriate type constructor like `Maybe` (we'll see more on this function, and others like it, later in the book, when we talk about functors):

К счастью, модуль Prelude предоставляет способ для этого. Оператор `map` может быть использован для того чтобы поднять функцию над подходящим конструктором типа, вроде `Maybe` (мы увидим больше об этой функции и ей аналогичных, дальше в книге, когда будем говорить о функторах):

```text
> import Prelude
> map showEntry (findEntry "John" "Smith" emptyBook)

Nothing
```

That's better - the return value `Nothing` indicates that the optional return value does not contain a value - just as we expected.

Так лучше - возвращаемое значение `Nothing` указывает что необязательное возвращаемое значение не содержит значения, ровно как мы и ожидали.

For ease of use, we can create a function which prints an `Entry` as a String, so that we don't have to use `showEntry` every time:

Для простоты, мы можем создать функцию, которая печатает `Entry` как строку, так чтобы не надо было использовать `showEntry` каждый раз:

```text
> let printEntry firstName lastName book = map showEntry (findEntry firstName lastName book)
```

Now let's create a non-empty address book, and try again. We'll reuse our example entry from earlier:

Теперь давайте создадим непустую адресную книгу и попробуем опять. Повторно используем наш пример из предыдущего раза:

```text
> let book1 = insertEntry entry emptyBook

> printEntry "John" "Smith" book1

Just ("Smith, John: 123 Fake St., Faketown, CA")
```

This time, the result contained the correct value. Try defining an address book `book2` with two names by inserting another name into `book1`, and look up each entry by name.

Теперь результат содержит правильное значение. Попробуйте определить новую адресную книгу `book2` с двумя именами путем вставление нового имени в `book1` и поискать каждую из записей по имени.

X> ## Exercises
X> ## Упражнения
X>
X> 1. (Easy) Test your understanding of the `findEntry` function by writing down the types of each of its major subexpressions. For example, the type of the `head` function as used is specialized to `AddressBook -> Maybe Entry`.

X> 1. (Легкое) Протестируйте свое понимание функции `findEntry` написанием типов всех основных подвыражений. Например тип функции `head` специализируется как `AddressBook -> Maybe Entry`.

X> 1. (Medium) Write a function which looks up an `Entry` given a street address, by reusing the existing code in `findEntry`. Test your function in PSCi.

X> 1. (Среднее) Напишите функцию, которая ищет `Entry` по адресу, повторно использовав существующий код для `findEntry`. Протестируйте свою функцию в PSCi.

X> 1. (Medium) Write a function which tests whether a name appears in a `AddressBook`, returning a Boolean value. _Hint_: Use PSCi to find the type of the `Data.List.null` function, which test whether a list is empty or not.

X> 1. (Среднее) Напишите функцию, которая проверяет наличие имени в `AddressBook` и возвращает значение `Boolean`. _Намёк_: Используйте PSCi чтобы определить тип функции `Data.List.null`, которая проверяет непуст ли список.

X> 1. (Difficult) Write a function `removeDuplicates` which removes duplicate address book entries with the same first and last names. _Hint_: Use PSCi to find the type of the `Data.List.nubBy` function, which removes duplicate elements from a list based on an equality predicate.

X> 1. (Сложное) Напишите функцию `removeDuplicates`, которая удаляет дубликаты записей с одинаковыми именем и фамилией. _Намёк_: Используйте PSCi чтобы определить тип функции `Data.List.nubBy, которая удаляет дубликаты элементов списка, базируясь на заданном предикате равенства.

## Conclusion
## Заключение

In this chapter, we covered several new functional programming concepts:
В этой главе мы обсудили несколько новых концепций функционального программирования:

- How to use the interactive mode PSCi to experiment with functions and test ideas.
- Как использовать интерактивный режим PSCi для экспериментов с функциями и тестирования идей.
- The role of types as both a correctness tool, and an implementation tool.
- Роль типов как инструмента для проверки верности, так и инструмента реализации.
- The use of curried functions to represent functions of multiple arguments.
- Использование каррированных функций для представления функций с несколькими аргументами.
- Creating programs from smaller components by composition.
- Создание программ из меньших компонентов путем композиции.
- Structuring code neatly using `where` expressions.
- Аккуратное структурирование кода при помощи выражений `where`.
- How to avoid null values by using the `Maybe` type.
- Как избегать нулевых значений используя тип `Maybe`.
- Using techniques like eta conversion and function composition to refactor code into a clear specification.
- Использование техник типа эта-преобразования и композиции функций для рефакторинга кода в ясную спецификацию.

In the following chapters, we'll build on these ideas.

В следующих главах мы продолжим строить на этом фундаменте.

### Примечания переводчика
#### 1
Record можно перевести как "запись" или "структура". Я выбрал "запись" согласно [статье в в Wikipedia](https://ru.wikipedia.org/wiki/%D0%97%D0%B0%D0%BF%D0%B8%D1%81%D1%8C_(%D1%82%D0%B8%D0%BF_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)) 
#### 2
Необязательные значения широко распространены в языках с развитыми системами типов. Это значения, которых "может не быть". В более простых языках это эквивалентно "невозможным" значениям, например отрицательное число для значений, которые могут быть только положительными, или широко распространенный NULL для строк. Такие условности служат постоянным источником ошибок, и в PureScript (как и в Haskell, OCaml, F# и многих других) такой подход считается плохой практикой и настоятельно рекомендуются "необязательные типы". В таких типах есть специальное значение, обозначающее "ничто" и его невозможно ни с чем спутать и случайно использовать в вычислениях с "основным" типом.
#### 3
[Квантор всеобщности](https://ru.wikipedia.org/wiki/%D0%9A%D0%B2%D0%B0%D0%BD%D1%82%D0%BE%D1%80_%D0%B2%D1%81%D0%B5%D0%BE%D0%B1%D1%89%D0%BD%D0%BE%D1%81%D1%82%D0%B8). В PureScript вы можете использовать юникод символ FOR ALL (U+2200 или &amp#8704;) вместо ключевого слова `forall`.
#### 4
[η-преобразование](https://ru.wikipedia.org/wiki/%D0%9B%D1%8F%D0%BC%D0%B1%D0%B4%D0%B0-%D0%B8%D1%81%D1%87%D0%B8%D1%81%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5#.CE.B7-.D0.BF.D1.80.D0.B5.D0.BE.D0.B1.D1.80.D0.B0.D0.B7.D0.BE.D0.B2.D0.B0.D0.BD.D0.B8.D0.B5)
#### 5
В данном случае "точка" не имеет никакого отношения к синтаксису языка программирования, этот термин пришел из топологии, где множества состоят из точек и соответственно точки аналогичны значениям в алгебре. Бесточечная запись, тем самым, это запись без указания значений, то есть аргументов функций.
