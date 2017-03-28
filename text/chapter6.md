# Классы типов

## Цели главы

This chapter will introduce a powerful form of abstraction which is enabled by PureScript's type system - type classes.

Эта глава познакомит с мощной формой абстракции, которую предоставляет система типов PureScript - классы типов.

This motivating example for this chapter will be a library for hashing data structures. We will see how the machinery of type classes allow us to hash complex data structures without having to think directly about the structure of the data itself.

Мотивирующим примером к данной главе будет библиотека для хэширования структур данных. Мы увидем как механизм классов типов позволит нам хэшировать сложные структуры данных без необходимости непосредственно думать о структуре этих данных.

We will also see a collection of standard type classes from PureScript's Prelude and standard libraries. PureScript code leans heavily on the power of type classes to express ideas concisely, so it will be beneficial to familiarize yourself with these classes.

Мы также увидем коллекцию стандартных классов типов из Prelude PureScript и стандартных библиотек. PureScript код сильно опирается на мощь классов типов, чтобы выразить идеи сжато, поэтому вам будет полезно ознакомиться с этими классами.

## Project Setup
## Настройка проекта

The source code for this chapter is defined in the file `src/Data/Hashable.purs`. 

Исходный код к данной главе содержится в файле `src/Data/Hashable.purs`.

The project has the following Bower dependencies:

Проект имеет следующие зависимости Bower:

- `purescript-maybe`, which defines the `Maybe` data type, which represents optional values.
- `purescript-tuples`, which defines the `Tuple` data type, which represents pairs of values.
- `purescript-either`, which defines the `Either` data type, which represents disjoint unions.
- `purescript-strings`, which defines functions which operate on strings.
- `purescript-functions`, which defines some helper functions for defining PureScript functions.

- `purescript-maybe`, содержащий тип `Maybe`, означающий опциональные значение.
- `purescript-tuples`, определяющий тип `Tuple` - пары значений.
- `purescript-either`, тип `Either`, представляющий непересекающиеся множества.
- `purescript-strings`, содаржащий функции для операции над строками.
- `purescript-functions`, содержащий некоторые вспомогательные функции для определения PureScript функций.

The module `Data.Hashable` imports several modules provided by these Bower packages.

Модуль `Data.Hashable` импортирует несколько модулей, предоставляемых этими пакетами Bower.

## Show Me!
## Покажи мне!

Our first simple example of a type class is provided by a function we've seen several times already: the `show` function, which takes a value and displays it as a string.

Наш первый простой пример класса типов предоставлен функцией, которую мы уже видели несколько раз: эта функция `show`, которая принимает некоторое значение и выводит его в виде строки.

`show` is defined by a type class in the `Prelude` module called `Show`, which is defined as follows:

Функция `show` определена классом типов `Show` в модуле `Prelude`. Определение выглядит следующим образом:

```haskell
class Show a where
  show :: a -> String
```

This code declares a new _type class_ called `Show`, which is parameterized by the type variable `a`.

Этот код объявляет новый _класс типов_ под названием `Show`, который параметаризован переменной типа `a`.

A type class _instance_ contains implementations of the functions defined in a type class, specialized to a particular type.

_Экземпляр_ класса типов содержит реализацию функций, определённых в классе типов, специализированные под определённый тип

For example, here is the definition of the `Show` type class instance for `Boolean` values, taken from the Prelude:

Например, ниже определение экземпляра `Show` для значений типа `Boolean`, взятое из Prelude:

```haskell
instance showBoolean :: Show Boolean where
  show true = "true"
  show false = "false"
```

This code declares a type class instance called `showBoolean` - in PureScript, type class instances are named to aid the readability of the generated JavaScript. We say that the `Boolean` type _belongs to the `Show` type class_.

Этот код объявляет экземпляр класса типов в PureScript под названием `showBoolean`. Экземпляры классов типов имеют собственные названия с целью повышения читаемости сгенерированного кода на JavaScript. Мы говорим, что тип `Boolean` _принадлежит к классу типов `Show`_.

We can try out the `Show` type class in PSCi, by showing a few values with different types:

Мы можем проверить класс типов `Show` в PSCi, показав несколько значений различных типов:

```text
> import Prelude

> show true
"true"

> show 1.0
"1.0"

> show "Hello World"
"\"Hello World\""
```

These examples demonstrate how to `show` values of various primitive types, but we can also `show` values with more complicated types:

Эти примеры демонстрирует значения `show` с различными примитивными типами, но мы также можем использовать `show` с более сложными типами:

```text
> import Data.Tuple

> show (Tuple 1 true)
"(Tuple 1 true)"

> import Data.Maybe

> show (Just "testing")
"(Just \"testing\")"
```

If we try to show a value of type `Data.Either`, we get an interesting error message:

Если мы попробуем использовать `show` на типе `Data.Either`, то мы получим интересное сообщение об ошибке:

```text
> import Data.Either
> show (Left 10)

The inferred type

    forall a. Show a => String

has type variables which are not mentioned in the body of the type. Consider adding a type annotation.
```

The problem here is not that there is no `Show` instance for the type we intended to `show`, but rather that PSCi was unable to infer the type. This is indicated by the _unknown type_ `a` in the inferred type.

Проблема здесь не в том, что нет экземпляра `Show` для типа, который мы намеревались показать, скорее в том, что PSCi не смог вывести этот тип. Это показано _неизвестным типом_ `a` в выведенном типе.

We can annotate the expression with a type, using the `::` operator, so that PSCi can choose the correct type class instance:

Мы можем аннотировать выражение типом, используя оператор `::`, и тогда PSCi сможет выбрать правильный экземпляр класса типов:

```text
> show (Left 10 :: Either Int String)
"(Left 10)"
```

Some types do not have a `Show` instance defined at all. One example of this is the function type `->`. If we try to `show` a function from `Int` to `Int`, we get an appropriate error message from the type checker:

Некоторые типы не имеют экземпляров класса типов `Show`. Один из примеров таких типов это тип функции `->`. Если мы попытаемся применить `show` к функции из `Int` в `Int`, то получим соответствующее сообщение об ошибке от проверщика типов:

