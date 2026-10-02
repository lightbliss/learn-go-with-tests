# Hello, World

**[Весь код из этого раздела находится здесь](https://github.com/quii/learn-go-with-tests/tree/main/hello-world)**

По традиции для первой программы на новом языке  выбирают [Hello, World](https://en.m.wikipedia.org/wiki/%22Hello,_World!%22_program).

- Создайте папку в любом удобном месте
- Создайте в папке новый файл и назовите его `hello.go` и перенесите в него следующий код

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world")
}
```

Для запуска используйте команду `go run hello.go`.

## Как это работает

Когда вы пишите программу на языке Go, у вас будет пакет `main` с функцией `main` внутри. Пакеты - это способ сгруппировать общий по смыслу код вместе.

Ключевое слово `func` определяет функцию с именем и телом.

С помощью `import "fmt"` мы добавляем пакет, который содержит функцию `Println`, которую мы используем для печати.

## Как протестировать

Как это протестировать? Хорошей практикой является отделение «доменного» кода от внешнего мира (побочных эффектов). fmt.Println — это побочный эффект (вывод в stdout), а строка, которую мы передаём, — это наш домен.

Итак, давайте разделим эти зоны ответственности, чтобы упростить тестирование.

```go
package main

import "fmt"

func Hello() string {
	return "Hello, world"
}

func main() {
	fmt.Println(Hello())
}
```

Мы создали новую функцию с помощью `func`, но на этот раз добавили в определение ещё одно ключевое слово — `string`. Это означает, что данная функция возвращает `строку`.

Теперь создайте новый файл с именем `hello_test.go`, в котором мы напишем тест для нашей функции `Hello`.

```go
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello()
	want := "Hello, world"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

Модули Go?

Следующий шаг — запустить тесты. Введите `go test` в терминале. Если тесты проходят, то вы, вероятно, используете более раннюю версию Go. Однако, если вы используете Go 1.16 или более позднюю версию, тесты, скорее всего, не запустятся. Вместо этого вы увидите в терминале сообщение об ошибке примерно такого вида:

```shell
$ go test
go: cannot find main module; see 'go help modules'
```

В чём проблема? Одним словом — [модули](https://blog.golang.org/go116-module-changes). К счастью, проблема легко исправляется. Введите `go mod init example.com/hello` в терминале. Это создаст новый файл со следующим содержимым:

```
module example.com/hello

go 1.16
```

Этот файл сообщает инструментам `go` основную информацию о вашем коде. Если бы вы планировали распространять своё приложение, вы бы указали, где код доступен для загрузки, а также информацию о зависимостях. Имя модуля, example\.com\/hello, обычно ссылается на URL, по которому модуль можно найти и загрузить. Для совместимости с инструментами, которые мы вскоре начнём использовать, убедитесь, что в имени вашего модуля где-нибудь есть точка — как точка в .com в example\.com\/hello. Пока что ваш файл модуля минимален, и вы можете оставить его таким. Чтобы узнать больше о модулях, [можно обратиться к справочнику в документации Golang](https://golang.org/doc/modules/gomod-ref). Теперь мы можем вернуться к тестированию и изучению Go, поскольку тесты должны запускаться даже на Go 1.16.

В последующих главах вам нужно будет выполнять `go mod init SOMENAME` в каждой новой папке перед запуском таких команд, как `go test` или `go build`.

## Возвращаемся к тестированию

Запустите `go test` в терминале. Тест должен был пройти! Чтобы убедиться, попробуйте намеренно сломать тест, изменив строку `want`.

Обратите внимание: вам не пришлось выбирать между несколькими фреймворками для тестирования, а затем разбираться с их установкой. Всё необходимое встроено в сам язык, а синтаксис совпадает с остальным кодом, который вы будете писать.

### Написание тестов

Написание теста — это почти то же самое, что и написание функции, но с несколькими правилами:

* Он должен находиться в файле с именем вида `xxx_test.go`
* Имя тестовой функции должно начинаться со слова `Test`
* Тестовая функция принимает только один аргумент — `t *testing.T`
* Чтобы использовать тип `*testing.T`, нужно импортировать пакет `"testing"`, как мы делали с `fmt` в другом файле

Пока достаточно знать, что ваш t типа *testing.T — это ваша «точка входа» во фреймворк тестирования, позволяющая делать такие вещи, как t.Fail(), когда вы хотите провалить тест.

Мы рассмотрели несколько новых тем:

#### `if`
Условные операторы `if` в Go очень похожи на аналогичные конструкции в других языках программирования.

#### Объявление переменных

Мы объявляем некоторые переменные с помощью синтаксиса `varName := value`, что позволяет переиспользовать значения в тесте для улучшения читаемости.

#### `t.Errorf`
Мы вызываем _метод_ `Errorf` у нашего `t`, который выведет сообщение и провалит тест. Буква `f` означает «format» (форматирование) — это позволяет строить строку с подстановкой значений в плейсхолдеры `%q`. Когда вы провалите тест, станет ясно, как это работает.

Подробнее о строках-плейсхолдерах можно прочитать в [документации пакета fmt](https://pkg.go.dev/fmt#hdr-Printing). В тестах `%q` очень удобен, так как оборачивает ваши значения в двойные кавычки.

Позже мы разберём разницу между методами и функциями.

### Документация Go

Ещё одна удобная возможность Go — это документация. Мы только что видели документацию пакета fmt на официальном сайте для просмотра пакетов, а Go также предоставляет способы быстро получать документацию офлайн.

В Go есть встроенный инструмент doc, который позволяет изучать любой пакет, установленный в вашей системе, или модуль, над которым вы работаете сейчас. Чтобы посмотреть ту же документацию по глаголам форматирования:

```
$ go doc fmt
package fmt // import "fmt"

Package fmt implements formatted I/O with functions analogous to C's printf and
scanf. The format 'verbs' are derived from C's but are simpler.

# Printing

The verbs:

General:

    %v	the value in a default format
    	when printing structs, the plus flag (%+v) adds field names
    %#v	a Go-syntax representation of the value
    %T	a Go-syntax representation of the type of the value
    %%	a literal percent sign; consumes no value
...
```

Второй инструмент Go для просмотра документации — команда `pkgsite`, на которой работает официальный сайт для просмотра пакетов Go. Установить pkgsite можно командой `go install golang.org/x/pkgsite/cmd/pkgsite@latest`, а затем запустить её с помощью `pkgsite -open .`. Команда install в Go скачает исходные файлы из этого репозитория и соберёт из них исполняемый бинарный файл. При установке Go по умолчанию этот исполняемый файл будет находиться в `$HOME/go/bin` для Linux и macOS и в `%USERPROFILE%\go\bin` для Windows. Если вы ещё не добавили эти пути в переменную $PATH, возможно, стоит это сделать, чтобы упростить запуск команд, установленных через go.

Подавляющая часть стандартной библиотеки имеет превосходную документацию с примерами. Перейти по адресу [http://localhost:8080/testing](http://localhost:8080/testing) будет полезно, чтобы увидеть, что вам доступно.

### Привет, ТЫ
Теперь, когда у нас есть тест, мы можем безопасно дорабатывать наш код.

В прошлом примере мы написали тест _после_ того, как код был написан, — чтобы вы могли увидеть пример того, как писать тест и объявлять функцию. С этого момента мы будем _писать тесты первыми_.

Наше следующее требование — дать возможность указывать получателя приветствия.

Давайте начнём с того, что зафиксируем эти требования в тесте. Это базовый подход разработки через тестирование (TDD), и он позволяет убедиться, что наш тест _действительно_ проверяет то, что мы хотим. Когда вы пишете тесты задним числом, есть риск, что ваш тест будет продолжать проходить, даже если код работает не так, как задумано.

```go
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello("Chris")
	want := "Hello, Chris"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

Теперь запустите `go test` — вы должны получить ошибку компиляции.

```text
./hello_test.go:6:18: too many arguments in call to Hello
    have (string)
    want ()
```

При работе со статически типизированным языком, таким как Go, важно _слушать компилятор_. Компилятор понимает, как ваш код должен стыковаться и работать, — так что вам не нужно делать это самому.

В данном случае компилятор сообщает вам, что нужно сделать, чтобы двигаться дальше. Нам нужно изменить нашу функцию `Hello`, чтобы она принимала аргумент.

Отредактируйте функцию `Hello`, чтобы она принимала аргумент типа `string`.

```go
func Hello(name string) string {
	return "Hello, world"
}
```

Если вы попробуете снова запустить тесты, ваш `hello.go` не скомпилируется, потому что вы не передаёте аргумент. Передайте "world", чтобы он скомпилировался.

```go
func main() {
	fmt.Println(Hello("world"))
}
```

Теперь, когда вы запустите тесты, вы должны увидеть что-то вроде

```text
hello_test.go:10: got 'Hello, world' want 'Hello, Chris''
```

Наконец-то у нас есть компилирующаяся программа, но она не соответствует нашим требованиям согласно тесту.

Давайте заставим тест пройти, использовав аргумент с именем и сконкатенировав его с `Hello,`.

```go
func Hello(name string) string {
	return "Hello, " + name
}
```

Когда вы запустите тесты, они должны пройти. Обычно в рамках цикла TDD теперь нам следует заняться _рефакторингом_.

### Замечание о системе контроля версий

На данном этапе, если вы используете систему контроля версий \(а вы должны!\), я бы сделал `commit` кода в текущем виде. У нас есть работающее программное обеспечение, подкреплённое тестом.

Но я бы _не_ стал делать push в main, потому что планирую заняться рефакторингом дальше. Приятно сделать коммит на этом этапе на случай, если вы каким-то образом запутаетесь в процессе рефакторинга — вы всегда можете вернуться к рабочей версии.

Рефакторить здесь особо нечего, но мы можем ввести ещё одну возможность языка — _константы_.

### Константы
Константы определяются так:

```go
const englishHelloPrefix = "Hello, "
```

Теперь мы можем отрефакторить наш код

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
	return englishHelloPrefix + name
}
```

After refactoring, re-run your tests to make sure you haven't broken anything.

It's worth thinking about creating constants to capture the meaning of values and sometimes to aid performance.

## Hello, world... again

The next requirement is when our function is called with an empty string it defaults to printing "Hello, World", rather than "Hello, ".

Start by writing a new failing test

```go
func TestHello(t *testing.T) {
	t.Run("saying hello to people", func(t *testing.T) {
		got := Hello("Chris")
		want := "Hello, Chris"

		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	})
	t.Run("say 'Hello, World' when an empty string is supplied", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"

		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	})
}
```

Here, we are introducing another tool in our testing arsenal: subtests. Sometimes, it is useful to group tests around a "thing" and then have subtests describing different scenarios.

A benefit of this approach is you can set up shared code that can be used in the other tests.

While we have a failing test, let's fix the code, using an `if`.

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

If we run our tests we should see it satisfies the new requirement and we haven't accidentally broken the other functionality.

It is important that your tests _are clear specifications_ of what the code needs to do. But there is repeated code when we check if the message is what we expect.

Refactoring is not _just_ for the production code!

Now that the tests are passing, we can and should refactor our tests.

```go
func TestHello(t *testing.T) {
	t.Run("saying hello to people", func(t *testing.T) {
		got := Hello("Chris")
		want := "Hello, Chris"
		assertCorrectMessage(t, got, want)
	})

	t.Run("empty string defaults to 'world'", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"
		assertCorrectMessage(t, got, want)
	})

}

func assertCorrectMessage(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

What have we done here?

We've refactored our assertion into a new function. This reduces duplication and improves the readability of our tests. We need to pass in `t *testing.T` so that we can tell the test code to fail when we need to.

For helper functions, it's a good idea to accept a `testing.TB` which is an interface that `*testing.T` and `*testing.B` both satisfy, so you can call helper functions from a test, or a benchmark (don't worry if words like "interface" mean nothing to you right now, it will be covered later).

`t.Helper()` is needed to tell the test suite that this method is a helper. By doing this, when it fails, the line number reported will be in our _function call_ rather than inside our test helper. This will help other developers track down problems more easily. If you still don't understand, comment it out, make a test fail and observe the test output. Comments in Go are a great way to add additional information to your code, or in this case, a quick way to tell the compiler to ignore a line. You can comment out the `t.Helper()` code by adding two forward slashes `//` at the beginning of the line. You should see that line turn grey or change to another color than the rest of your code to indicate it's now commented out.

When you have more than one argument of the same type \(in our case two strings\) rather than having `(got string, want string)` you can shorten it to `(got, want string)`.

### Back to source control

Now that we are happy with the code, I would amend the previous commit so that we only check in the lovely version of our code with its test.

### Discipline

Let's go over the cycle again

* Write a test
* Make the compiler pass
* Run the test, see that it fails and check the error message is meaningful
* Write enough code to make the test pass
* Refactor

On the face of it this may seem tedious but sticking to the feedback loop is important.

Not only does it ensure that you have _relevant tests_, it helps ensure _you design good software_ by refactoring with the safety of tests.

Seeing the test fail is an important check because it also lets you see what the error message looks like. As a developer it can be very hard to work with a codebase when failing tests do not give a clear idea as to what the problem is.

By ensuring your tests are _fast_ and setting up your tools so that running tests is simple you can get in to a state of flow when writing your code.

By not writing tests, you are committing to manually checking your code by running your software, which breaks your state of flow. You won't be saving yourself any time, especially in the long run.

## Keep going! More requirements

Goodness me, we have more requirements. We now need to support a second parameter, specifying the language of the greeting. If a language is passed in that we do not recognise, just default to English.

We should be confident that we can easily use TDD to flesh out this functionality!

Write a test for a user passing in Spanish. Add it to the existing suite.

```go
	t.Run("in Spanish", func(t *testing.T) {
		got := Hello("Elodie", "Spanish")
		want := "Hola, Elodie"
		assertCorrectMessage(t, got, want)
	})
```

Remember not to cheat! _Test first_. When you try to run the test, the compiler _should_ complain because you are calling `Hello` with two arguments rather than one.

```text
./hello_test.go:27:19: too many arguments in call to Hello
    have (string, string)
    want (string)
```

Fix the compilation problems by adding another string argument to `Hello`

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

When you try and run the test again it will complain about not passing through enough arguments to `Hello` in your other tests and in `hello.go`

```text
./hello.go:15:19: not enough arguments in call to Hello
    have (string)
    want (string, string)
```

Fix them by passing through empty strings. Now all your tests should compile _and_ pass, apart from our new scenario

```text
hello_test.go:29: got 'Hello, Elodie' want 'Hola, Elodie'
```

We can use `if` here to check the language is equal to "Spanish" and if so change the message

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == "Spanish" {
		return "Hola, " + name
	}
	return englishHelloPrefix + name
}
```

The tests should now pass.

Now it is time to _refactor_. You should see some problems in the code, "magic" strings, some of which are repeated. Try and refactor it yourself, with every change make sure you re-run the tests to make sure your refactoring isn't breaking anything.

```go
	const spanish = "Spanish"
	const englishHelloPrefix = "Hello, "
	const spanishHelloPrefix = "Hola, "

	func Hello(name string, language string) string {
		if name == "" {
			name = "World"
		}

		if language == spanish {
			return spanishHelloPrefix + name
		}
		return englishHelloPrefix + name
	}
```

### French

* Write a test asserting that if you pass in `"French"` you get `"Bonjour, "`
* See it fail, check the error message is easy to read
* Do the smallest reasonable change in the code

You may have written something that looks roughly like this

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == spanish {
		return spanishHelloPrefix + name
	}
	if language == french {
		return frenchHelloPrefix + name
	}
	return englishHelloPrefix + name
}
```

## `switch`

When you have lots of `if` statements checking a particular value it is common to use a `switch` statement instead. We can use `switch` to refactor the code to make it easier to read and more extensible if we wish to add more language support later

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	prefix := englishHelloPrefix

	switch language {
	case spanish:
		prefix = spanishHelloPrefix
	case french:
		prefix = frenchHelloPrefix
	}

	return prefix + name
}
```

Write a test to now include a greeting in the language of your choice and you should see how simple it is to extend our _amazing_ function.

### one...last...refactor?

You could argue that maybe our function is getting a little big. The simplest refactor for this would be to extract out some functionality into another function.

```go

const (
	spanish = "Spanish"
	french  = "French"

	englishHelloPrefix = "Hello, "
	spanishHelloPrefix = "Hola, "
	frenchHelloPrefix  = "Bonjour, "
)

func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	return greetingPrefix(language) + name
}

func greetingPrefix(language string) (prefix string) {
	switch language {
	case french:
		prefix = frenchHelloPrefix
	case spanish:
		prefix = spanishHelloPrefix
	default:
		prefix = englishHelloPrefix
	}
	return
}
```

A few new concepts:

* In our function signature we have made a _named return value_ `(prefix string)`.
* This will create a variable called `prefix` in your function.
  * It will be assigned the "zero" value. This depends on the type, for example `int`s are 0 and for `string`s it is `""`.
    * You can return whatever it's set to by just calling `return` rather than `return prefix`.
  * This will display in the Go Doc for your function so it can make the intent of your code clearer.
* `default` in the switch case will be branched to if none of the other `case` statements match.
* The function name starts with a lowercase letter. In Go, public functions start with a capital letter, and private ones start with a lowercase letter. We don't want the internals of our algorithm exposed to the world, so we made this function private.
* Also, we can group constants in a block instead of declaring them on their own line. For readability, it's a good idea to use a line between sets of related constants.

## Wrapping up

Who knew you could get so much out of `Hello, world`?

By now you should have some understanding of:

### Some of Go's syntax around

* Writing tests
* Declaring functions, with arguments and return types
* `if`, `const` and `switch`
* Declaring variables and constants

### The TDD process and _why_ the steps are important

* _Write a failing test and see it fail_ so we know we have written a _relevant_ test for our requirements and seen that it produces an _easy to understand description of the failure_
* Writing the smallest amount of code to make it pass so we know we have working software
* _Then_ refactor, backed with the safety of our tests to ensure we have well-crafted code that is easy to work with

In our case, we've gone from `Hello()` to `Hello("name")` and then to `Hello("name", "French")` in small, easy-to-understand steps.

Of course, this is trivial compared to "real-world" software, but the principles still stand. TDD is a skill that needs practice to develop, but by breaking problems down into smaller components that you can test, you will have a much easier time writing software.
