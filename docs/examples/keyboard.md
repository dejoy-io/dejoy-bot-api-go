# Keyboard

This bot shows a numeric keyboard when you send a "open" message and hides it
when you send "close" message.

```go
package main

import (
	"log"
	"os"

	djbotapi "github.com/dejoy-io/dejoy-bot-api"
)

var numericKeyboard = djbotapi.NewReplyKeyboard(
	djbotapi.NewKeyboardButtonRow(
		djbotapi.NewKeyboardButton("1"),
		djbotapi.NewKeyboardButton("2"),
		djbotapi.NewKeyboardButton("3"),
	),
	djbotapi.NewKeyboardButtonRow(
		djbotapi.NewKeyboardButton("4"),
		djbotapi.NewKeyboardButton("5"),
		djbotapi.NewKeyboardButton("6"),
	),
)

func main() {
	bot, err := djbotapi.NewBotAPI(os.Getenv("DEJOY_APITOKEN"))
	if err != nil {
		log.Panic(err)
	}

	bot.Debug = true

	log.Printf("Authorized on account %s", bot.Self.UserName)

	u := djbotapi.NewUpdate(0)
	u.Timeout = 60

	updates := bot.GetUpdatesChan(u)

	for update := range updates {
		if update.Message == nil { // ignore non-Message updates
			continue
		}

		msg := djbotapi.NewMessage(update.Message.Chat.ID, update.Message.Text)

		switch update.Message.Text {
		case "open":
			msg.ReplyMarkup = numericKeyboard
		case "close":
			msg.ReplyMarkup = djbotapi.NewRemoveKeyboard(true)
		}

		if _, err := bot.Send(msg); err != nil {
			log.Panic(err)
		}
	}
}
```