```text
> import Prelude
> show $ \n -> n + 1

No type class instance was found for

  Data.Show.Show (Int -> Int)
```

X> ## Упражнения
X>
X> 1. (Easy) Use the `showShape` function from the previous chapter to define a `Show` instance for the `Shape` type.

X> 1. (Лёгкое) Используя функцию `showShape` из предыдущей главы определите экземпляр `Show` для типа `Shape`.

## Common Type Classes
## Общие классы типов

In this section, we'll look at some standard type classes defined in the Prelude and standard libraries. These type classes form the basis of many common patterns of abstraction in idiomatic PureScript code, so a basic understanding of their functions is highly recommended.

В этом разделе мы посмотрим на некоторые стандартные классы типов, определённые в Prelude и стандартных библиотеках. Эти классы типов образуют базис для многих общих моделей абстракций идиоматического кода PureScript, поэтому базовое понимание их функций настоятельно рекомендуется.

### Eq

The `Eq` type class defines the `eq` function, which tests two values for equality. The `==` operator is actually just an alias for `eq`.

Класс типов `Eq` определяет функцию `eq`, которая проверяет два значение на равенство. Оператор `==` на самом деле является псеводнимом для `eq`.

```haskell
class Eq a where
  eq :: a -> a -> Boolean
```

Note that in either case, the two arguments must have the same type: it does not make sense to compare two values of different types for equality.

Отметим, что в любых случаях два аргумента должны иметь один и тот же тип: не имеет смысла проверять два значения различных типов на равенство.

Try out the `Eq` type class in PSCi:

Попробуем класс типов `Eq` в PSCi:

```text
> 1 == 2
false

> "Test" == "Test"
true
```

### Ord

The `Ord` type class defines the `compare` function, which can be used to compare two values, for types which support ordering. The comparison operators `<` and `>` along with their non-strict companions `<=` and `>=`, can be defined in terms of `compare`.

Класс типов `Ord` определяет функцию `compare`, которая может быть использована для сравнения двух значений, типы которых поддерживают упорядычевание.  Операторы сравнения `<` и `>` вместе с их не строгими компаньонами `<=` и `>=` могут быть определены в терминах `compare`.

```haskell
data Ordering = LT | EQ | GT

class Eq a <= Ord a where
  compare :: a -> a -> Ordering
```

The `compare` function compares two values, and returns an `Ordering`, which has three alternatives:

Функция `compare` сравнивает два значения и возвращает `Ordering`, которое имеет три варианта:

- `LT` - if the first argument is less than the second.
- `EQ` - if the first argument is equal to the second.
- `GT` - if the first argument is greater than the second.

- `LT` - если первый аргумент меньше второго.
- `EQ` - если первый аргумент равен второму.
- `GT` - если первый аргумент больше второго.

Again, we can try out the `compare` function in PSCi:

И опять, мы можем проверить функцию `compare` в PSCi:

```text
> compare 1 2
LT

> compare "A" "Z"
LT
```

### Field

The `Field` type class identifies those types which support numeric operators such as addition, subtraction, multiplication and division. It is provided to abstract over those operators, so that they can be reused where appropriate.

Класс типов `Field` идентифицирует те типы, которые поддерживают числовые операции, такие как сложение, вычитание, умножение и деление. Он предоставлен для абстрагирования над этими операторами, поэтому они могут быть переиспользованы в соответствующих случаях.

_Note_: Just like the `Eq` and `Ord` type classes, the `Field` type class has special support in the PureScript compiler, so that simple expressions such as `1 + 2 * 3` get translated into simple JavaScript, as opposed to function calls which dispatch based on a type class implementation.

_Примечание_: Так же как классы `Eq` и `Ord`, класс типов `Field` имеет специальную поддержку компилятора PureScript, поэтому простые выражения, такие как `1 + 2 * 3` транслируются в простой JavaScript, в противоположность вызовам функций, которые диспетчеризуются в соответствие с реализацией класса типов.

```haskell
class EuclideanRing a <= Field a
```

The `Field` type class is composed from several more general _superclasses_. This allows us to talk abstractly about types which support some but not all of the `Field` operations. For example, a type of natural numbers would be closed under addition and multiplication, but not necessarily under subtraction, so that type might have an instance of the `Semiring` class (which is a superclass of `Num`), but not an instance of `Ring` or `Field`.

Класс типов `Field` составлен из нескольких более общих _суперклассов_. Это позволяет нам говорить более абстрактно о типах, которые поддерживают некоторые, но не все операции `Field`. ....

Superclasses will be explained later in this chapter, but the full numeric type class hierarchy is beyond the scope of this chapter. The interested reader is encouraged to read the documentation for the superclasses of `Field` in `purescript-prelude`.

Суперклассы будут объяснены далее в главе, но объяснение полной числовой иерархии классов типов выходит за рамки данной главы. Заинтересованным читателям рекомендуется ознакомиться с документацией суперкласса `Field` в `purescript-prelude`.

### Semigroups and Monoids

### Semigroup и Monoid

The `Semigroup` type class identifies those types which support an `append` operation to combine two values:

Класс типов `Semigroup` (Полугруппа) определяет те типы, которые поддерживают операцию `append` для объединения двух значений:


```haskell
class Semigroup a where
  append :: a -> a -> a
```

Strings form a semigroup under regular string concatenation, and so do arrays. Several other standard instances are provided by the `purescript-monoid` package.

Строки образуют полугруппу посредством обычной строковой конкатенации, как и массивы. Несколько других стандартных экземпляров предоставляются пакетом `purescript-monoid`.

The `<>` concatenation operator, which we have already seen, is provided as an alias for `append`.

Оператор конкатенации `<>`, который мы уже видели, является псевдонимом для `append`.

The `Monoid` type class (provided by the `purescript-monoid` package) extends the `Semigroup` type class with the concept of an empty value, called `mempty`:

Класс типов `Monoid` (предоставленный пакетом `purescript-monoid`) расширяет класс типов `Semigroup` концепцией пустого значения, под названием `mempty`:

```haskell
class Semigroup m <= Monoid m where
  mempty :: m
```

Again, strings and arrays are simple examples of monoids.

И снова, строки и массивы являются простыми примерами моноидов.

A `Monoid` type class instance for a type describes how to _accumulate_ a result with that type, by starting with an "empty" value, and combining new results. For example, we can write a function which concatenates an array of values in some monoid by using a fold. In PSCi:

Экземпляр класса типов `Monoid` для некоторого типа описывает как _аккумулировать_ результат для данного типа, начиная с "пустого" значения, комбинируя новые результаты. Например, мы можем написать функция, которая конкатенирует массив значений в некотором моноиде, используя fold. В PSCi:

```haskell
> import Data.Monoid
> import Data.Foldable

> foldl append mempty ["Hello", " ", "World"]  
"Hello World"

> foldl append mempty [[1, 2, 3], [4, 5], [6]]
[1,2,3,4,5,6]
```

The `purescript-monoid` package provides many examples of monoids and semigroups, which we will use in the rest of the book.

Пакет `purescript-monoid` предоставляет множество примеров моноидов и полугрупп, которые мы будем использовать в оставшейся части книги.

### Foldable

If the `Monoid` type class identifies those types which act as the result of a fold, then the `Foldable` type class identifies those type constructors which can be used as the source of a fold.

Если класс типов `Monoid` определяет типы, которые выступают(?) как результат свёртки, тогда класс типов `Foldable` определяет те конструкторы типов, которые могут быть использованы в качестве источника свёртки.

The `Foldable` type class is provided in the `purescript-foldable-traversable` package, which also contains instances for some standard containers such as arrays and `Maybe`.

Класс типов `Foldable` предоставлен пакетом `purescript-foldable-traversable`, который так же содержит экземпляры для некоторых стандартных контейнеров, таких как массивы и тип `Maybe`.

The type signatures for the functions belonging to the `Foldable` class are a little more complicated than the ones we've seen so far:

Сигнатуры типа для функций, относящихся к классу `Foldable`, немного сложнее, чем те, что мы видели до сих пор:

```haskell
class Foldable f where
  foldr :: forall a b. (a -> b -> b) -> b -> f a -> b
  foldl :: forall a b. (b -> a -> b) -> b -> f a -> b
  foldMap :: forall a m. Monoid m => (a -> m) -> f a -> m
```

It is instructive to specialize to the case where `f` is the array type constructor. In this case, we can replace `f a` with `Array a` for any a, and we notice that the types of `foldl` and `foldr` become the types that we saw when we first encountered folds over arrays.

Будет познавательно специализироваться на примере, когда `f` является конструктором массива. В этом случае мы можем заменить `f a` на `Array a` для любого a. Заметим, что типы функций `foldl` и `foldr` стали теми, что мы увидели, когда впервые столкулись со свёртками над массивами.

What about `foldMap`? Well, that becomes `forall a m. Monoid m => (a -> m) -> Array a -> m`. This type signature says that we can choose any type `m` for our result type, as long as that type is an instance of the `Monoid` type class. If we can provide a function which turns our array elements into values in that monoid, then we can accumulate over our array using the structure of the monoid, and return a single value.

Но что насчёт `foldMap`? Что ж, он превратиться в `forall a m. Monoid m => (a -> m) -> Array a -> m`. Эта сигнатура типа говорит, что мы можем выбрать любой тип `m` для результата, при условии, что выбранный тип является экземпляром класса типов `Monoid`. Если мы предоставим функцию, которая превращает массив элементов в значения в данном моноиде, тогда мы можем производить аккумулирование по нашему массиву, используя структуру моноида, а затем возвращать единственное значение.

Давайте попробуем `foldMap` в PSCi:

```text
> import Data.Foldable

> foldMap show [1, 2, 3, 4, 5]
"12345"
```

Here, we choose the monoid for strings, which concatenates strings together, and the `show` function which renders an `Int` as a `String`. Then, passing in an array of integers, we see that the results of `show`ing each integer have been concatenated into a single `String`.

Здесь мы выбрали моноид для строк,  который соединяет строки вместе, а также функцию `show`, которая отображает `Int` в `String`. Затем, передавая массив чисел, мы видем, что результат отображения каждого числа конкатенируется в единый `String`.

But arrays are not the only types which are foldable. `purescript-foldable-traversable` also defines `Foldable` instances for types like `Maybe` and `Tuple`, and other libraries like `purescript-lists` define `Foldable` instances for their own data types. `Foldable` captures the notion of an _ordered container_.

Но массивы не единственные свёртываемые (foldable) типы. `purescript-foldable-traversable` также определяет `Foldable` инстантсы для таких типов, как `Maybe` и `Tuple`, а другие библиотеки, например `purescript-lists`, определяют экземпляры `Foldable` для своих собственных типов. `Foldable` отражает понятие _упорядоченного контейнера_.

### Functor, and Type Class Laws
## Functor и законы классов типов

The Prelude also defines a collection of type classes which enable a functional style of programming with side-effects in PureScript: `Functor`, `Applicative` and `Monad`. We will cover these abstractions later in the book, but for now, let's look at the definition of the `Functor` type class, which we have seen already in the form of the `map` function:

Prelude определяет также коллекцию классов типов, которые дают возможность программирования в функциональном стиле с побочными эффектами в PureScript: `Functor`, `Applicative` и `Monad`. Мы рассмотрим данные абстракции далее в книге, но сейчас, давайте посмотрим на определения класса типов `Functor`, который мы уже видели в форме функции `map`:

```haskell
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b
```

The `map` function (and its alias `<$>`) allows a function to be "lifted" over a data structure. The precise definition of the word "lifted" here depends on the data structure in question, but we have already seen its behavior for some simple types:

Функция `map` (а также её псевдоним `<$>`) позволяет "поднять" функцию над структурой данных. Точное определение слова "поднятый" зависит от структуры данных, но мы уже видели это поведение для некоторых простых типов:

