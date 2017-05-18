# Recursion, Maps And Folds
# Рекурсия, отображения и свёртки

## Цели главы

In this chapter, we will look at how recursive functions can be used to structure algorithms. Recursion is a basic technique used in functional programming, which we will use throughout this book.

В этой главе мы увидим как задавать алгоритмы при помощи рекурсивных функций. Рекурсия - одна из основных техник функционального программирования, которую мы будем использовать на протяжение всей книги.

We will also cover some standard functions from PureScript's standard libraries. We will see the `map` and `fold` functions, as well as some useful special cases, like `filter` and `concatMap`.

Мы также рассмотрим некоторые стандартные функции из стандартных библиотек PureScript. Мы увидим такие функции как `map` и `fold`, и их частные случаи, как `filter` и `concatMap`.

The motivating example for this chapter is a library of functions for working with a virtual filesystem. We will apply the techniques learned in this chapter to write functions which compute properties of the files represented by a model of a filesystem.

Мотивирующий пример данной главы - библиотека функций для работы с виртуальной файловой системой. Техники этой главы мы применим для написания функций, вычисляющих свойства файлов, представленных моделью файловой системы.

## Project Setup
## Настройка проекта

Исходный код примеров этой главы содержится в двух файлах - `src/Data/Path.purs` и `src/FileOperations.purs`.
Модуль `Data.Path` содержит модель виртуальной файловой системы. Вам не нужно изменять содержимое этого модуля.
Модуль `FileOperations` содержит функции, которые используют API `Data.Path`. Решения к упражнениям могут быть добавлены к этому модулю.

Проект имеет следующие зависимости Bower:

- `purescript-maybe`, который определяет тип `Maybe`
- `purescript-arrays`, определяющий функции для работы с массивами
- `purescript-strings`, содержащий функции для работы со строками Javascript
- `purescript-foldable-traversable`, который содержит функции для свертки массивов и других структур данных 
- `purescript-console`, предоставляющий функции печати в консоль

## Введение

Recursion is an important technique in programming in general, but particularly common in pure functional programming, because, as we will see in this chapter, recursion helps to reduce the mutable state in our programs.

Рекурсия является важной техникой для программирования вообще, но особенно часто встречается в чистом функциональном программировании, потому что, как мы увидем в этой главе, она помогает сократить количество изменяемого состояния в наших программах.

Recursion is closely linked to the _divide and conquer_ strategy: to solve a problem on certain inputs, we can break down the inputs into smaller parts, solve the problem on those parts, and then assemble a solution from the partial solutions.

Рекурсия тесно связана со стратегией _разделяй и властвуй_: чтобы решить задачу на некоторых входных данных, мы разбиваем данные на меньшие части, решаем задачу на этих частичных данных, а потом собираем общее решение из частных решений.

Давайте рассмотрим простые примеры рекурсии в PureScript.

Вот обычный пример _функции нахождения факториала_:

```haskell
fact :: Int -> Int
fact 0 = 1
fact n = n * fact (n - 1)
```

Here, we can see how the factorial function is computed by reducing the problem to a subproblem - that of computing the factorial of a smaller integer. When we reach zero, the answer is immediate.

Тут мы видим как функция факториала вычисляется путем сведения задачи к более простой - вычисления факториала меньшего числа. Когда мы достигаем нуля, ответ очевиден.

Here is another common example, which computes the _Fibonnacci function_:

Вот еще один распространённый пример - _функция вычисляющая число Фибоначчи_:

```haskell
fib :: Int -> Int
fib 0 = 1
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
```

Again, this problem is solved by considering the solutions to subproblems. In this case, there are two subproblems, corresponding to the expressions `fib (n - 1)` and `fib (n - 2)`. When these two subproblems are solved, we assemble the result by adding the partial results.

И снова задача решается путем разбиения на позадачи. В данном случае, есть две подзадачи, соответствующие выражениям `fib (n - 1)` и `fib (n - 2)`. Когда две эти задачи решены, мы собираем результат путем сложения частичных результатов.


## Recursion on Arrays
## Рекурсия на массивах

We are not limited to defining recursive functions over the `Int` type! We will see recursive functions defined over a wide array of data types when we cover _pattern matching_ later in the book, but for now, we will restrict ourselves to numbers and arrays.

Мы не ограничены созданием рекурсий на типе `Int`! Мы увидем, что рекурсивные функции определяются на широком массиве типов данных когда дойдем до _сопоставления с образцом_ далее в книге, но сейчас мы ограничемся числами и массивами.

Just as we branch based on whether the input is non-zero, in the array case, we will branch based on whether the input is non-empty. Consider this function, which computes the length of an array using recursion:

Подобно тому, как мы ответвлялись на случае, когда аргумент был не равен нулю, в случае с массивом, мы будем ответвлятся на случае когда массив не пустой. Рассмотрим данную функцию, вычисляющую длину массива, используя рекурсию:


```haskell
import Prelude

import Data.Array (null)
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

length :: forall a. Array a -> Int
length arr =
  if null arr
    then 0
    else 1 + length (unsafePartial tail arr)
```

In this function, we use an `if .. then .. else` expression to branch based on the emptiness of the array. The `null` function returns `true` on an empty array. Empty arrays have length zero, and a non-empty array has a length that is one more than the length of its tail.

В данной функции мы используем выражение `if .. then .. else` для ветвления на условии пустоты массива. Функция `null` возвращает `true` если массив пустой. Пустые массивы имеют нулевую длину, а непустые - 1 плюс длина хвоста **(не понял фразу после запятой, написал своими словами)**

This example is obviously a very impractical way to find the length of an array in JavaScript, but should provide enough help to allow you to complete the following exercises:

Очевидно, этот пример - очень непрактичный способ нахождения длины массива в JavaScript, но он должен обеспечить достаточную помощь, чтобы позволить вам решить следующие упражнения:

X> ## Упражнения

X>

X> 1. (Easy) Write a recursive function which returns `true` if and only if its input is an even integer.

X> 1. (Легкое) Напишите рекурсивную функцию, которая возвращает `true` тогда, и только тогда, когда на вход подаётся четное число. (не понял)

