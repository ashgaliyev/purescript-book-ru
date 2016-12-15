# Введение 

## Функциональный JavaScript

Functional programming techniques have been making appearances in JavaScript for some time now:
Функциональное программирование стало появляться в JavaScript какое-то время назад:

- Libraries such as [UnderscoreJS](http://underscorejs.org) allow the developer to leverage tried-and-trusted functions such as `map`, `filter` and `reduce` to create larger programs from smaller programs by composition:
- Такие библиотеки как [UnderscoreJS](http://underscorejs.org) позволили разработчику использовать проверенные и надежные функции, такие как `map`, `filter` и `reduce` для создания бОльших программ из композиции таких функций:

    ```javascript
    var sumOfPrimes =
        _.chain(_.range(1000))
         .filter(isPrime)
         .reduce(function(x, y) {
             return x + y;
         })
         .value();
    ```

- Asynchronous programming in NodeJS leans heavily on functions as first-class values to define callbacks.
- Асинхронное программирование в NodeJS опирается на функции высшего порядка для создания колбэков

    ```javascript
    require('fs').readFile(sourceFile, function (error, data) {
      if (!error) {
        require('fs').writeFile(destFile, data, function (error) {
          if (!error) {
            console.log("File copied");
          }
        });
      }
    });
    ```

- Libraries such as [React](http://facebook.github.io/react/) and [virtual-dom](https://github.com/Matt-Esch/virtual-dom) model views as pure functions of application state.
- Библиотеки такие как [React](http://facebook.github.io/react/) и [virtual-dom](https://github.com/Matt-Esch/virtual-dom) моделируют представления как чистые функции, отображающие состояние приложения.

Functions enable a simple form of abstraction which can yield great productivity gains. However, functional programming in JavaScript has its own disadvantages: JavaScript is verbose, untyped, and lacks powerful forms of abstraction. Unrestricted JavaScript code also makes equational reasoning very difficult.

Функции разрешают простые формы абстракции, которые дают большой прирост продуктивности. Однако, функциональное программирование в JavaScript имеет свои недостатки: JavaScript многословен, нетипизирован, не имеет мощних форм абстракции. Неограниченный JavaScript код делает трудным разбор вычислений

PureScript is a programming language which aims to address these issues. It features lightweight syntax, which allows us to write very expressive code which is still clear and readable. It uses a rich type system to support powerful abstractions. It also generates fast, understandable code, which is important when interoperating with JavaScript, or other languages which compile to JavaScript. All in all, I hope to convince you that PureScript strikes a very practical balance between the theoretical power of purely functional programming, and the fast-and-loose programming style of JavaScript.

PureScript это язык программирования нацеленный на решение этих проблем. Он имеет легковесный синтаксис, позволяющий писать очень выразительный, но в тоже время чистый и читаемый код. Он имеет богатую систему типов для создания мощных абстракций. Он также генерирует быстрый, понятный код, что важно при взаимодействии с JavaScript или другими языками, компилируемыми в JavaScript. В общем, я надеюсь убедить вас в том, что PureScript ударяет в очень практичный баланс между мощью чистого функционального программирования и fast-and-loose стиля программирования на JavaScript.  

## Types and Type Inference
## Типы и вывод типов


The debate over statically typed languages versus dynamically typed languages is well-documented. PureScript is a _statically typed_ language, meaning that a correct program can be given a _type_ by the compiler which indicates its behavior. Conversely, programs which cannot be given a type are _incorrect programs_, and will be rejected by the compiler. In PureScript, unlike in dynamically typed languages, types exist only at _compile-time_, and have no representation at runtime.

Дебат по поводу статически типизированных языков против динамически типизированных хорошо задокументирован. PureScript - это _статически типизированный_ язык, означающее то, что корректной программе присваивается _тип_ компилятором, указывающий её поведение. Напротив, программы, которым не может быть присвоен тип являются _некорректными программами_, и будут отвергнуты компилятором. В PureScript, в отличие от динамических языков, типы существуют только на _этапе компиляции_, и не существуют на этапе исполнения. 

It is important to note that in many ways, the types in PureScript are unlike the types that you might have seen in other languages like Java or C#. While they serve the same purpose at a high level, the types in PureScript are inspired by languages like ML and Haskell. PureScript's types are expressive, allowing the developer to assert strong claims about their programs. Most importantly, PureScript's type system supports _type inference_ - it requires far fewer explicit type annotations than other languages, making the type system a _tool_ rather than a hindrance. As a simple example, the following code defines a _number_, but there is no mention of the `Number` type anywhere in the code:

Важно отметить что во многих случаях типы в PureScript отличаются от типов, которые вы могли бы видеть в других языках программирования, таких как Java или C#. В то время, как они служат одной и той же цели на высоком уровне, типы в PureScript вдохновлены такими языками, как ML и Haskell. Типы PureScript выразительные, что позволяет разработчику делать сильные утверждения о работе его программы. Самое важное - система типов PureScript поддерживает _вывод типов_ - это требует намного меньше явного аннотирования типами, чем в других языках, делая систему типов _инструментом_ а не помехой. В качестве примера - следующий код определяет _число_, но здесь нигде не упоминается тип `Number` в коде:  


```haskell
iAmANumber =
  let square x = x * x
  in square 42.0
```

A more involved example shows that type-correctness can be confirmed without type annotations, even when there exist types which are _unknown to the compiler_:
Более сложный пример, показывающий что типо-корректность может быть доказана без аннотации типами, даже когда есть типы которые _неизвестны компилятору_:

```haskell
iterate f 0 x = x
iterate f n x = iterate f (n - 1) (f x)
```

Here, the type of `x` is unknown, but the compiler can still verify that `iterate` obeys the rules of the type system, no matter what type `x` might have.
Тип `x` неизвестен, но компилятор всё еще может подтвердить подчинение `iterate` правилам системы типов, в не зависимости от того, какой тип может принять `x`.

In this book, I will try to convince you (or reaffirm your belief) that static types are not only a means of gaining confidence in the correctness of your programs, but also an aid to development in their own right. Refactoring a large body of code in JavaScript can be difficult when using any but the simplest of abstractions, but an expressive type system together with a type checker can even make refactoring into an enjoyable, interactive experience.

В этой книге я постараюсь убедить вас (вновь) что статические типы это не только стредство достижения корректности ваших программ, но а также помощь для разработки в своём праве(хз как сказать еще). Рефакторинг большого куска кода в JavaScript может быть тяжелым занятием, когда используешь какие угодно простые абстракции, но выразетильная система типов вкупе с тайп-чекером может превратить рефакторинг в весёлое, интерарактивный опыт.

In addition, the safety net provided by a type system enables more advanced forms of abstraction. In fact, PureScript provides a powerful form of abstraction which is fundamentally type-driven: type classes, made popular in the functional programming language Haskell.

В дополнение, сеть безопасности предоставляемая системой типов допускает более продвинутые формы абстракциии. Фактически, PureScript предоставляет мощные формы абстракции основанные на типах: классы типов, популярные в функциональном языке программирования Haskell 

## Polyglot Web Programming
## Полиглотное Веб-программирование 

Functional programming has its success stories - applications where it has been particularly successful: data analysis, parsing, compiler implementation, generic programming, parallelism, to name a few.
Функциональное программирование имеет свои истории успешного применения - вот приложения, где оно успешно применялось: анализ данных, парсинг, создание компиляторов, обобщенное программирование, параллелизм и т.д.

It would be possible to practice end-to-end application development in a functional language like PureScript. PureScript provides the ability to import existing JavaScript code, by providing types for its values and functions, and then to use those functions in regular PureScript code. We'll see this approach later in the book.
Можно создать приложение на PureScript от начала и до конца (незнаю как правильно). PureScript предоставляет возможность использовать существующий JavaScript код, через указание типов для его значений и функции, чтобы использовать эти функции в обычном PureScript коде. Мы разберем этот подход далее в книге.

However, one of PureScript's strengths is its interoperability with other languages which target JavaScript. Another approach would be to use PureScript for a subset of your application's development, and to use one or more other languages to write the rest of the JavaScript.
Однако, одна из сильных сторон PureScript это взаимодействие с другими языками, компилируемыми в JavaScript. Другой подход будет использование PureScript для разработки определенных частей приложения, и использование одного или более языков для написания оставшейся части JavaScript

Here are some examples:
Несколько примеров:

- Core logic written in PureScript, with the user interface written in JavaScript.
- Основная логика написана на PureScript, интерфейс - на JavaScript

- Application written in JavaScript or another compile-to-JS language, with tests written in PureScript.
- Приложение написано на JavaScript или другом языке, компилируемом в JS, тесты написаны на PureScript

- PureScript used to automate user interface tests for an existing application.
- PureScript использован для автоматизирование тестов проверки интерфейса для существующего приложения

In this book, we'll focus on solving small problems with PureScript. The solutions could be integrated into a larger application, but we will also look at how to call PureScript code from JavaScript, and vice versa.
В этой книге я сфокусируюсь на решении небольших проблем при помощи PureScript. Решения могут быть интегрированы в большое приложение, но мы также посмотрим на то, как вызвать PureScript код из JavaScript и так далее.

## Prerequisites
## Необходимые требования

The software requirements for this book are minimal: the first chapter will guide you through setting up a development environment from scratch, and the tools we will use are available in the standard repositories of most modern operating systems.
Требования к ПО в данной книге минимальны: первая глава будет посвящена настройке среды разработки с нуля, и те инструменты, которые мы будем использовать доступны в стандартных репозиториях большинства современных операционных систем.

The PureScript compiler itself can be downloaded as a binary distribution, or built from source on any system running an up-to-date installation of the GHC Haskell compiler, and we will walk through this process in the next chapter.
Компилятор PureScript может быть скачан в виде скомпилированного пакета или скомпилирован из исходников на любой системе, где установлена последняя версия компилятора GHC. Мы пройдем этот процесс в следующей главе

The code in this version of the book is compatible with versions `0.10.*` of
the PureScript compiler.
Код в данной версии книги совместим с версией `0.10.*` компилятора PureScript 

## About You
## О Вас

I will assume that you are familiar with the basics of JavaScript. Any prior familiarity with common tools from the JavaScript ecosystem, such as NPM and Gulp, will be beneficial if you wish to customize the standard setup to your own needs, but such knowledge is not necessary.
Я буду считать что вы знакомы с JavaScript на базовом уровне. Любое знакомство с общими инструментами из экосистемы JavaScript, такими как NPM и Gulp, будет полезно, если вы хотите настроить стандартную установку под свои нужды. Но знание не обязательно. 

No prior knowledge of functional programming is required, but it certainly won't hurt. New ideas will be accompanied by practical examples, so you should be able to form an intuition for the concepts from functional programming that we will use.
Предварительных знаний из функционального программирование не требуется, но это не повредит. Новые идеи будут сопровождены практическими примерами, так что у вас должна будет сформирована интуиция для тех концепций из функционального программирования, что мы будем использовать. 

Readers who are familiar with the Haskell programming language will recognize a lot of the ideas and syntax presented in this book, because PureScript is heavily influenced by Haskell. However, those readers should understand that there are a number of important differences between PureScript and Haskell. It is not necessarily always appropriate to try to apply ideas from one language in the other, although many of the concepts presented here will have some interpretation in Haskell.
Читатели, уже знакомые с языком программирования Haskell увидят много знакомых идей, а также синтакс в данной книге, так как Haskell сильно повлиял на PureScript. Однако, эти читатели должны понять, что тут есть некоторое число важных отличий между PureScript и Haskell. Не всегда целесообразно применять идеи из одного языка в другом, хотя многие концепции, представленные здесь, будут иметь некоторую интерпретацию в Haskell

## How to Read This Book
## Как читать эту книгу

The chapters in this book are largely self contained. A beginner with little functional programming experience would be well-advised, however, to work through the chapters in order. The first few chapters lay the groundwork required to understand the material later on in the book. A reader who is comfortable with the ideas of functional programming (especially one with experience in a strongly-typed language like ML or Haskell) will probably be able to gain a general understanding of the code in the later chapters of the book without reading the preceding chapters.

Главы в этой книге по большей части самодостаточны. Новичек с небольшим опытом функционального программирования будет чувствовать себя комфортно, проходя главы по-порядку. Первые несколько глав объясняют основы, необходимые для понимания дальнейшего материала в книге. Читатель, знакомый с идеями функционального программирования (особенно, имеющий опыт работы со строго-типизированными языками такими, как ML или Haskell) получит общее представление о коде в поздних главах книги, не читая начальные.

Each chapter will focus on a single practical example, providing the motivation for any new ideas introduced. Code for each chapter are available from the book's [GitHub repository](https://github.com/paf31/purescript-book). Some chapters will include code snippets taken from the chapter's source code, but for a full understanding, you should read the source code from the repository alongside the material from the book. Longer sections will contain shorter snippets which you can execute in the interactive mode PSCi to test your understanding.

Каждая глава сфокусирована на единственном примере, мотивирующая к знакомству с новыми идеями. Код каждой главы доступен в исходных кодах книги в репозитории на [GitHub](http://github.com/paf31/purescript-book). Некоторые главы включают сниппеты, взятые из исходников, но для полного понимания, вам необходимо читать исходники с примерами бок о бок с материалом из книги. Длинные разделы будут содержать небольшие примеры кода, которые вы можете запустить в интерактивном режиме PSCi для проверки своего понимания.

Code samples will appear in a monospaced font, as follows:
Примеры кода будут представлены в моноширинном шрифте, как тут:

```haskell
module Example where

import Control.Monad.Eff.Console (log)

main = log "Hello, World!"
```

Commands which should be typed at the command line will be preceded by a dollar symbol:
Команды, которые должны быть запущены в консоли будут со знаком доллара в начале:

```text
$ pulp build
```

Usually, these commands will be tailored to Linux/Mac OS users, so Windows users may need to make small changes such as modifying the file separator, or replacing shell built-ins with their Windows equivalents.
Эти команды обычно указываются для пользоватлей Linux/MacOS, поэтому пользователям Windows нужно делать небольшие изменения, например - поменять разделитель для директорий или заменить встроенные программы shell на их Windows эквиваленты.

Commands which should be typed at the PSCi interactive mode prompt will be preceded by an angle bracket:
Команды, которые печатаются в интерактивном режиме PSCi будут обозначены угловой скобкой:

```text
> 1 + 2
3
```

Each chapter will contain exercises, labelled with their difficulty level. It is strongly recommended that you attempt the exercises in each chapter to fully understand the material.
Каждая глава будет содержать упражнения, для которых будут выставлены уровни сложности. Строго рекомендуется пытаться решить все упражнения в каждой главе для полного понимания материала.

This book aims to provide an introduction to the PureScript language for beginners, but it is not the sort of book that provides a list of template solutions to problems. For beginners, this book should be a fun challenge, and you will get the most benefit if you read the material, attempt the exercises, and most importantly of all, try to write some code of your own.
Цель данной книги - предоставить введение в язык PureScript для новичков, но это не тот сорт книги, которые дают типичные решения определенных проблем. Для новичков эта книга должна быть увлекательным `вызовом`, и вы получите наибольшую выгоду если читаете материал, решаете упражения и важнее всего - пытаетесь сделать что-нибудь своё. 

## Getting Help
## Получение помощи

If you get stuck at any point, there are a number of resources available online for learning PureScript:
Если вы застряли на любой точке, существуют множество online-ресурсов для обучения PureScript:

- The PureScript IRC channel is a great place to chat about issues you may be having. Point your IRC client at irc.freenode.net, and connect to the #purescript channel.
- IRC канал PureScript - это отличное место для обсуждения проблем, с которыми вы столкнулись. Настройте свой IRC клиент на irc.freenode.net и подключитесь к каналу #purescript

- The [PureScript website](http://purescript.org) contains links to several learning resources, including code samples, videos and other resources for beginners.
- [PureScript website](http://purescript.org) содержит ссылки на некоторые обучащие ресурсы, включающие примеры кода, видео и другие материалы для новичков.

- The [PureScript wiki](http://wiki.purescript.org) collects articles and examples on a wide variety of topics, written by PureScript developers and users.
- На [PureScript вики](http://wiki.purescript.org) собираются статьи и примеры, написанные разработчиками и пользователями PureScript

- [Try PureScript!](http://try.purescript.org) is a website which allows users to compile PureScript code in the web browser, and contains several simple examples of code.
- [Try PureScript!](http://try.purescript.org) - вебсайт, который позволяет пользователям скомпилировать PureScript код прямо в браузере. Содержит несколько простых примеров кода 


- [Pursuit](http://pursuit.purescript.org) is a searchable database of PureScript types and functions.
- [Pursuit](http://pursuit.purescript.org) база данных с функцией поиска по типам и функциям PureScript.

If you prefer to learn by reading examples, the `purescript`, `purescript-node` and `purescript-contrib` GitHub organizations contain plenty of examples of PureScript code.

## About the Author

I am the original developer of the PureScript compiler. I'm based in Los Angeles, California, and started programming at an early age in BASIC on an 8-bit personal computer, the Amstrad CPC. Since then I have worked professionally in a variety of programming languages (including Java, Scala, C#, F#, Haskell and PureScript).

Not long into my professional career, I began to appreciate functional programming and its connections with mathematics, and enjoyed learning functional concepts using the Haskell programming language.

I started working on the PureScript compiler in response to my experience with JavaScript. I found myself using functional programming techniques that I had picked up in languages like Haskell, but wanted a more principled environment in which to apply them. Solutions at the time included various attempts to compile Haskell to JavaScript while preserving its semantics (Fay, Haste, GHCJS), but I was interested to see how successful I could be by approaching the problem from the other side - attempting to keep the semantics of JavaScript, while enjoying the syntax and type system of a language like Haskell.

I maintain [a blog](http://blog.functorial.com), and can be [reached on Twitter](http://twitter.com/paf31).

## Acknowledgements

I would like to thank the many contributors who helped PureScript to reach its current state. Without the huge collective effort which has been made on the compiler, tools, libraries, documentation and tests, the project would certainly have failed.  

The PureScript logo which appears on the cover of this book was created by Gareth Hughes, and is gratefully reused here under the terms of the [Creative Commons Attribution 4.0 license](https://creativecommons.org/licenses/by/4.0/).

Finally, I would like to thank everyone who has given me feedback and corrections on the contents of this book.