```text
> import Prelude

> map (\n -> n < 3) [1, 2, 3, 4, 5]
[true, true, false, false, false]

> import Data.Maybe
> import Data.String (length)

> map length (Just "testing")
(Just 7)
```

How can we understand the meaning of the `map` function, when it acts on many different structures, each in a different way?

Как мы можем понять смысл функции `map`, когда она взаимодействует со множеством различных структур, и с каждой - по-своему?

Well, we can build an intuition that the `map` function applies the function it is given to each element of a container, and builds a new container from the results, with the same shape as the original. But how do we make this concept precise?

Ну, мы можем  построить интуицию, что функция `map` применяет функцию к каждому элементу некоторого контейнера и создаёт новый контейнер с полученными результатами, и с той же формой контейнера, что и оригинал. Но как мы можем сделать этот концепт более точным?

Type class instances for `Functor` are expected to adhere to a set of _laws_, called the _functor laws_:

Ожидается, что экземпляр класса типов `Functor` придерживается некоторого набора _законов_, под названием _законы функтора_:

- `map id xs = xs`
- `map g (map f xs) = map (g <<< f) xs`

The first law is the _identity law_. It states that lifting the identity function (the function which returns its argument unchanged) over a structure just returns the original structure. This makes sense since the identity function does not modify its input.

Первый закон - это _закон идентичности_. Он утверждает, что "поднятие" функции-идентификатора (это функция, которая возвращает свой аргумент без изменений) над структурой данных просто возвращает оригиналную структуру. Это имеет смысл, поскольку функция-идентификатор не изменяет свои входные данные.

The second law is the _composition law_. It states that mapping one function over a structure, and then mapping a second, is the same thing as mapping the composition of the two functions over the structure.

Второй закон - это _закон композиции_. Он утверждает, что проход одной функции над структурой, а затем проход второй функции, будет означать тоже самое, что проход композиции двух функций над данной структурой.

Whatever "lifting" means in the general sense, it should be true that any reasonable definition of lifting a function over a data structure should obey these rules.

Чтобы не значило слово "поднятие" в общем смысле, должно быть верно то, что любое разумное определение поднятия функции над структурой данных должно подчиняться этим правилам.

Many standard type classes come with their own set of similar laws. The laws given to a type class give structure to the functions of that type class and allow us to study its instances in generality. The interested reader can research the laws ascribed to the standard type classes that we have seen already.

Многие стандартные классы типов поставляются с собственным набором похожих законов. Законы, установленные для класса типов, предоставляют структуру для функций данного класса типов и позволяют нам изучать его экземпляры в общности. Заинтересованный читатель может исследовать законы, приписываемые стандартным классам типов, которые мы уже рассматривали.

X> ## Упражнения
X>
X> 1. (Лёгкое) Следующий тип представляет из себя комплексное число:
X>
X>     ```haskell
X>     newtype Complex = Complex
X>       { real :: Number
X>       , imaginary :: Number
X>       }
X>     ```
X>       
X>     Определите экземпляры `Show` и `Eq` для `Complex`.

## Type Annotations
## Аннотации к типам

Types of functions can be constrained by using type classes. Here is an example: suppose we want to write a function which tests if three values are equal, by using equality defined using an `Eq` type class instance.

На типы функций можно накладывать ограничения при помощи классов типов. Приводим пример: предположим, мы хотим написать функцию, которая проверяет три значение на равенство, используя равенство, определённое с помощью экземпляра класса типов `Eq`.

```haskell
threeAreEqual :: forall a. Eq a => a -> a -> a -> Boolean
threeAreEqual a1 a2 a3 = a1 == a2 && a2 == a3
```

The type declaration looks like an ordinary polymorphic type defined using `forall`. However, there is a type class constraint in parentheses, separated from the rest of the type by a double arrow `=>`.

Объявление типа выглядит как обычный полиморфный тип, определённый при помощи `forall`. Однако, здесь есть ограничение класса типов, отделённое от остальной части определения двойной стрелкой `=>`.

This type says that we can call `threeAreEqual` with any choice of type `a`, as long as there is an `Eq` instance available for `a` in one of the imported modules.

Тип указывает, что мы можем вызывать `threeAreEqual` с любым типом `a`, пока существует экземпляр `Eq` для `a` в одном из импортируемых модулей.

Constrained types can contain several type class instances, and the types of the instances are not restricted to simple type variables. Here is another example which uses `Ord` and `Show` instances to compare two values:

Ограниченные типы могут содержать несколько экземпляров классов типов, и типы экземпляров не ограничены простыми переменнами типа. Вот еще один пример, в котором использованы экземпляры `Ord` и `Show` для сравнения двух значений:

```haskell
showCompare :: forall a. (Ord a, Show a) => a -> a -> String
showCompare a1 a2 | a1 < a2 =
  show a1 <> " is less than " <> show a2
showCompare a1 a2 | a1 > a2 =
  show a1 <> " is greater than " <> show a2
showCompare a1 a2 =
  show a1 <> " is equal to " <> show a2
```

The PureScript compiler will try to infer constrained types when a type annotation is not provided. This can be useful if we want to use the most general type possible for a function.

Компилятор PureScript будет пытаться вывести ограничения типов когда аннотации не предоставлены. Это может быть полезно, если мы хотим использовать наиболее общий возможный тип для функции.

To see this, try using one of the standard type classes like `Semiring` in PSCi:

Чтобы это увидеть, попробуем использовать в PSCi один из стандартных классов типов, например `Semiring`:

```text
> import Prelude

> :type \x -> x + x
forall a. Semiring a => a -> a
```

Here, we might have annotated this function as `Int -> Int`, or `Number -> Number`, but PSCi shows us that the most general type works for any `Semiring`, allowing us to use our function with both `Int`s and `Number`s.

Здесь мы могли бы аннотировать функцию как `Int -> Int`  или `Number -> Number`, но PSCi показал нам наиболее обобщенный тип, работающий с любым `Semiring`, позволяющий нам использовать нашу функцию как с `Int`, так и с `Number`.

## Overlapping Instances
## Перекрывающиеся экземпляры