X> 2. (Medium) Write a recursive function which counts the number of even integers in an array. _Hint_: the function `unsafePartial head` (where `head` is also imported from `Data.Array.Partial`) can be used to find the first element in a non-empty array.

X> 2. (Среднее) Напишите рекурсивную функцию, которая подсчитывает число четных чисел в массиве. _Подсказка_: функция `unsafePartial head` (где `head` также импортирована из `Data.Array.Partial`) может быть использована для получения первого элемента из непустого массива.

## Maps
## Отображения

The `map` function is an example of a recursive function on arrays. It is used to transform the elements of an array by applying a function to each element in turn. Therefore, it changes the _contents_ of the array, but preserves its _shape_ (i.e. its length).

Функция `map` - это пример рекурсивной функции на массивах. Она используется для изменения элементов массива, путем применения передаваемой ей функции к каждому элементу по-очереди. Следовательно, она изменяет _содержимое_ массива, но сохраняет его _форму_ (то есть длину).

When we cover _type classes_ later in the book we will see that the `map` function is an example of a more general pattern of shape-preserving functions which transform a class of type constructors called _functors_.

Когда мы ознакомимся с _классами типов_ далее в книге, то увидем, что функция `map` это пример более общего шаблона функций, сохраняющих форму. Они образуют класс конструкторов типа под названием _функторы_. (?)

Let's try out the `map` function in PSCi:

Давайте попробуем функцию `map` в PSCi:

