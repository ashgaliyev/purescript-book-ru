# Начало обучения

## Цели главы

Цели данной главы: настроить рабочее окружение для разработки на PureScript и написать первую программу.

Нашим первым проектом будет простая библиотека, предоставляющая единственную функцию для вычиcления диагонали прямоугольного треугольника.

## Введение

Вот те инструменты которые мы будем использовать для разработки на PureScript: 

- [`psc`](http://purescript.org) - сам компилятор PureScript.
- [`npm`](http://npmjs.org) - менеджер пакетов для nodejs, который позволит установить оставшую часть инструментов.
- [Pulp](https://github.com/bodil/pulp) - консольное приложение, автоматизирующие множество различных заданий, связанных с управлением PureScript проектов

Оставшаяся часть главы поможет вам установить и настроить эти инструменты. 

## Установка PureScript

Рекомендуемый способ установки компилятора PureScript - это скачать бинарный релиз для вашей платформы [с сайта PureScript](http://purescript.org).

Вам необходимо убедиться в том, что компилятор PureScript доступен в переменной PATH. Попробуйте запустить его в командной строке для проверки:

```text
$ psc
```

Другие способы установки компилятора:

- Использовать популярный менеджер пакетов, такой как NPM или Homebrew (на MacOS).
- Собрать компилятор из исходников. Инструкции можно найти на сайте PureScript.

## Установка инструментов

Необходимо установить [NodeJS](http://nodejs.org/), если у вас не установлена эта платформа. Вместе с ней также установится менеджер пакетов `npm`. Убедитесь в том, что `npm` установлен и доступен в PATH.

Вам также необходимо установить консольную утилиту Pulp и менеджер пакетов Bower, используя `npm`:

```text
$ npm install -g pulp bower
```

This will place the `pulp` and `bower` command line tools on your path. At this point, you will have all the tools needed to create your first PureScript project.

Это команда поместит `pulp` и `bower` для доступа через PATH. Теперь, у вас должны быть все необходимые инструменты для создания вашего первого PureScript проекта.

## Hello, PureScript!

Let's start out simple. We'll use Pulp to compile and run a simple Hello World! program.
Начнем с простого. Мы будем использовать Pulp для компиляции и запуска простой Hello World! программы.

Begin by creating a project in an empty directory, using the `pulp init` command:
Начнем с создания проекта в пустой директории, используя команду `pulp init`:

```text
$ mkdir my-project
$ cd my-project
$ pulp init

* Generating project skeleton in ~/my-project

$ ls

bower.json	src		test
```

Pulp has created two directories, `src` and `test`, and a `bower.json` configuration file for us. The `src` directory will contain our source files, and the `test` directory will contain our tests. We will use the `test` directory later in the book.

Pulp создала для нас две директории - `src` и `test`, а также конфигурационный файл `bower.json`. Директория `src` будет содержать наши исходники, а `test` - наши тесты. Директорию `test` мы будем использовать позднее в книге.

Измените содержимое файла `src/Main.purs` на это:

```haskell
module Main where

import Control.Monad.Eff.Console

main = log "Hello, World!"
```

Этот небольшой пример иллюстрирует несколько ключевых идей:

- Every file begins with a module header. A module name consists of one or more capitalized words separated by dots. In this case, only a single word is used, but `My.First.Module` would be an equally valid module name.
- Каджый файл начинается с заголовка модуля. Имя модуля состоит из одного или нескольких слов, начинающихся с заглавной буквы, и разделенных точками. В нашем случае используется одно слово, но имя `My.First.Module` тоже будет считаться правильным.
- Modules are imported using their full names, including dots to separate the parts of the module name. Here, we import the `Control.Monad.Eff.Console` module, which provides the `log` function.
- Модули импортируются, используя их полные имена, включая точки, разделяющие части имени модуля. Здесь мы импортируем модуль `Control.Monad.Eff.Console`, который предоставляет функцию `log`.
- The `main` program is defined as a function application. In PureScript, function application is indicated with whitespace separating the function name from its arguments.
- Программа `main` определена как применение функции (?). В PureScript применение функции отображается как название, отделенное пробельными символами от её аргументов (?)

Давайте соберем и запустим проект, используя следующую команду:

```text
$ pulp run

* Building project in ~/my-project
* Build successful.
Hello, World!
```

Поздравляю! Вы только что скомпилировали и запустили вашу первую PureScript программу!

## Компилирование для браузера

Pulp can be used to turn our PureScript code into Javascript suitable for use in the web browser, by using the `pulp browserify` command:

Pulp может быть использована для создания из PureScript кода на Javascript, подходящего для браузера, через вызов команды `pulp browserify`:


```text
$ pulp browserify

* Browserifying project in ~/my-project
* Building project in ~/my-project
* Build successful.
* Browserifying...
```

Following this, you should see a large amount of Javascript code printed to the console. This is the output of the [Browserify](http://browserify.org/) tool, applied to a standard PureScript library called the _Prelude_, as well as the code in the `src` directory. This Javascript code can be saved to a file, and included in a HTML document. If you try this, you should see the words "Hello, World!" printed to your browser's console.

Следуя этому, вы должны увидеть в консоли большой объем кода на Javascript. Это результат работы программы [Browserify](http://browserify.org/), которая была применена к стандартной библиотеке PureScript _Prelude_, а так же к коду в директории `src`. Javascript код может быть сохранен в файл и включен в HTML документ. Если вы это сделаете, то увидите фразу "Hello, World!", выведенную в консоли браузера.

## Удаление неиспользуемого кода

Pulp provides an alternative command, `pulp build`, which can be used with the `-O` option to apply _dead code elimination_, which removes unnecessary Javascript from the output. The result is much smaller:

Pulp предоставляет альтернативную команду - `pulp build`, которая может быть использована с опцией `-O` для удаление неиспользуемого кода (dead code elimination) из вывода. Результирующий код будет гораздо меньше:

```text
$ pulp build -O --to output.js

* Building project in ~/my-project
* Build successful.
* Bundling Javascript...
* Bundled.
```

Again, the generated code can be used in a HTML document. If you open `output.js`, you should see a few compiled modules which look like this:

И опять, генерируемый код может быть использован в HTML документе. Если вы откроете `output.js` вы должны увидеть несколько скомпилированных модулей, которые выглядят примерно так:

```javascript
(function(exports) {
  "use strict";

  var Control_Monad_Eff_Console = PS["Control.Monad.Eff.Console"];

  var main = Control_Monad_Eff_Console.log("Hello, World!");
  exports["main"] = main;
})(PS["Main"] = PS["Main"] || {});
```

This illustrates a few points about the way the PureScript compiler generates Javascript code:
Это иллюстрирует несколько моментов о том, как PureScript компилятор генерирует Javascript код:

- Every module gets turned into an object, created by a wrapper function, which contains the module's exported members.
- Каждый модуль становится объектом, оборачиваемый в функцию, который содержит экспортируемые члены модуля.
- PureScript tries to preserve the names of variables wherever possible
- PureScript пытается сохранить имена переменных где это возможно.
- Function applications in PureScript get turned into function applications in JavaScript.
- Применение функций в PureScript преобразовываются в применение функций в JavaScript(?)
- The main method is run after all modules have been defined, and is generated as a simple method call with no arguments.
- Метод main запускается после того, как все модули будут определены, и генерируется в виде простого вызова метода без аргументов.
- PureScript code does not rely on any runtime libraries. All of the code that is generated by the compiler originated in a PureScript module somewhere which your code depended on.
- код PureScript не зависит ни от какой runtime библиотеки. Весь код, сгенерированный компилятором порожден где-то в модуле PureScript, от которого зависит ваш код (?)

These points are important, since they mean that PureScript generates simple, understandable code. In fact, the code generation process in general is quite a shallow transformation. It takes relatively little understanding of the language to predict what JavaScript code will be generated for a particular input.

Эти важные моменты показывают то, что PureScript генерирует простой и понятный код. Фактически, процесс кодогенерации представляет собой довольно небольшое преобразование. Требуется сравнительно небольшое понимание языка, чтобы предугадать какой Javascript код будет сгенерирован на определенном входном коде.

## Компилирование модулей CommonJS

Pulp can also be used to generate CommonJS modules from PureScript code. This can be useful when using NodeJS, or just when developing a larger project which uses CommonJS modules to break code into smaller components.

Pulp может быть так-же использован для генерации модулей CommonJS из PureScript кода. Это может быть полезно во время использования NodeJS, или просто для разработки большого проекта, который использует модули CommonJS, для разбиения кода на меньшие компоненты.

To build CommonJS modules, use the `pulp build` command (without the `-O` option):

Чтобы создать модули CommonJS, используйте команду `pulp build` (без опции `-O`):

```text
$ pulp build

* Building project in ~/my-project
* Build successful.
```

The generated modules will be placed in the `output` directory by default. Each PureScript module will be compiled to its own CommonJS module, in its own subdirectory.

Сгенерированный код по-умолчанию будет помещен в папку `output`. Каждый модуль PureScript будет скомпилирован в свой собственный модуль CommonJS и помещен в свою собственную поддиректорию.

## Отслеживание зависимостей с Bower

To write the `diagonal` function (the goal of this chapter), we will need to be able to compute square roots. The `purescript-math` package contains type definitions for functions defined on the JavaScript `Math` object, so let's install it:

Чтобы написать функцию `diagonal` (цель данной главы), нам необходимо иметь возможность вычислять квадратные корни. Пакет `purescript-math` содержит определения типов для функций, определенных в JavaScript объекте `Math`, поэтому, давайте установим его:

```text
$ bower install purescript-math --save
```

The `--save` option causes the dependency to be added to the `bower.json` configuration file.

Опция `--save` добавляет зависимость в конфигурационный файл `bower.json`

The `purescript-math` library sources should now be available in the `bower_components` subdirectory, and will be included when you compile your project.

Исходники библиотеки `purescript-math` должны быть теперь доступны в поддиректории `bower_components`, и будут подключены, когда вы будете компилировать проект.

## Вычисление диагоналей

Let's write the `diagonal` function, which will be an example of using a function from an external library.

Давайте напишем функцию `diagonal`, которая будет примером использования функции из внешней библиотеки.

First, import the `Math` module by adding the following line at the top of the `src/Main.purs` file:

Во-первых, импортируем модуль `Math`, добавив следующие строки в начало файла `src/Main.purs`:

```haskell
import Math (sqrt)
```

It's also necessary to import the `Prelude` module, which defines very basic operations such as numeric addition and multiplication:

Также необходимо добавить модуль `Prelude`, который определяет очень базовые операции, такие как числовое сложение и умножение:

```haskell
import Prelude
```

Now define the `diagonal` function as follows:

Теперь определим функцию `diagonal` следущим образом: 

```haskell
diagonal w h = sqrt (w * w + h * h)
```

Note that there is no need to define a type for our function. The compiler is able to infer that `diagonal` is a function which takes two numbers and returns a number. In general, however, it is a good practice to provide type annotations as a form of documentation. 

Обратим внимание, что здесь нет необходимости определять тип функции. Компилятор способен сделать вывод что `diagonal` это функция, которая берёт два числа и возвращает число. В целом, однако, это хорошая практика - предоставлять аннотации типами, как форму документации. (?)

Let's also modify the `main` function to use the new `diagonal` function:

Давайте также изменим функцию `main` для использования новой функции `diagonal`:

```haskell
main = logShow (diagonal 3.0 4.0)
```

Now compile and run the project again, using `pulp run`:

Теперь опять скомпилируем и запустим проект, используя `pulp run`:

```text
$ pulp run

* Building project in ~/my-project
* Build successful.
5.0
```

## Тестирование кода с использованием интерактивного режима

The PureScript compiler also ships with an interactive REPL called PSCi. This can be very useful for testing your code, and experimenting with new ideas. Let's use PSCi to test the `diagonal` function.

Компилятор PureScript поставляется также вместе с интерактивным REPL под названием PSCi. Это может быть очень полезно для тестирование кода и для экспериментирования с новыми идеями. Давате используем PSCi чтобы проверить функцию `diagonal`:

Pulp can load source modules into PSCi automatically, via the `pulp psci` command:

Pulp может загрузить исходники модулей в psci автоматически, через команду `pulp psci`:

```text
$ pulp psci
>
```

You can type `:?` to see a list of commands:

Вы можете увидеть список команд, напечатав `:?`:

```text
> :?
The following commands are available:

    :?                        Show this help menu
    :quit                     Quit PSCi
    :reset                    Reset
    :browse      <module>     Browse <module>
    :type        <expr>       Show the type of <expr>
    :kind        <type>       Show the kind of <type>
    :show        import       Show imported modules
    :show        loaded       Show loaded modules
    :paste       paste        Enter multiple lines, terminated by ^D
```

By pressing the Tab key, you should be able to see a list of all functions available in your own code, as well as any Bower dependencies and the Prelude modules.

Нажав на Tab, вы должны будете увидеть список всех доступных функций в вашем коде, во всех зависимостях из Bower, а также в модулях Prelude:

Start by importing the `Prelude` module:

Начнем с импортирования модуля `Prelude`:

```text
> import Prelude
```

Try evaluating a few expressions now:

Теперь попробуем вычислить несколько выражений:

```text
> 1 + 2
3

> "Hello, " <> "World!"
"Hello, World!"
```

Let's try out our new `diagonal` function in PSCi:

Давайте попробуем нашу новую функцию `diagonal` в PSCi.


```text
> import Main
> diagonal 5.0 12.0

13.0
```

You can also use PSCi to define functions:

Вы можете также использовать PSCi для определения функции:

```text
> let double x = x * 2

> double 10
20
```

Don't worry if the syntax of these examples is unclear right now - it will make more sense as you read through the book.

Не беспокойтесь, если синтаксис этих примеров вам сейчас не ясен - он приобретет больше смысла по ходу чтения книги (?)

Finally, you can check the type of an expression by using the `:type` command:

Наконец, вы можете проверить тип выражения, используя команду `:type`:

```text
> :type true
Boolean

> :type [1, 2, 3]
Array Int
```

Try out the interactive mode now. If you get stuck at any point, simply use the Reset command `:reset` to unload any modules which may be compiled in memory. 

Теперь попробуйте интерактивный режим. Если вы где-то застряли, просто воспользуйтесь командой `:reset` для выгрузки всех модулей, скомпилированных в памяти.

X> ## Exercises
X>
X> 1. (Easy) Use the `Math.pi` constant to write a function `circleArea` which computes the area of a circle with a given radius. Test your function using PSCi (_Hint_: don't forget to import `pi` by modifying the `import Math` statement).
X> 1. (Medium) Use `bower install` to install the `purescript-globals` package as a dependency. Test out its functions in PSCi (_Hint_: you can use the `:browse` command in PSCi to browse the contents of a module).

X> ## Упражнения
X>
X> 1. (Легко) Используя константу `Math.pi`, напишите функцию `circleArea`, вычисляющаю площадь круга по заданному радиусу. Проверьте функцию, используя PSCi (_Подсказка_: не забудьте импортировать `pi`, изменив инструкцию `import Math`)
X> 1. (Средне) Используя `bower install` установите пакет `purescript-globals` в качестве зависимости. Проверьте его функции в PSCi (_Подсказка_: вы можете воспользоваться командой `:browse` в PSCi, чтобы просмотреть содержимое модуля)

## Заключение

In this chapter, we set up a simple PureScript project using the Pulp tool.

В этой главе мы создали простой PureScript проект использовав инструмент Pulp

We also wrote our first PureScript function, and a JavaScript program which could be compiled and executed either in the browser or in NodeJS.

Мы также написали нашу первую PureScript функцию, а также JavaScript программу, которая может быть скомпилирована и запущена, и в браузере, и на NodeJS

We will use this development setup in the following chapters to compile, debug and test our code, so you should make sure that you are comfortable with the tools and techniques involved.

Мы будем использовать эту рабочую конфигурацию в последующих главах для компиляции, отладки и запуска нашего кода, поэтому вы должны убедиться, что чувствуете себя комфортно с используемыми инструментами и техниками.