PureScript has another rule regarding type class instances, called the _overlapping instances rule_. Whenever a type class instance is required at a function call site, PureScript will use the information inferred by the type checker to choose the correct instance. At that time, there should be exactly one appropriate instance for that type. If there are multiple valid instances, the compiler will issue a warning.

PureScript имеет еще одно правило, относящееся к экземплярам классов типов, под названием _правило перекрывающихся экземпляров (overlapping instances rule)_. Всякий раз, когда требуется экземпляр класса типов для вызова функции, PureScript будет использовать информацию, выведенную тайпчекером для выбора правильного экземпляра. В это время должен быть только один соответствующий экземпляр для данного типа. Если имеется несколько допустимых экземпляров, компилятор выдаст предупреждение.

To demonstrate this, we can try creating two conflicting type class instances for an example type. In the following code, we create two overlapping `Show` instances for the type `T`:

Чтобы продемонстрировать это, мы можем попытаться создать два конфликтующего экземпляров класса типов для типа-примера. В следующем коде мы создаем два перекрывающихся экземпляра `Show` для типа` T`:  

```haskell
module Overlapped where

import Prelude

data T = T

instance showT1 :: Show T where
  show _ = "Instance 1"

instance showT2 :: Show T where
  show _ = "Instance 2"
```

This module will compile with no warnings. However, if we _use_ `show` at type `T` (requiring the compiler to to find a `Show` instance), the overlapping instances rule will be enforced, resulting in a warning:

Этот модуль будет компилироваться без каких-либо предупреждений. Однако, если мы _используем_ `show` в типе` T` (требуя, чтобы компилятор мог найти экземпляр `Show`), будет применяться правило перекрывающихся экземпляров, приводящее к предупреждению:

```text
Overlapping instances found for Prelude.Show T
```

The overlapping instances rule is enforced so that automatic selection of type class instances is a predictable process. If we allowed two type class instances for a type to exist, then either could be chosen depending on the order of module imports, and that could lead to unpredictable behavior of the program at runtime, which is undesirable.

Правило перекрывающихся экземпляров применяется принудительно, чтобы автоматичекий выбор экземпляра класса типов был предсказуемым процессом. Если для некоторого типы мы допустим существование двух экземпляров класса типов, то любой из них может быть выбран в зависимости от порядка импорта модулей, что может привести к непредсказуемому поведению программы в процессе выполнения, что недопустипо.

If it is truly the case that there are two valid type class instances for a type, satisfying the appropriate laws, then a common approach is to define newtypes which wrap the existing type. Since different newtypes are allowed to have different type class instances under the overlapping instances rule, there is no longer an issue. This approach is taken in PureScript's standard libraries, for example in `purescript-monoids`, where the `Maybe a` type has multiple valid instances for the `Monoid` type class.

В случае, если действительно есть два валидных экземпляра класса типов для некоторого типа, удовлетворяющие соответствующим законам, то общепринятым подходом является определение newtype типов, оборачивающие существующий тип. Посколько различным newtype типам разрешано иметь разные экземпляры классов типов в рамках правила перекрывающихся экземпляров, то проблемы больше не возникает. Этот подход используется в стандартных библиотеках PureScript, например в `purescript-monoids`, где тип `Maybe a` имеет множество валидных экземпляров для класса типов `Monoid`.

## Instance Dependencies
## Зависимости экземпляров

Just as the implementation of functions can depend on type class instances using constrained types, so can the implementation of type class instances depend on other type class instances. This provides a powerful form of program inference, in which the implementation of a program can be inferred using its types.

Так же как и реализация функций может зависеть от эклемпляров класса типов, используя ограниченные типы (constrained types), так и реализация экземпляров класса типов может зависеть от другого экземпляра класса типов. Это обеспечивает мощную форму вывода программы, в которой реализация самой программы может быть выведена, используя её типы.

For example, consider the `Show` type class. We can write a type class instance to `show` arrays of elements, as long as we have a way to `show` the elements themselves:

Например, рассмотрим класс типов `Show`. Мы можем написать экземпляр класса типов для показа (`show`) массива элементов, при условии, что у нас есть способ показа самих элементов:

```haskell
instance showArray :: Show a => Show (Array a) where
  ...
```

This type class instance is provided in the `purescript-prelude` library.

Данный экземпляр класса типов предоставлен библиотекой `purescript-prelude`.

When the program is compiled, the correct type class instance for `Show` is chosen based on the inferred type of the argument to `show`. The selected instance might depend on many such instance relationships, but this complexity is not exposed to the developer.

Когда программа скомпилированна, правильный экземпляр класса типов `Show` выбирается исходя из вывода типа аргумента `show`. Выбранный экземпляр может зависеть от множества таких отношений, но эта сложность не раскрывается разработчику.

X> ## Упражнения
X>
X> 1. (Лёгкое) Следующее объявление определяет тип непустых массивов элементов некоторого типа `a`::
X>
X>     ```haskell
X>     data NonEmpty a = NonEmpty a (Array a)
X>     ```
X>      
X>     Напишите экземпляр `Eq` для типа `NonEmpty a`, который переиспользует экземпляры для `Eq a` и `Eq (Array a)`.
X> 1. (Среднее) Напишите экземпляр `Semigroup` для `NonEmpty a` путём переиспользования экземпляра `Semigroup` для `Array`.
X> 1. (Среднее) Напишите экземпляр `Functor` для `NonEmpty`.
X> 1. (Среднее) Для любого типа `a` с экземпляром `Ord`, мы можем добавить значение "бесконечности", которое больше любого другого значения:
X>
X>     ```haskell
X>     data Extended a = Finite a | Infinite
X>     ```
X>         
X>     Напишите экземпляр `Ord` для `Extended a`, который переиспользует экземпляр `Ord` для `a`.
X> 1. (Сложное) Напишите экземпляр `Foldable` для `NonEmpty`. _Подсказка_: используйте экземпляр `Foldable` для массивов.
X> 1. (Сложное) Для данного конструктора типа `f`, который определяет упорядоченный контейнер (а также имеет экземпляр `Foldable`), мы можем создать новый контейнер типа, который содержит дополнительный элемент впереди::
X>
X>     ```haskell
X>     data OneMore f a = OneMore a (f a)
X>     ```
X>         
X>     Контейнер `OneMore f` так же является упорядоченным, где новый элемент идёт перед любым элементом `f`. Напишите экземпляр `Foldable` для `OneMore f`:
X>   
X>     ```haskell
X>     instance foldableOneMore :: Foldable f => Foldable (OneMore f) where
X>       ...
X>     ```

