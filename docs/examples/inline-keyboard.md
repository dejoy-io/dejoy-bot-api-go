# Inline Keyboard

This bot waits for you to send it the message "open" before sending you an
inline keyboard containing a URL and some numbers. When a number is clicked, it
sends you a message with your selected number.

```go
package main

import (
	"log"
	"os"

	djbotapi "github.com/dejoy-io/dejoy-bot-api"
)

var numericKeyboard = djbotapi.NewInlineKeyboardMarkup(
	djbotapi.NewInlineKeyboardRow(
		djbotapi.NewInlineKeyboardButtonURL("1.com", "http://1.com"),
		djbotapi.NewInlineKeyboardButtonData("2", "2"),
		djbotapi.NewInlineKeyboardButtonData("3", "3"),
	),
	djbotapi.NewInlineKeyboardRow(
		djbotapi.NewInlineKeyboardButtonData("4", "4"),
		djbotapi.NewInlineKeyboardButtonData("5", "5"),
		djbotapi.NewInlineKeyboardButtonData("6", "6"),
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

	// Loop through each update.
	for update := range updates {
		// Check if we've gotten a message update.
		if update.Message != nil {
			// Construct a new message from the given chat ID and containing
			// the text that we received.
			msg := djbotapi.NewMessage(update.Message.Chat.ID, update.Message.Text)

			// If the message was open, add a copy of our numeric keyboard.
			switch update.Message.Text {
			case "open":
				msg.ReplyMarkup = numericKeyboard

			}

			// Send the message.
			if _, err = bot.Send(msg); err != nil {
				panic(err)
			}
		} else if update.CallbackQuery != nil {
			// Respond to the callback query, telling Dejoy to show the user
			// a message with the data received.
			callback := djbotapi.NewCallback(update.CallbackQuery.ID, update.CallbackQuery.Data)
			if _, err := bot.Request(callback); err != nil {
				panic(err)
			}

			// And finally, send a message containing the data received.
			msg := djbotapi.NewMessage(update.CallbackQuery.Message.Chat.ID, update.CallbackQuery.Data)
			if _, err := bot.Send(msg); err != nil {
				panic(err)
			}
		}
	}
}
```
