---
id: error-handling
title: 🐛 Error Handling
description: >-
  Fiber supports centralized error handling by returning an error to the handler
  which allows you to log errors to external services or send a customized HTTP
  response to the client.
sidebar_position: 5
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## Catching Errors

It’s essential to ensure that Fiber catches all errors that occur while running route handlers and middleware. You must return them to the handler function, where Fiber will catch and process them.

<Tabs>
<TabItem value="example" label="Example">

```go
app.Get("/", func(c *fiber.Ctx) {
    err := c.SendFile("file-does-not-exist")

    if err != nil {
        c.Next(err) // Pass error to Fiber
    }
})
```
</TabItem>
</Tabs>

Fiber does not handle [panics](https://blog.golang.org/defer-panic-and-recover) by default. To recover from a panic thrown by any handler in the stack, you need to include the `Recover` middleware below:

```go title="Example"
package main

import (
    "github.com/gofiber/fiber"
    "github.com/gofiber/fiber/middleware"
)

func main() {
    app := fiber.New()

    app.Use(middleware.Recover())

    app.Get("/", func(c *fiber.Ctx) {
        panic("This panic is catched by the ErrorHandler")
    })

    log.Fatal(app.Listen(3000))
}
```

Because `ctx.Next()` accepts an `error` interface, you could use Fiber's custom error struct to pass an additional `status code` using `fiber.NewError()`. It's optional to pass a message; if this is left empty, it will default to the status code message \(`404` equals `Not Found`\).

```go title="Example"
app.Get("/", func(c *fiber.Ctx) {
    err := fiber.NewError(503)
    c.Next(err) // 503 Service Unavailable

    err := fiber.NewError(404, "Sorry, not found!")
    c.Next(err) // 404 Sorry, not found!
})
```

## Default Error Handler

Fiber provides an error handler by default. For a standard error, the response is sent as **500 Internal Server Error**. If error is of type [fiber\*Error](https://godoc.org/github.com/gofiber/fiber#Error), response is sent with the provided status code and message.

```go title="Example"
// Default error handler
app.Settings.ErrorHandler = func(ctx *fiber.Ctx, err error) {
    // Statuscode defaults to 500
    code := fiber.StatusInternalServerError

    // Check if it's an fiber.Error type
    if e, ok := err.(*fiber.Error); ok {
        code = e.Code
    }

    // Return HTTP response
    ctx.Set(fiber.HeaderContentType, fiber.MIMETextPlainCharsetUTF8)
    ctx.Status(code).SendString(err.Error())
}
```

## Custom Error Handler

A custom error handler can be set via `app.Settings.ErrorHandler`

In most cases, the default error handler should be sufficient. However, a custom error handler can come in handy if you want to capture different types of errors and take action accordingly e.g., send a notification email or log an error to the centralized system. You can also send customized responses to the client e.g., error page or just a JSON response.

The following example shows how to display error pages for different types of errors.

```go title="Example"
app := fiber.New()

// Custom error handler
app.Settings.ErrorHandler = func(ctx *fiber.Ctx, err error) {
    // Statuscode defaults to 500
    code := fiber.StatusInternalServerError

    // Retrieve the custom statuscode if it's an fiber.*Error
    if e, ok := err.(*fiber.Error); ok {
        code = e.Code
    }

    // Send custom error page
    err = ctx.Status(code).SendFile(fmt.Sprintf("./%d.html", code))
    if err != nil {
        ctx.Status(500).SendString("Internal Server Error")
    }
}
```

> Special thanks to the [Echo](https://echo.labstack.com/) & [Express](https://expressjs.com/) framework for inspiration regarding error handling.