## Multi Parameter Type Classes
## Многопараметаризованные классы типов

It's not the case that a type class can only take a single type as an argument. This is the most common case, but in fact, a type class can be parameterized by _zero or more_ type arguments.

Это не правда, что класс типов может принимать только один тип в качестве аргумента. Это наиболее общий случай. Но на самом деле, класс типов может быть параметаризован _от нуля и более_ аргументами типов.

Let's see an example of a type class with two type arguments.

Давайте посмотрим на пример класса типов с двумя аргументами типа.

```haskell
module Stream where

import Data.Array as Array
import Data.Maybe (Maybe)
import Data.String as String

class Stream stream element where
  uncons :: stream -> Maybe { head :: element, tail :: stream }

instance streamArray :: Stream (Array a) a where
  uncons = Array.uncons

instance streamString :: Stream String Char where
  uncons = String.uncons
```

The `Stream` module defines a class `Stream` which identifies types which look like streams of elements, where elements can be pulled from the front of the stream using the `uncons` function.

Модуль `Stream` определяет класс `Stream`, который идентифицирует типы, похожие на потоки элементов, где элементы могут быть извлечены из передней части потока, используя функцию `uncons`.

Note that the `Stream` type class is parameterized not only by the type of the stream itself, but also by its elements. This allows us to define type class instances for the same stream type but different element types.

Отметим, что класс типов `Stream` параметаризован не только типом самого потока, но так же и его элементами. Это позволяет нам определять экземпляры классов типов для одного и того же типа потока, но различных типов элементов.

The module defines two type class instances: an instance for arrays, where `uncons` removes the head element of the array using pattern matching, and an instance for String, which removes the first character from a String.

Модуль определяет два экземпляра класса типов: экземпляр для массивов, где `uncons` удаляет головной элемент массива, используя сопоставление с образцом, а также экземпляр для строк, который удаляет первый символ из строки.

We can write functions which work over arbitrary streams. For example, here is a function which accumulates a result in some `Monoid` based on the elements of a stream:

Мы можем написать функции, которые работают с произвольными потоками. Например, ниже представлена функция, которая аккумулирует результат в некоторый _монойд_ на основе элементах потока:

```haskell
import Prelude
import Data.Monoid (class Monoid, mempty)

foldStream :: forall l e m. (Stream l e, Monoid m) => (e -> m) -> l -> m
foldStream f list =
  case uncons list of
    Nothing -> mempty
    Just cons -> f cons.head <> foldStream f cons.tail
```

Try using `foldStream` in PSCi for different types of `Stream` and different types of `Monoid`.

Попробуйте `foldStream` в PSCi для различных типов `Stream` и различных типов `Monoid`.

## Functional Dependencies
## Функциональные зависимости

Multi-parameter type classes can be very useful, but can easily lead to confusing types and even issues with type inference. As a simple example, consider writing a generic `tail` function on streams using the `Stream` class given above:

Многопараметаризованные классы типов могут быть очень полезны, однако могут легко привести к путанице типов, и даже к пробемам с выводом типов. В качестве простого примера, рассмотрим написание обобщенной функции `tail` на потоках, используя класс типов `Stream`, данный выше:

```haskell
genericTail xs = map _.tail (uncons xs)
```

This gives a somewhat confusing error message:

Это выдаёт несколько запутанное сообщение об ошибке:

```text
The inferred type

  forall stream a. Stream stream a => stream -> Maybe stream

has type variables which are not mentioned in the body of the type. Consider adding a type annotation.
```

The problem is that the `genericTail` function does not use the `element` type mentioned in the definition of the `Stream` type class, so that type is left unsolved.

Проблема в том, что функция `genericTail` не использует тип `element`, упомянутый в определении класса типов `Stream`, и поэтому её тип остаётся не разрешенный.

Worse still, we cannot even use `genericTail` by applying it to a specific type of stream:

Хуже того, мы даже не можем использовать `genericTail`, применив её к специализированному типу потока:

```text
> map _.tail (uncons "testing")

The inferred type

  forall a. Stream String a => Maybe String

has type variables which are not mentioned in the body of the type. Consider adding a type annotation.
```

Here, we might expect the compiler to choose the `streamString` instance. After all, a `String` is a stream of `Char`s, and cannot be a stream of any other type of elements.

Тут мы могли бы ожидать, что компилятор выберет экземпляр `streamString`. В конце концов, строка это поток симполов (`String` is a stream of `Char`s), и он не может быть потоком другого типа элементов.

The compiler is unable to make that deduction automatically, and cannot commit to the `streamString` instance. However, we can help the compiler by adding a hint to the type class definition:

Компилятор не может сделать такой вывод автоматически, и не может определить экземпляр `streamString`. Однако, мы можем помочь компилятору, добавив подсказку в определение класса типов:


```haskell
class Stream stream element | stream -> element where
  uncons :: stream -> Maybe { head :: element, tail :: stream }
```

Here, `stream -> element` is called a _functional dependency_. A functional dependency asserts a functional relationship between the type arguments of a multi-parameter type class. This functional dependency tells the compiler that there is a function from stream types to (unique) element types, so if the compiler knows the stream type, then it can commit to the element type.

Конструкция `stream -> element` здесь называется _функциональной зависимостью_. Функциональная зависимость утверждает функциональные взаимотношения между типами-аргументами в многопараметаризированном классе типов.

This hint is enough for the compiler to infer the correct type for our generic tail function above:

Этой подсказки достаточно, чтобы компилятор вывел правильный тип для нашей функции выше, отдающей обобщенный хвост (generic tail).

