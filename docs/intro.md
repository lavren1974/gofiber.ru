---
slug: /
id: welcome
title: 👋 Добро пожаловать
sidebar_position: 1
---

Онлайн-документация по API с примерами, чтобы вы могли начать создавать веб-приложения с помощью Fiber прямо сейчас!

**Fiber**- это веб-фреймворк в стиле [Express](https://github.com/expressjs/express), построенный поверх [Fasthttp](https://github.com/valyala/fasthttp), самого быстрого HTTP-движка для [Go](https://go.dev/doc/). Разработан, чтобы **упростить** процесс **быстрой** разработки с учетом **нулевого выделения памяти** и **производительности**.

Эти документы предназначены для версии **Fiber v2**, которая была выпущена **15 сентября 2020 года**.

### Установка

Прежде всего, [загрузите](https://go.dev/dl/) и установите Go. `1.17` требуется или выше.

Установка выполняется с помощью [`go get`](https://pkg.go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) команды:

```bash
go get github.com/gofiber/fiber/v2
```

### Нулевое распределение
Некоторые значения, возвращенные из \***fiber.Ctx** являются **не** неизменными по умолчанию.

Поскольку fiber оптимизирован для **высокой производительности**, значения, возвращаемые из **fiber.Ctx** по умолчанию **не** являются неизменяемыми и **будут** повторно использоваться в запросах. Как правило, вы **должны** использовать только значения контекста внутри обработчика, и вы **не должны** сохранять какие-либо ссылки. Как только вы вернетесь из обработчика, любые значения, полученные вами из контекста, будут повторно использованы в будущих запросах и изменятся у вас под ногами. Вот пример:

```go
func handler(c *fiber.Ctx) error {
    // Переменная допустима только в пределах этого обработчика
    result := c.Params("foo") 

    // ...
}
```

Если вам нужно сохранить такие значения вне обработчика, сделайте копии их **базового буфера**, используя встроенную функцию [копирования](https://pkg.go.dev/builtin/#copy). Вот пример сохранения строки:

```go
func handler(c *fiber.Ctx) error {
    // Переменная допустима только в пределах этого обработчика
    result := c.Params("foo")

    // Сделайте копию
    buffer := make([]byte, len(result))
    copy(buffer, result)
    resultCopy := string(buffer) 
    // Переменная теперь действительна вечно

    // ...
}
```

Мы создали пользовательскую `CopyString` функцию, которая выполняет вышеуказанное и доступна в [gofiber/utils](https://github.com/gofiber/fiber/tree/master/utils).

```go
app.Get("/:foo", func(c *fiber.Ctx) error {
	// Переменная теперь неизменяема
	result := utils.CopyString(c.Params("foo")) 

	// ...
})
```

В качестве альтернативы вы также можете использовать `Immutable` настройку. Это сделает все значения, возвращаемые из контекста, неизменяемыми, что позволит вам сохранять их где угодно. Конечно, это достигается за счет производительности.

```go
app := fiber.New(fiber.Config{
	Immutable: true,
})
```

Для получения дополнительной информации, пожалуйста, проверьте [**\#426**](https://github.com/gofiber/fiber/issues/426) и [**\#185**](https://github.com/gofiber/fiber/issues/185).

### Привет, Мир!

Встроенное ниже, по сути, самое простое приложение **Fiber**, которое вы можете создать:

```go
package main

import "github.com/gofiber/fiber/v2"

func main() {
	app := fiber.New()

	app.Get("/", func(c *fiber.Ctx) error {
		return c.SendString("Привет, Мир!")
	})

	app.Listen(":3000")
}
```

```text
go run server.go
```

Перейдите к `http://localhost:3000` и вы должны увидеть `Привет, Мир!` на странице.

### Базовая маршрутизация

Маршрутизация относится к определению того, как приложение отвечает на запрос клиента к конкретной конечной точке, которая представляет собой URI (или путь) и конкретный метод HTTP-запроса (`GET`, `PUT`, `POST` и т.д.).

У каждого маршрута может быть **несколько функций обработки**, которые выполняются при сопоставлении маршрута.

Определение маршрута принимает следующие структуры:

```go
// Сигнатура функции
app.Method(path string, ...func(*fiber.Ctx) error)
```

- `app` является примером **Fiber**
- `Method` это [метод HTTP-запроса](https://docs.gofiber.io/api/app#route-handlers): `GET`, `PUT`, `POST` и т.д.
- `path` это виртуальный путь на сервере
- `func(*fiber.Ctx) error` является ли функция обратного вызова, содержащая [Контекст](https://docs.gofiber.io/api/ctx), выполняемой при сопоставлении маршрута

**Простой маршрут**

```go
// Ответьте "Привет, Мир!" по корневому пути, "/"
app.Get("/", func(c *fiber.Ctx) error {
	return c.SendString("Привет, Мир!")
})
```

**Параметры**

```go
// GET http://localhost:8080/hello%20world

app.Get("/:value", func(c *fiber.Ctx) error {
	return c.SendString("value: " + c.Params("value"))
	// => Получить запрос со значением: hello world
})
```

**Необязательный параметр**

```go
// GET http://localhost:3000/john

app.Get("/:name?", func(c *fiber.Ctx) error {
	if c.Params("name") != "" {
		return c.SendString("Hello " + c.Params("name"))
		// => Hello john
	}
	return c.SendString("Where is john?")
})
```

**Подстановочные знаки**

```go
// GET http://localhost:3000/api/user/john

app.Get("/api/*", func(c *fiber.Ctx) error {
	return c.SendString("API path: " + c.Params("*"))
	// => API path: user/john
})
```

### Статические файлы

Чтобы обслуживать статические файлы, такие как **images**, файлы **CSS** и **JavaScript**, замените обработчик вашей функции строкой файла или каталога.

Сигнатура функции:

```go
app.Static(prefix, root string, config ...Static)
```

Используйте следующий код для обслуживания файлов в каталоге с именем `./public`:

```go
app := fiber.New()

app.Static("/", "./public") 

app.Listen(":3000")
```

Теперь вы можете загрузить файлы, которые находятся в `./public` каталоге:

```bash
http://localhost:8080/hello.html
http://localhost:8080/js/jquery.js
http://localhost:8080/css/style.css
```

### Примечание

Для получения дополнительной информации о том, как создавать API в Go с помощью Fiber, пожалуйста, ознакомьтесь с этой превосходной статьей [о создании API в стиле express в Go с помощью Fiber](https://blog.logrocket.com/express-style-api-go-fiber/).
