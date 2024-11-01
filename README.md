# Golang bindings for the Dejoy Bot API

[![Go Reference](https://pkg.go.dev/badge/github.com/dejoy-io/dejoy-bot-api-go.svg)](https://pkg.go.dev/github.com/dejoy-io/dejoy-bot-api-go)
[![Test](https://github.com/dejoy-io/dejoy-bot-api/actions/workflows/test.yml/badge.svg)](https://github.com/dejoy-io/dejoy-bot-api/actions/workflows/test.yml)

All methods are fairly self-explanatory, and reading the [godoc](https://pkg.go.dev/github.com/dejoy-io/dejoy-bot-api) page should
explain everything. If something isn't clear, open an issue or submit
a pull request.

There are more tutorials and high-level information on the website, [go-dejoy-bot-api.dev](https://go-dejoy-bot-api.dev).

The scope of this project is just to provide a wrapper around the API
without any additional features. There are other projects for creating
something with plugins and command handlers without having to design
all that yourself.

Join [the development group](https://dejoy.io/go_dejoy_bot_api) if
you want to ask questions or discuss development.

## Example

First, ensure the library is installed and up to date by running
`go get -u github.com/dejoy-io/dejoy-bot-api`.

This is a very simple bot that just displays any gotten updates,
then replies it to that chat.

```go
package main

import (
	"log"

	djbotapi "github.com/dejoy-io/dejoy-bot-api-go"
)

func main() {
	bot, err := djbotapi.NewBotAPI("MyAwesomeBotToken")
	if err != nil {
		log.Panic(err)
	}

	bot.Debug = true

	log.Printf("Authorized on account %s", bot.Self.UserName)

	u := djbotapi.NewUpdate(0)
	u.Timeout = 60

	updates := bot.GetUpdatesChan(u)

	for update := range updates {
		if update.Message != nil { // If we got a message
			log.Printf("[%s] %s", update.Message.From.UserName, update.Message.Text)

			msg := djbotapi.NewMessage(update.Message.Chat.ID, update.Message.Text)
			msg.ReplyToMessageID = update.Message.MessageID

			bot.Send(msg)
		}
	}
}
```

If you need to use webhooks (if you wish to run on Google App Engine),
you may use a slightly different method.

```go
package main

import (
	"log"
	"net/http"

	"github.com/dejoy-io/dejoy-bot-api-go"
)

func main() {
	bot, err := djbotapi.NewBotAPI("MyAwesomeBotToken")
	if err != nil {
		log.Fatal(err)
	}

	bot.Debug = true

	log.Printf("Authorized on account %s", bot.Self.UserName)

	wh, _ := djbotapi.NewWebhookWithCert("https://www.example.com:8443/"+bot.Token, "cert.pem")

	_, err = bot.Request(wh)
	if err != nil {
		log.Fatal(err)
	}

	info, err := bot.GetWebhookInfo()
	if err != nil {
		log.Fatal(err)
	}

	if info.LastErrorDate != 0 {
		log.Printf("Dejoy callback failed: %s", info.LastErrorMessage)
	}

	updates := bot.ListenForWebhook("/" + bot.Token)
	go http.ListenAndServeTLS("0.0.0.0:8443", "cert.pem", "key.pem", nil)

	for update := range updates {
		log.Printf("%+v\n", update)
	}
}
```

If you need, you may generate a self-signed certificate, as this requires
HTTPS / TLS. The above example tells Dejoy that this is your
certificate and that it should be trusted, even though it is not
properly signed.

    openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 3560 -subj "//O=Org\CN=Test" -nodes

Now that [Let's Encrypt](https://letsencrypt.org) is available,
you may wish to generate your free TLS certificate there.