```text
> :type genericTail
forall stream element. Stream stream element => stream -> Maybe stream

> genericTail "testing"
(Just "esting")
```

Functional dependencies can be quite useful when using multi-parameter type classes to design certain APIs.

Функциональные зависимости могут быть довольно полезны при использовании многопараметаризированных классах типов для создания определенных API.

## Nullary Type Classes
## Безаргументные классы типов

We can even define type classes with zero type arguments! These correspond to compile-time assertions about our functions, allowing us to track global properties of our code in the type system.

Мы даже можем определить классы типов у которых отсутствуют аргументы! Они соответствуют утвеждениям на этапе компиляции для наших функций, позволяя нам следить за свойствами нашего кода при помощи системы типов:

An important example is the `Partial` class which we saw earlier when discussing partial functions. We've seen the partial functions `head` and `tail`, defined in `Data.Array.Partial` already:

Важным примером является класс `Partial`, который мы видели ранее, когда обсуждали частичные функции. Мы уже рассматривали частичные функции `head` и `tail`, определённые в `Data.Array.Partial`:

```haskell
head :: forall a. Partial => Array a -> a

tail :: forall a. Partial => Array a -> Array a
```

Note that there is no instance defined for the `Partial` type class! Doing so would defeat its purpose: attempting to use the `head` function directly will result in a type error:

Обратите внимание что для класса типов `Partial` не определён экземпляр! Это противоречивало бы его цели: попытка непосредственного использования функции `head` приведёт к ошибке типов:

```text
> head [1, 2, 3]

No type class instance was found for

  Prim.Partial
```

Instead, we can republish the `Partial` constraint for any functions making use of partial functions:

Вместо этого, мы можем указывать ограничение `Partial` для любых функций, которые используют частичные функции:

```haskell
secondElement :: forall a. Partial => Array a -> a
secondElement xs = head (tail xs)
```

We've already seen the `unsafePartial` function, which allows us to treat a partial function as a regular function (unsafely). This function is defined in the `Partial.Unsafe` module:

Мы уже видели функцию `unsafePartial`, которая позволяет нам обращаться с частичной функцией как с обычной (небезопасно). Эта функция определена в модуле `Partial.Unsafe`:

```haskell
unsafePartial :: forall a. (Partial => a) -> a
```

Note that the `Partial` constraint appears _inside the parentheses_ on the left of the function arrow, but not in the outer `forall`. That is, `unsafePartial` is a function from partial values to regular values.

Обратите внимание, что ограничение `Partial` представлено _внутри круглых скобок_ слева от функциональной стрелки, но не во внешнем `forall` (пер. не понял фразы после запятой). То есть, `unsafePartial` - это функция из частичных значений в обычные.

## Superclasses
## Суперклассы

Just as we can express relationships between type class instances by making an instance dependent on another instance, we can express relationships between type classes themselves using so-called _superclasses_.

Так же, как мы можем выразить отношения между экземплярами классов типов, создавая экземпляр, зависящий от другого экземпляра, мы так же можем выразить отношения между самими классами типов, используя так называемые _суперклассы_.

We say that one type class is a superclass of another if every instance of the second class is required to be an instance of the first, and we indicate a superclass relationship in the class definition by using a backwards facing double arrow.

Мы говорим, что один класс типов является суперклассом другого, если каждый экземпляр второго класса требуется в качестве экземпляра первого. И мы обозначаем отношение суперкласса в определении класса, используя обратную двойную стрелку.

We've already seen some examples of superclass relationships: the `Eq` class is a superclass of `Ord`, and the `Semigroup` class is a superclass of `Monoid`. For every type class instance of the `Ord` class, there must be a corresponding `Eq` instance for the same type. This makes sense, since in many cases, when the `compare` function reports that two values are incomparable, we often want to use the `Eq` class to determine if they are in fact equal.

Мы уже видели несколько примеров отношений суперкласса: класс `Eq` является суперклассом `Ord`, а `Semigroup` является суперклассом `Monoid`. Для каждого экземпляра класса типов `Ord` должен быть соответствующий экземпляр `Eq` одного и того же типа. 

In general, it makes sense to define a superclass relationship when the laws for the subclass mention the members of the superclass. For example, it is reasonable to assume, for any pair of `Ord` and `Eq` instances, that if two values are equal under the `Eq` instance, then the `compare` function should return `EQ`. In order words, `a == b` should be true exactly when `compare a b` evaluates to `EQ`. This relationship on the level of laws justifies the superclass relationship between `Eq` and `Ord`.

Another reason to define a superclass relationship is in the case where there is a clear "is-a" relationship between the two classes. That is, every member of the subclass _is a_ member of the superclass as well.

X> ## Exercises
X>
X> 1. (Medium) The `Action` class is a multi-parameter type class which defines an action of one type on another:
X>
X>     ```haskell
X>     class Monoid m <= Action m a where
X>       act :: m -> a -> a
X>     ```
X>           
X>     An _action_ is a function which describes how a monoid can be used to modify a value of another type. We expect the action to respect the concatenation operator of the monoid. For example, the monoid of natural numbers under addition (defined in the `Data.Monoid.Additive` module) _acts_ on strings by repeating a string some number of times:
X>  
X>     ```haskell
X>     import Data.Monoid.Additive (Additive(..))
X>
X>     instance repeatAction :: Action (Additive Int) String where
X>       act (Additive n) s = repeat n s where
X>         repeat 0 _ = ""
X>         repeat m s = s <> repeat (m - 1) s
X>     ```
X>
X>     Note that `act (Additive 2) s` is equal to the combination `act (Additive 1) s <> act (Additive 1) s`, and `Additive 1 <> Additive 1 = Additive 2`.
X>   
X>     Write down a reasonable set of laws which describe how the `Action` class should interact with the `Monoid` class. _Hint_: how do we expect `mempty` to act on elements? What about `append`?
X> 1. (Medium) Write an instance `Action m a => Action m (Array a)`, where the action on arrays is defined by acting on the elements independently.
X> 1. (Difficult) Should the arguments of the multi-parameter type class `Action` be related by some functional dependency? Why or why not?
X> 1. (Difficult) Given the following newtype, write an instance for `Action m (Self m)`, where the monoid `m` acts on itself using `append`:
X>
X>     ```haskell
X>     newtype Self m = Self m
X>     ```
X>
X> 1. (Medium) Define a partial function which finds the maximum of a non-empty array of integers. Your function should have type `Partial => Array Int -> Int`. Test out your function in PSCi using `unsafePartial`. _Hint_: Use the `maximum` function from `Data.Foldable`.