```text
$ psci

> import Prelude
> map (\n -> n + 1) [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

Notice how `map` is used - we provide a function which should be "mapped over" the array in the first argument, and the array itself in its second.

Обратите внимание на то, как используется функция `map` - в качестве первого аргумента мы передаем функцию, "шагаещую по массиву", и во второй аргумент - сам массив.

## Infix Operators
## Инфиксные операторы

The `map` function can also be written between the mapping function and the array, by wrapping the function name in backticks:

Функция `map` может быть так же записана между передаваемой функцией и массивом, включив название функции в обратные кавычки:

```text
> (\n -> n + 1) `map` [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

This syntax is called _infix function application_, and any function can be made infix in this way. It is usually most appropriate for functions with two arguments.

Такой синтаксис называется _инфиксным применением функции_, и любая функция может быть инфиксной таким образом. Это наиболее подходящий способ записи для функций с двумя аргументами.

There is an operator which is equivalent to the `map` function when used with arrays, called `<$>`. This operator can be used infix like any other binary operator:

Есть оператор, который эквивалентен функции `map` применённой к массиву, записываемый как `<$>`. Этот оператор может быть использован в инфиксной форме, как и любой бинарный оператор:

```text
> (\n -> n + 1) <$> [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

Let's look at the type of `map`:

Давайте посмотрим на тип функции `map`:

```text
> :type map
forall a b f. (Functor f) => (a -> b) -> f a -> f b
```

The type of `map` is actually more general than we need in this chapter. For our purposes, we can treat `map` as if it had the following less general type:

Тип функции `map` является более обобщенным, чем тот, который нужен в нашей главе. Для наших целей, мы можем рассматривать `map` так, если бы она имела следующий, менее общий тип:

```text
forall a b. (a -> b) -> Array a -> Array b
```

This type says that we can choose any two types, `a` and `b`, with which to apply the `map` function. `a` is the type of elements in the source array, and `b` is the type of elements in the target array. In particular, there is no reason why `map` has to preserve the type of the array elements. We can use `map` or `<$>` to transform integers to strings, for example:

Этот тип говорит, что мы можем выбрать любые два типа `a` и `b`, с которыми применяется функция `map`. `a` это тип элементов в исходном массиве, а `b` - тип элементов в целевом массиве. В частности, нет никаких причин, почему `map` должен сохранять тип элементов массива. Мы можем использовать `map` или `<$>` для преобразования целых чисел в строки, к примеру:

```text
> show <$> [1, 2, 3, 4, 5]

["1","2","3","4","5"]
```

Even though the infix operator `<$>` looks like special syntax, it is in fact just an alias for a regular PureScript function. The function is simply _applied_ using infix syntax. In fact, the function can be used like a regular function by enclosing its name in parentheses. This means that we can used the parenthesized name `(<$>)` in place of `map` on arrays:

Хоть инфиксный оператор `<$>` выглядит как специальный синтаксис, на самом деле - это всего лишь псевдоним для обычной функции PureScript. Функция просто _применяется_ используя инфиксный синтаксис. В действительности функцию можно использовать как обычную функцию, заключив её имя в скобки. Это означает, что мы можем ипользовать функцию в скобках `(<$>)` вместо `map` на массивах:


```text
> (<$>) show [1, 2, 3, 4, 5]
["1","2","3","4","5"]
```

Infix function names are defined as _aliases_ for existing function names. For example, the `Data.Array` module defines an infix operator `(..)` as a synonym for the `range` function, as follows:

Имена инфиксных функций определяются как _псевдонимы_ для существующих названий функций. Например, модуль `Data.Array` определяет инфиксный оператор `(..)` как синоним функции `range` следующим образом:

```haskell
infix 8 range as ..
```

We can use this operator as follows:

Мы можем использовать этот оператор вот так:

```text
> import Data.Array

> 1 .. 5
[1, 2, 3, 4, 5]

> show <$> (1 .. 5)
["1","2","3","4","5"]
```

_Note_: Infix operators can be a great tool for defining domain-specific languages with a natural syntax. However, used excessively, they can render code unreadable to beginners, so it is wise to exercise caution when defining any new operators.

_Примечание_: Инфиксные операторы могут быть отличным инструментом для определения предметно-ориентированных языков с естественным синтаксисом. Однако, чрезмерное использование может сделать код нечитаемым для новичков, поэтому стоит быть осторожным при определении каких-либо новых операторов.

In the example above, we parenthesized the expression `1 .. 5`, but this was actually not necessary, because the `Data.Array` module assigns a higher precedence level to the `..` operator than that assigned to the `<$>` operator. In the example above, the precedence of the `..` operator was defined as `8`, the number after the `infix` keyword. This is higher than the precedence level of `<$>`, meaning that we do not need to add parentheses:

В примере выше мы заключили выражение `1 .. 5` в скобки, но на самом деле это необязательно, потому что модуль `Data.Array` присваивает бóльший приоритет оператору `..` чем `<$>`. В примере выше, приоретет оператора `..` определен как `8` - число после ключевого слова `infix`. Это больше, чем приоритет у `<$>`, означающий, что нам не нужно добавлять скобки: 

```text
> show <$> 1 .. 5
["1","2","3","4","5"]
```

If we wanted to assign an _associativity_ (left or right) to an infix operator, we could do so with the `infixl` and `infixr` keywords instead.

Если мы хотим присвоить _ассоциативность_ (правую или левую) к инфиксному оператору, мы можем использовать ключевые слова `infixl` и `infixr`.

## Filtering Arrays
## Фильтрация массивов

The `Data.Array` module provides another function `filter`, which is commonly used together with `map`. It provides the ability to create a new array from an existing array, keeping only those elements which match a predicate function.

Модуль `Data.Array` предоставляет еще одну функцию под названием `filter`, которая обычно используется вместе с `map`. Она позволяет создавать новый массив из существующего, сохраняя только те элементы, которые удовлетворяют условию функции-предиката.

For example, suppose we wanted to compute an array of all numbers between 1 and 10 which were even. We could do so as follows:

Например, предположим, что мы хотим найти все чётные элементы массива чисел от 1 до 10. Мы могли бы сделать это следующим образом:

```text
> import Data.Array

> filter (\n -> n `mod` 2 == 0) (1 .. 10)
[2,4,6,8,10]
```

X> ## Упражнения
X>
X> 1. (Easy) Use the `map` or `<$>` function to write a function which calculates the squares of an array of numbers.

X> 1. (Легкое) Используя функции `map` или `<$>` напишите функцию, вычисляющую квадраты в массиве чисел.

X> 1. (Easy) Use the `filter` function to write a function which removes the negative numbers from an array of numbers.

X> 1. (Легкое) Используя функцию `filter` напишите функцию, которая удаляет все отрицательные числа из массива чисел.

X> 1. (Medium) Define an infix synonym `<$?>` for `filter`. Rewrite your answer to the previous question to use your new operator. Experiment with the precedence level and associativity of your operator in PSCi.

X> 1. (Среднее) Определите синоним `<$?>` для `filter`. Перепишите предыдущее решение, используя новый оператор. По-экспериментируйте с приоритетом и ассоциативностью оператора в PSCi.

## Flattening Arrays
## Уплощение массивов

Another standard function on arrays is the `concat` function, defined in `Data.Array`. `concat` flattens an array of arrays into a single array:

Другая стандартная функция для массивов - `concat`, определённая в `Data.Array`. `concat` уплощает массив массивов в единый массив:

```text
> import Data.Array

> :type concat
forall a. Array (Array a) -> Array a

> concat [[1, 2, 3], [4, 5], [6]]
[1, 2, 3, 4, 5, 6]
```

There is a related function called `concatMap` which is like a combination of the `concat` and `map` functions. Where `map` takes a function from values to values (possibly of a different type), `concatMap` takes a function from values to arrays of values.

Существую так же функция под названием `concatMap`, которая подобна комбинации функций `concat` и `map`. Если `map` использует функцию из значений в значения (которые могут быть другого типа), `concatMap` использует функцию из значений в массив значений: 

Let's see it in action:

Давайте посмотрим на неё в действии:

```text
> import Data.Array

> :type concatMap
forall a b. (a -> Array b) -> Array a -> Array b

> concatMap (\n -> [n, n * n]) (1 .. 5)
[1,1,2,4,3,9,4,16,5,25]
```

Here, we call `concatMap` with the function `\n -> [n, n * n]` which sends an integer to the array of two elements consisting of that integer and its square. The result is an array of ten integers: the integers from 1 to 5 along with their squares.

Здесь мы вызываем `concatMap` вместе с функцией `\n -> [n, n * n]`, которая отправляет целое число в массив из двух элементов, состоящих из этого целого числа и его квадрата. Результат является массивом из десяти чисел: целые числа от 1 до 5 рядом с их квадратами.

Note how `concatMap` concatenates its results. It calls the provided function once for each element of the original array, generating an array for each. Finally, it collapses all of those arrays into a single array, which is its result.

Обратите внимание как `concatMap` соединяет свои результаты. Она вызывает предоставленную функцию по-одному разу на каждом элементе исходного массива, тем самым генерируя к каждому элементу свой массив. Наконец, она схлопывает все эти массивы в единый массив, который и является результатом.

`map`, `filter` and `concatMap` form the basis for a whole range of functions over arrays called "array comprehensions".

`map`, `filter` и `concatMap` формируют базу для целого спектра функций на массивах, под названием "генераторы массивов" (array comprehensions). 

## Array Comprehensions
## Генераторы массивов

Suppose we wanted to find the factors of a number `n`. One simple way to do this would be by brute force: we could generate all pairs of numbers between 1 and `n`, and try multiplying them together. If the product was `n`, we would have found a pair of factors of `n`.

Предположим, что мы хотим найти все множители числа `n`. Один простой способ это сделать - мог быть грубый пебор: мы могли бы сгенерировать пары чисел от 1 до `n` и попытаться их перемножить. Если произведение будет равно `n` - мы нашли пару множителей числа `n`.

We can perform this computation using an array comprehension. We will do so in steps, using PSCi as our interactive development environment.

Мы можем произвести это вычисление используя генератор массива. Мы будем делать это по-шагам, используя PSCi в качестве нашей среды разработки. 

The first step is to generate an array of pairs of numbers below `n`, which we can do using `concatMap`.

Первым шагом будет генерация массива пар чисел меньше `n`, которую можно сделать, используя `concatMap`.

Let's start by mapping each number to the array `1 .. n`:

Давайте начнем с отображения каждого числа на массив `1 .. n`:

```text
> let pairs n = concatMap (\i -> 1 .. n) (1 .. n)
```

We can test our function

Протестируем функцию

```text
> pairs 3
[1,2,3,1,2,3,1,2,3]
```

This is not quite what we want. Instead of just returning the second element of each pair, we need to map a function over the inner copy of `1 .. n` which will allow us to keep the entire pair:

Это не совсем то что мы хотели. Вместо того, чтобы просто возвращать второй элемент каждой пары, нам нужно отобразить функцию на внутреннюю копию `1 .. n`, что позволит сохранить целую пару:

```text
> :paste
… let pairs' n =
…       concatMap (\i ->
…         map (\j -> [i, j]) (1 .. n)
…       ) (1 .. n)
… ^D

> pairs' 3
[[1,1],[1,2],[1,3],[2,1],[2,2],[2,3],[3,1],[3,2],[3,3]]
```

This is looking better. However, we are generating too many pairs: we keep both [1, 2] and [2, 1] for example. We can exclude the second case by making sure that `j` only ranges from `i` to `n`:

Выглядит уже лучше. Однако, мы генерируем слишком много пар: мы создаем и [1, 2], и [2, 1], к примеру. Мы можем исключить второй случай убедившись в том, что `j` находится в диапазоне от `i` до `n`:

```text
> :paste
… let pairs'' n =
…       concatMap (\i ->
…         map (\j -> [i, j]) (i .. n)
…       ) (1 .. n)
… ^D
> pairs'' 3
[[1,1],[1,2],[1,3],[2,2],[2,3],[3,3]]
```

Great! Now that we have all of the pairs of potential factors, we can use `filter` to choose the pairs which multiply to give `n`:

Отлично! Теперь, когда у нас есть все потенциальные множители, мы можем использовать `filter`, выбрав только те пары, которые являются множителями `n`:

```text
> import Data.Foldable

> let factors n = filter (\pair -> product pair == n) (pairs'' n)

> factors 10
[[1,10],[2,5]]
```

This code uses the `product` function from the `Data.Foldable` module in the `purescript-foldable-traversable` library.

Этот код использует функцию `product` из модуля `Data.Foldable` в библиотеке `purescript-foldable-traversavle`.

Excellent! We've managed to find the correct set of factor pairs without duplicates.

Превосходно! Нам удалось найти правильный набор пар множителей без дублей.

## Do-нотация

However, we can improve the readability of our code considerably. `map` and `concatMap` are so fundamental, that they (or rather, their generalizations `map` and `bind`) form the basis of a special syntax called _do notation_.

Тем не менее, мы можем значительно улучшить читаемость нашего кода. `map` и `concatMap` настолько фундаментальны, что они (или скорее, их обобщения `map` и `bind`) формируют базу специального синтаксиса под названием _do-нотация_.

_Note_: Just like `map` and `concatMap` allowed us to write _array comprehensions_, the more general operators `map` and `bind` allow us to write so-called _monad comprehensions_. We'll see plenty more examples of _monads_ later in the book, but in this chapter, we will only consider arrays.

_Примечание_: Так же как `map` и `concatMap` позволяет нам создавать _генераторы массивов_, более общие операторы `map` и `bind` позволяют писать созвучные _генераторы монад_.  Мы увидем еще много примеров _монад_ далее в книге, но в этой главе мы рассмотрим только массивы.

We can rewrite our `factors` function using do notation as follows:

Мы можем переписать нашу функцию `factors`, используя do-нотацию, следующим образом:

```haskell
factors :: Int -> Array (Array Int)
factors n = filter (\xs -> product xs == n) $ do
  i <- 1 .. n
  j <- i .. n
  pure [i, j]
```

The keyword `do` introduces a block of code which uses do notation. The block consists of expressions of a few types:

Ключевое слово `do`, открывает блок кода, использующего do-нотацию. Блок состоит из выражений нескольких типов:

- Expressions which bind elements of an array to a name. These are indicated with the backwards-facing arrow `<-`, with a name on the left, and an expression on the right whose type is an array. 

- Выражения, которые связывают элементы массива с именем. Они обозначены стрелкой назад `<-`, с именем слева, и с выражением справа, чей тип является массивом. 

- Expressions which do not bind elements of the array to names. The last line `pure [i, j]` is an example of this kind of expression.

- Выражения, которые не связывают элементы массивов с именами. Последняя строка `pure [i, j]` является примером такого рода выражений.

- Expressions which give names to expressions, using the `let` keyword.

- Выражения, которые присваивают имена выражениям, используя ключевое слово `let`.

This new notation hopefully makes the structure of the algorithm clearer. If you mentally replace the arrow `<-` with the word "choose", you might read it as follows: "choose an element `i` between 1 and n, then choose an element `j` between `i` and `n`, and return `[i, j]`".

Эта новая нотация, мы надеямся, делает структуру алгоритма более ясной. Если вы мысленно замените стрелку `<-` словом "выбрать", вы сможете прочитать его следующим образом: "выбрать элемент `i` между 1 и n, затем выбрать элемент `j` между `i` и `n`, а потом вернуть `[i, j]`".

In the last line, we use the `pure` function. This function can be evaluated in PSCi, but we have to provide a type:

В последней строке мы использовали функцию `pure`. Эта функция может быть использована в PSCi, но нам нужно указывать тип:

```text
> pure [1, 2] :: Array (Array Int)
[[1, 2]]
```

In the case of arrays, `pure` simply constructs a singleton array. In fact, we could modify our `factors` function to use this form, instead of using `pure`:

В случае массивов, `pure` просто создает одиночный массив. Фактически, мы можем изменить функцию `factors` для использования данной формы, вместо `pure`:

```haskell
factors :: Int -> Array (Array Int)
factors n = filter (\xs -> product xs == n) $ do
  i <- 1 .. n
  j <- i .. n
  [[i, j]]
```

and the result would be the same.

и результат будет такой же.

## Guards
## Охранные выражения

One further change we can make to the `factors` function is to move the filter inside the array comprehension. This is possible using the `guard` function from the `Control.MonadZero` module (from the `purescript-control` package):

Еще одно изменение которое мы можем сделать с функцией `factors` - это переместить фильтрацию в генератор массива. Это возможно, путем использования функции `guard` из модуля `Control.MonadZero` (пакета `purescript-control`):


```haskell
import Control.MonadZero (guard)

factors :: Int -> Array (Array Int)
factors n = do
  i <- 1 .. n
  j <- i .. n
  guard $ i * j == n
  pure [i, j]
```

Just like `pure`, we can apply the `guard` function in PSCi to understand how it works. The type of the `guard` function is more general than we need here:

Так же как и `pure`, мы можем проверить функцию `guard` в PSCi, чтобы понять как она работает. Тип функции `guard` более общий, чем тот который нам здесь нужен:

```text
> import Control.MonadZero

> :type guard
forall m. MonadZero m => Boolean -> m Unit
```

In our case, we can assume that PSCi reported the following type:

В нашем случае, мы можем допустить, что PSCi вывела следующий тип:


```haskell
Boolean -> Array Unit
```

For our purposes, the following calculations tell us everything we need to know about the `guard` function on arrays:

Для наших целей, следующие вычисления говорят нам всё, что нам нужно знать о функции `guard` на массивах:

```text
> import Data.Array

> length $ guard true
1

> length $ guard false
0
```

That is, if `guard` is passed an expression which evaluates to `true`, then it returns an array with a single element. If the expression evaluates to `false`, then its result is empty.

То есть, если `guard` получила на вход выражение, которое вычисляется в `true`, тогда возвращается массив с единственным элементом. Если выражение вычисляется в `false`, тогда результатом будет пустой массив.


This means that if the guard fails, then the current branch of the array comprehension will terminate early with no results. This means that a call to `guard` is equivalent to using `filter` on the intermediate array. Depending on the application, you might prefer to use `guard` instead of a `filter`. Try the two definitions of `factors` to verify that they give the same results.

Это означает, что если охранное выражение не выполнится, то текущая ветка генератора массива завершиться раньше, без результата. (?) Это значит, что вызов `guard` эквивалентен использованию `filter` на промежуточном массиве. В зависимости от приложения, вы можете предпочесть использование `guard` вместо `filter`. Попробуйте два варианта `factor`, чтобы убедиться, что оба они дают одинаковые результаты. 

X> ## Упражнения

X>
X> 1. (Easy) Use the `factors` function to define a function `isPrime` which tests if its integer argument is prime or not.

X> 1. (Лёгкое) Используйте функцию `factors` для написания функции `isPrime`, которое проверяет, является ли его аргумент (целое число) простым или нет.

X> 1. (Medium) Write a function which uses do notation to find the _cartesian product_ of two arrays, i.e. the set of all pairs of elements `a`, `b`, where `a` is an element of the first array, and `b` is an element of the second.

X> 1. (Среднее) Напишите функцию, которая использует do-нотацию для нахождения _декартового произведения_ двух массивов, то есть множество всех пар элементов `a`, `b`, где `a` - элемент первого массива, а `b` - элемент второго массива.


X> 1. (Medium) A _Pythagorean triple_ is an array of numbers `[a, b, c]` such that `a² + b² = c²`. Use the `guard` function in an array comprehension to write a function `triples` which takes a number `n` and calculates all Pythagorean triples whose components are less than `n`. Your function should have type `Int -> Array (Array Int)`.

X> 1. (Срнеднее)  _Пифагорова тройка_ - это массив чисел `[a, b, c]`, удовлетворяющих выражению `a² + b² = c²`. Используйте функцию `guard` в генераторе массива для написания функции `triples`, которая принимает число `n` и вычисляет все пифагоровы тройки, чьи компоненты меньше `n`. Ваша функция должна иметь тип `Int -> Array (Array Int)`.

X> 1. (Difficult) Write a function `factorizations` which produces all _factorizations_ of an integer `n`, i.e. arrays of integers whose product is `n`. _Hint_: for an integer greater than 1, break the problem down into two subproblems: finding the first factor, and finding the remaining factors.

X> 1. (Сложное) Напишите функцию `factorizations`, которая возвращает все _простые множители_ числа `n`, то есть массив чисел, чьё произведение равно `n`. _Подсказка_: Для целых чисел больше 1, разбейте задачу на две подзадачи: нахождения первого множителя, а затем нахождения оставшихся множетелей.

## Свёртки

Left and right folds over arrays provide another class of interesting functions which can be implemented using recursion.

Левые и правые свёртки массивов предоствляют еще один класс интересных функций, которые могут быть реализованы с использованием рекурсии.

Start by importing the `Data.Foldable` module, and inspecting the types of the `foldl` and `foldr` functions using PSCi:

Начнем с импорта модуля `Data.Foldable` и просмотра типов `foldl`, и `foldr`, используя PSCi:

```text
> import Data.Foldable

> :type foldl
forall a b f. (Foldable f) => (b -> a -> b) -> b -> f a -> b

> :type foldr
forall a b f. (Foldable f) => (a -> b -> b) -> b -> f a -> b
```

These types are actually more general than we are interested in right now. For the purposes of this chapter, we can assume that PSCi had given the following (more specific) answer:

Эти типы на самом деле более общие, чем те, что нас сейчас интересуют. Для целей нашей главы, мы предположим, что PSCi выдала следующий ответ:

```text
> :type foldl
forall a b. (b -> a -> b) -> b -> Array a -> b

> :type foldr
forall a b. (a -> b -> b) -> b -> Array a -> b
```

In both of these cases, the type `a` corresponds to the type of elements of our array. The type `b` can be thought of as the type of an "accumulator", which will accumulate a result as we traverse the array.

В обоих случаях тип `a` соответствует типу элементов нашего массива. Тип `b` можно рассматривать как тип "аккумулятора", который аккумулирует результат по мере прохождения по массиву.

The difference between the `foldl` and `foldr` functions is the direction of the traversal. `foldl` folds the array "from the left", whereas `foldr` folds the array "from the right".

Разница между функциями `foldl` и `foldr` это направление обхода. `foldl` свёртывает массив слева, а `foldr` свёртывает справа.

Let's see these functions in action. Let's use `foldl` to sum an array of integers. The type `a` will be `Int`, and we can also choose the result type `b` to be `Int`. We need to provide three arguments: a function `Int -> Int -> Int`, which will add the next element to the accumulator, an initial value for the accumulator of type `Int`, and an array of `Int`s to add. For the first argument, we can just use the addition operator, and the initial value of the accumulator will be zero:

Давайте увидем эти функции в действии. Используем `foldl` для складывание массива целых чисел. Типом `a` будет `Int`, и мы можем выбрать тип `Int` и для `b`. Нам нужно предоставить три аргумента: функцию `Int -> Int -> Int`, которая складывает следующий элемент с аккумулятором, начальное значение аккумулятора с типом `Int`, а также массив целых чисел для складывания. В качестве первого аргумента, мы можем просто использовать оператор сложения, а начальное значение аккумулятора будет ноль:

```text
> foldl (+) 0 (1 .. 5)
15
```

In this case, it didn't matter whether we used `foldl` or `foldr`, because the result is the same, no matter what order the additions happen in:

В данном случае, не имеет значения, какая используется функция `foldl` или `foldr`, потому что результат будет одинаковый. Не важно, в каком порядке происходит сложение:

```text
> foldr (+) 0 (1 .. 5)
15
```

Let's write an example where the choice of folding function does matter, in order to illustrate the difference. Instead of the addition function, let's use string concatenation to build a string:

Давайте напишем пример, где выбор функции будет иметь значение, чтобы продемонстрировать разницу. Вместо функции сложения, используем функцию конкатенации для построения строки: 

```text
> foldl (\acc n -> acc <> show n) "" [1,2,3,4,5]
"12345"

> foldr (\n acc -> acc <> show n) "" [1,2,3,4,5]
"54321"
```

This illustrates the difference between the two functions. The left fold expression is equivalent to the following application:

Это иллюстрирует разницу между двумя этими функциями. Выражение левой свёртки эквивалентно следующей конструкции:

```text
((((("" <> show 1) <> show 2) <> show 3) <> show 4) <> show 5)
```

whereas the right fold is equivalent to this:

в то время как правая свёртка эквивалентна этой:

```text
((((("" <> show 5) <> show 4) <> show 3) <> show 2) <> show 1)
```

## Tail Recursion
## Хвостовая рекурсия (нужно внимательно перечитать раздел)

Recursion is a powerful technique for specifying algorithms, but comes with a problem: evaluating recursive functions in JavaScript can lead to stack overflow errors if our inputs are too large.

Рекурсия является мощным средством для создания алгоритмов, но которая приходит с проблемой: вычисление рекурсивных функций в JavaScript может привести к ошибкам переполнения стека, если количество вызовов будет очень много.

It is easy to verify this problem, with the following code in PSCi:

Легко подтвердить эту проблему, введя следующий код в PSCi:


```text
> let f 0 = 0
      f n = 1 + f (n - 1)

> f 10
10

> f 10000
RangeError: Maximum call stack size exceeded
```

This is a problem. If we are going to adopt recursion as a standard technique from functional programming, then we need a way to deal with possibly unbounded recursion.

Это проблема. Если мы собираемся принять рекурсию как стандартную технику функционального программирования, тогда нам необходим способ борьбы с возможной неограниченной рекурсией.

PureScript provides a partial solution to this problem in the form of _tail recursion optimization_.

PureScript предоставляет частичное решение этой проблемы в форме _оптимизации хвостовой рекурсии_.

_Note_: more complete solutions to the problem can be implemented in libraries using so-called _trampolining_, but that is beyond the scope of this chapter. The interested reader can consult the documentation for the `purescript-free` and `purescript-tailrec` packages.

_Примечание_: более полные решения проблемы могут быть реализованы в библиотеках с использованием так называемого _трамплининга_ (_trampolining_), но этой выходит за рамки данной главы. Заинтересованный читатель может обратиться к документации пакетов `purescript-free` и `purescript-tailrec`. (?)

The key observation which enables tail recursion optimization is the following: a recursive call in _tail position_ to a function can be replaced with a _jump_, which does not allocate a stack frame. A call is in _tail position_ when it is the last call made before a function returns. This is the reason why we observed a stack overflow in the example - the recursive call to `f` was _not_ in tail position.

Ключевое наблюдение, которое позволяет оптимизировать хвостовую рекурсию, следующее: рекурсивный вызов функции в _хвостовой позиции_ может быть заменён на _прыжок_, который не занимает стековый фрейм. Вызов находится в _хвостовой позиции_, когда он сделан перед возвратом функции. Эта причина, почему мы наблюдали переполнение стека в примере выше - рекурсивный вызов `f` _не был_ в хвостовой позиции.

In practice, the PureScript compiler does not replace the recursive call with a jump, but rather replaces the entire recursive function with a _while loop_.

На практике, компилятор PureScript не заменяет рекурсивный вызов прыжком, а заменяет всю рекурсивную функцию циклом _while_.

Here is an example of a recursive function with all recursive calls in tail position:

Вот пример рекурсивной функции с вызовами в хвостовой позиции:

```haskell
fact :: Int -> Int -> Int
fact 0 acc = acc
fact n acc = fact (n - 1) (acc * n)
```

Notice that the recursive call to `fact` is the last thing that happens in this function - it is in tail position.

Обратите внимание что рекурсивный вызов `fact` это последнее, что происходит внутри функции - он в хвостовой позиции. (?)

## Аккумуляторы

One common way to turn a function which is not tail recursive into a tail recursive function is to use an _accumulator parameter_. An accumulator parameter is an additional parameter which is added to a function which _accumulates_ a return value, as opposed to using the return value to accumulate the result.

Один из известных способов превратить рекурсивную функцию, не являющейся хвостовой, в хвостовую - это использовать _аккумуляторный параметр_. Аккумуляторный параметр - это дополнительный параметр, добавляемый в функцию, который _аккумулирует_ возвращающее значение, в отличие от использования возвращаемого значения для аккумулирования результата.

For example, consider this array recursion which reverses the input array by appending elements at the head of the input array to the end of the result.

Для примера, рассмотрим рекурсию на массиве, которая меняет местами элементы массива путём добавления головы входного массива в конец результирующего.


```haskell
reverse :: forall a. Array a -> Array a
reverse [] = []
reverse xs = snoc (reverse (unsafePartial tail xs))
                  (unsafePartial head xs)
```

This implementation is not tail recursive, so the generated JavaScript will cause a stack overflow when executed on a large input array. However, we can make it tail recursive, by introducing a second function argument to accumulate the result instead:

Эта реализация не является хвостовой рекурсией, поэтому сгенерированный код будет создавать переполнение стека, при передачи в него большого массива. Однако, мы можем сделать из неё хвостовую рекурсию, добавив второй аргумент в функцию для аккумуляции результата:

```haskell
reverse :: forall a. Array a -> Array a
reverse = reverse' []
  where
    reverse' acc [] = acc
    reverse' acc xs = reverse' (unsafePartial head xs : acc)
                               (unsafePartial tail xs)
```

In this case, we delegate to the helper function `reverse'`, which performs the heavy lifting of reversing the array. Notice though that the function `reverse'` is tail recursive - its only recursive call is in the last case, and is in tail position. This means that the generated code will be a _while loop_, and will not blow the stack for large inputs.

В данном случае мы делегируем выполнение вызова вспомогательной функции `reverse'`, которая производит всю работу по перевёртыванию массива. Обратите внимание, однако, что функция `reverse'` является хвостовой рекурсией - она осуществляет рекурсивный вызов в последнем случае в хвостовой позиции. Это означает, что сгенерированный код будет циклом _while_, и не "взорвёт" стек на больших входных данных.

To understand the second implementation of `reverse`, note that the helper function `reverse'` essentially uses the accumulator parameter to maintain an additional piece of state - the partially constructed result. The result starts out empty, and grows by one element for every element in the input array. However, because later elements are added at the front of the array, the result is the original array in reverse!

Чтобы понять вторую реализацию `reverse`, обратите внимание, что вспомогательная функция `reverse'` по сути использует аккумуляторный параметр для сохранения кусочка состояния - это частично сконструированный результат. Результат сначала пустой, затем он возрастает по-одному элементу из входного массива. Однако, так как дальнейшие элементы добавляются в начало массива, результатом становится перевёрнутый оригинальный массив!

Note also that while we might think of the accumulator as "state", there is no direct mutation going on. The accumulator is an immutable array, and we simply use function arguments to thread the state through the computation.

Отметим, что пока мы могли думать об аккумуляторе как о "состоянии", здесь не происходит прямой мутации. Аккумулятор является неизменяемым массивом, мы просто используем аргументы функции чтобы связать состояние через вычисление. 

## Prefer Folds to Explicit Recursion
## Предпочтение Свёрток прямой рекурсии

If we can write our recursive functions using tail recursion, then we can benefit from tail recursion optimization, so it becomes tempting to try to write all of our functions in this form. However, it is often easy to forget that many functions can be written directly as a fold over an array or similar data structure. Writing algorithms directly in terms of combinators such as `map` and `fold` has the added advantage of code simplicity - these combinators are well-understood, and as such, communicate the _intent_ of the algorithm much better than explicit recursion.

Если мы можем писать наши рекурсивные функции используя хвостовую рекурсию, то мы можем извлечь выгоду из оптимизации хвостовой рекурсии, и поэтому написание наших функций в данной форме становится довольно заманчивым. Однако, легко забыть то, что многие функции могут быть написаны как _свёртки_ массивов или других структур данных. Написание алгоритмов с точки зрения комбинаторов, таких как `map` и `fold` имеет дополнительное приемущество в простоте кода - эти комбинаторы хорошо понятны, и поэтому связь с _намерением_ алгоритма более лучше, чем при использовании явной рекурсии.

For example, the `reverse` example can be written as a fold in at least two ways. Here is a version which uses `foldr`:

К примеру, код с `reverse` может быть написан в виде свёртки по крайней мере двумя способами. Тут версия с использованием `foldr`:

```text
> import Data.Foldable

> :paste
… let reverse :: forall a. Array a -> Array a
…     reverse = foldr (\x xs -> xs <> [x]) []
… ^D

> reverse [1, 2, 3]
[3,2,1]
```

Writing `reverse` in terms of `foldl` will be left as an exercise for the reader.

Написание `reverse` с использованием `foldl` будет оставлено в качестве упражнения читателю.

X> ## Упражнения
X>
X> 1. (Easy) Use `foldl` to test whether an array of boolean values are all true.

X> 1. (Лёгкое) Используйте `foldl` для проверки - состоит ли массив булевых значений только из true.

X> 2. (Medium) Characterize those arrays `xs` for which the function `foldl (==) false xs` returns true.

X> 2. (Среднее) Опишите массив `xs` для которого функция `foldl (==) false xs` возвращает true.

X> 3. (Medium) Rewrite the following function in tail recursive form using an accumulator parameter:

X> 3. (Среднее) Перепишите следующую функцию в форму хвостовой рекурсии, используя аккумуляторный параметр:

X>
X>     ```haskell
X>     import Prelude
X>     import Data.Array.Partial (head, tail)
X>     
X>     count :: forall a. (a -> Boolean) -> Array a -> Int
X>     count _ [] = 0
X>     count p xs = if p (unsafePartial head xs)
X>                    then count p (unsafePartial tail xs) + 1
X>                    else count p (unsafePartial tail xs)
X>     ```
X>
X> 4. (Medium) Write `reverse` in terms of `foldl`.

X> 4. (Среднее) Напишите `reverse` используя `foldl`.

## Виртуальная файловая система

In this section, we're going to apply what we've learned, writing functions which will work with a model of a filesystem. We will use maps, folds and filters to work with a predefined API.

В этом разделе мы собираемся применить то, что выучили, написав функции, которые будут работать с моделью файловой системы. Мы будем использовать отображения, свёртки и фильтры для работы с предопределённым API.

The `Data.Path` module defines an API for a virtual filesystem, as follows:

Модуль `Data.Path` определяет API виртуальной файловой системы следующим образом:

- There is a type `Path` which represents a path in the filesystem.
- There is a path `root` which represents the root directory.
- The `ls` function enumerates the files in a directory.
- The `filename` function returns the file name for a `Path`.
- The `size` function returns the file size for a `Path` which represents a file.
- The `isDirectory` function tests whether a function is a file or a directory.

- Тип `Path`, обозначающий путь в файловой системе
- Константа `root` типа `Path`, обозначающая корневую директорию
- Функция `ls`, перечисляющая файлы в директории
- Функция `size`, возвращающая размер файла, представленного типом `Path`
- Функция `isDirectory`, проверяющая - является ли путь файлом или директорией

In terms of types, we have the following type definitions:

В терминах типов мы имеем следующие определения:

```haskell
root :: Path

ls :: Path -> Array Path

filename :: Path -> String

size :: Path -> Maybe Number

isDirectory :: Path -> Boolean
```

We can try out the API in PSCi:

Мы можем проверить API в PSCi:

```text
$ pulp psci

> import Data.Path

> root
/

> isDirectory root
true

> ls root
[/bin/,/etc/,/home/]
```

The `FileOperations` module defines functions which use the `Data.Path` API. You do not need to modify the `Data.Path` module, or understand its implementation. We will work entirely in the `FileOperations` module.

Модуль `FileOperations` определяет функции для работы с API `Data.Path`. Вам не нужно изменять модуль `Data.Path` или понимать его реализацию. Мы будет целиком работать в модуле `FileOperations`.

## Listing All Files
## Вывод списка файлов

Let's write a function which performs a deep enumeration of all files inside a directory. This function will have the following type:

Давайте напишем функцию, выполняющую глубокое перечисление всех файлов внутри каталога. Эта функция будет иметь следующий тип:

```haskell
allFiles :: Path -> Array Path
```

We can define this function by recursion. First, we can use `ls` to enumerate the immediate children of the directory. For each child, we can recursively apply `allFiles`, which will return an array of paths. `concatMap` will allow us to apply `allFiles` and flatten the results at the same time.

Мы можем определить эту функцию при помощи рекурсии. Сначала, мы можем использовать `ls` для перечисления непосредственных дочерних элементов каталога. Для каждого дочернего мы можем применить рекурсивно функцию `allFiles`, которая возвращает массив путей. А `concatMap` позволит нам применять `allFiles` и одновременно уплощать результат.

Finally, we use the cons operator `:` to include the current file:

Наконец, мы используем оператор `:` чтобы прикрепить текущий файл:

```haskell
allFiles file = file : concatMap allFiles (ls file)
```

_Note_: the cons operator `:` actually has poor performance on immutable arrays, so it is not recommended in general. Performance can be improved by using other data structures, such as linked lists and sequences.

_Примечание_: оператор `:` на самом деле имеет низкую производительность на неизменяемых массивах, поэтому его использование не рекомендуется. Производительность может быть улучшена, если использовать другие структуры данных, такие как связный список и перечисления.

Let's try this function in PSCi:

Давайте проверим функцию в PSCi:

```text
> import FileOperations
> import Data.Path

> allFiles root

[/,/bin/,/bin/cp,/bin/ls,/bin/mv,/etc/,/etc/hosts, ...]
```

Great! Now let's see if we can write this function using an array comprehension using do notation.

Отлично! Теперь давайте посмотрим, как мы можем записать эту функцию используя генератор массива через do-нотацию.

Recall that a backwards arrow corresponds to choosing an element from an array. The first step is to choose an element from the immediate children of the argument. Then we simply call the function recursively for that file. Since we are using do notation, there is an implicit call to `concatMap` which concatenates all of the recursive results.

Вспомним, что обратная стрелка соответствует выбору элемента из массива. Первым шагом будет выбор элемента из прямых потомков аргумента. Затем мы просто запускаем функцию рекурсивно для данного элемента. Так как мы используем do-нотацию, то здесь происходит неявный вызов `concatMap`, который соединяет все результаты рекурсивных вызовов. 


Here is the new version:

Вот новая версия:

```haskell
allFiles' :: Path -> Array Path
allFiles' file = file : do
  child <- ls file
  allFiles' child
```

Try out the new version in PSCi - you should get the same result. I'll let you decide which version you find clearer.

Попробуйте новую версию в PSCi - вы должны получить такой же результат. Позволю вам самим решить, какую из этих версий вы найдете более ясной.

X> ## Упражнения
X>
X> 1. (Easy) Write a function `onlyFiles` which returns all _files_ (not directories) in all subdirectories of a directory.

X> 1. (Лёгкое) Напишите функцию `onlyFiles`, которая возвращает все _файлы_ (не каталоги) во всех подкаталогах директории.

X> 1. (Medium) Write a fold to determine the largest and smallest files in the filesystem.

X> 1. (Среднее) Напишите свёртку для определения самого большого и самого маленького файла в файловой системе.

X> 1. (Difficult) Write a function `whereIs` to search for a file by name. The function should return a value of type `Maybe Path`, indicating the directory containing the file, if it exists. It should behave as follows:

X> 1. (Сложное) Напишите функцию `whereIs` для поиска файла по имени. Функция должна возвращать значение типа `Maybe Path`, обозначающее папку, в которой содержится файл, если он существует. Её поведение должно быть следующим:
X>
X>     ```text
X>     > whereIs "/bin/ls"
X>     Just (/bin/)
X>     
X>     > whereIs "/bin/cat"
X>     Nothing
X>     ```
X>
X>     _Hint_: Try to write this function as an array comprehension using do notation.

X>     _Подсказка_: Попробуйте написать эту функцию в виде генератора массива, используя do-нотацию.

## Заключение

In this chapter, we covered the basics of recursion in PureScript, as a means of expressing algorithms concisely. We also introduced user-defined infix operators, standard functions on arrays such as maps, filters and folds, and array comprehensions which combine these ideas. Finally, we showed the importance of using tail recursion in order to avoid stack overflow errors, and how to use accumulator parameters to convert functions to tail recursive form.

В этой главе мы рассмотрели основы рекурсии в PureScript, как средство лаконичного выражения алгоритмов. Мы также ознакомились с определяемыми пользователем инфиксными операторами, стандартными функции на массивах, такие как отображения, фильтры и свёртки, а также с генераторами массивов, которые комбинируют данные идеи. Наконец, мы показали важность использования хвостовой рекурсии, чтобы избежать ошибок переполнения стека, и как использовать аккумуляторный параметр для преобразования функции в хвостовую рекурсию.