## A Type Class for Hashes

In the last section of this chapter, we will use the lessons from the rest of the chapter to create a library for hashing data structures.

Note that this library is for demonstration purposes only, and is not intended to provide a robust hashing mechanism.

What properties might we expect of a hash function?

- A hash function should be deterministic, and map equal values to equal hash codes.
- A hash function should distribute its results approximately uniformly over some set of hash codes.

The first property looks a lot like a law for a type class, whereas the second property is more along the lines of an informal contract, and certainly would not be enforceable by PureScript's type system. However, this should provide the intuition for the following type class:

```haskell
newtype HashCode = HashCode Int

hashCode :: Int -> HashCode
hashCode h = HashCode (h `mod` 65535)

class Eq a <= Hashable a where
  hash :: a -> HashCode
```

with the associated law that `a == b` implies `hash a == hash b`.

We'll spend the rest of this section building a library of instances and functions associated with the `Hashable` type class.

We will need a way to combine hash codes in a deterministic way:

```haskell
combineHashes :: HashCode -> HashCode -> HashCode
combineHashes (HashCode h1) (HashCode h2) = hashCode (73 * h1 + 51 * h2)
```

The `combineHashes` function will mix two hash codes and redistribute the result over the interval 0-65535.

Let's write a function which uses the `Hashable` constraint to restrict the types of its inputs. One common task which requires a hashing function is to determine if two values hash to the same hash code. The `hashEqual` relation provides such a capability:

```haskell
hashEqual :: forall a. Hashable a => a -> a -> Boolean
hashEqual = eq `on` hash
```

This function uses the `on` function from `Data.Function` to define hash-equality in terms of equality of hash codes, and should read like a declarative definition of hash-equality: two values are "hash-equal" if they are equal after each value has been passed through the `hash` function.

Let's write some `Hashable` instances for some primitive types. Let's start with an instance for integers. Since a `HashCode` is really just a wrapped integer, this is simple - we can use the `hashCode` helper function:

```haskell
instance hashInt :: Hashable Int where
  hash = hashCode
```

We can also define a simple instance for `Boolean` values using pattern matching:

```haskell
instance hashBoolean :: Hashable Boolean where
  hash false = hashCode 0
  hash true  = hashCode 1
```

With an instance for hashing integers, we can create an instance for hashing `Char`s by using the `toCharCode` function from `Data.Char`:

```haskell
instance hashChar :: Hashable Char where
  hash = hash <<< toCharCode
```

To define an instance for arrays, we can `map` the `hash` function over the elements of the array (if the element type is also an instance of `Hashable`) and then perform a left fold over the resulting hashes using the `combineHashes` function:

```haskell
instance hashArray :: Hashable a => Hashable (Array a) where
  hash = foldl combineHashes (hashCode 0) <<< map hash
```

Notice how we build up instances using the simpler instances we have already written. Let's use our new `Array` instance to define an instance for `String`s, by turning a `String` into an array of `Char`s:

```haskell
instance hashString :: Hashable String where
  hash = hash <<< toCharArray
```

How can we prove that these `Hashable` instances satisfy the type class law that we stated above? We need to make sure that equal values have equal hash codes. In cases like `Int`, `Char`, `String` and `Boolean`, this is simple because there are no values of those types which are equal in the sense of `Eq` but not equal identically.

What about some more interesting types? To prove the type class law for the `Array` instance, we can use induction on the length of the array. The only array with length zero is `[]`. Any two non-empty arrays are equal only if they have equals head elements and equal tails, by the definition of `Eq` on arrays. By the inductive hypothesis, the tails have equal hashes, and we know that the head elements have equal hashes if the `Hashable a` instance must satisfy the law. Therefore, the two arrays have equal hashes, and so the `Hashable (Array a)` obeys the type class law as well.

The source code for this chapter includes several other examples of `Hashable` instances, such as instances for the `Maybe` and `Tuple` type.

X> ## Exercises
X>
X> 1. (Easy) Use PSCi to test the hash functions for each of the defined instances.
X> 1. (Medium) Use the `hashEqual` function to write a function which tests if an array has any duplicate elements, using hash-equality as an approximation to value equality. Remember to check for value equality using `==` if a duplicate pair is found. _Hint_: the `nubBy` function in `Data.Array` should make this task much simpler.
X> 1. (Medium) Write a `Hashable` instance for the following newtype which satisfies the type class law:
X>
X>     ```haskell
X>     newtype Hour = Hour Int
X>     
X>     instance eqHour :: Eq Hour where
X>       eq (Hour n) (Hour m) = mod n 12 == mod m 12
X>     ```
X>     
X>     The newtype `Hour` and its `Eq` instance represent the type of integers modulo 12, so that 1 and 13 are identified as equal, for example. Prove that the type class law holds for your instance.
X> 1. (Difficult) Prove the type class laws for the `Hashable` instances for `Maybe`, `Either` and `Tuple`.

## Conclusion

In this chapter, we've been introduced to _type classes_, a type-oriented form of abstraction which enables powerful forms of code reuse. We've seen a collection of standard type classes from the PureScript standard libraries, and defined our own library based on a type class for computing hash codes.

This chapter also gave an introduction to the notion of type class laws, a technique for proving properties about code which uses type classes for abstraction. Type class laws are part of a larger subject called _equational reasoning_, in which the properties of a programming language and its type system are used to enable logical reasoning about its programs. This is an important idea, and will be a theme which we will return to throughout the rest of the book.
