# Rasa Chatbot for Customer Care 

Questo chatbot è un'esempio di chatbot per l'assistenza clienti. Include al suo interno, sia modelli di AI sviluppati sul framework open-source rasa, sia un widget per pagine web che permette di utilizzare il rasa chatbot in qualsiasi sito web aziendale. Il chatbot sviluppato permette all' utente di inserire ticket di assistenza clienti e  di chiederne lo stato di avanzamento della richiesta di assistenza. Il chatbot è collegato ad un database relazione sql che ne permette la persistenza dei dati.
Si puo utilizzare questo chatbot come punto di partenza per creare assistenti per il servizio clienti più strutturati e su misura della propria azienda.


Un esempio di conversazione con il bot:

![Screenshot](./screenshots/demo_ss.png?raw=true)


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Rasa Helpdesk Assistant Example](#rasa-helpdesk-assistant-example)
  - [Installazione](#setup)
    - [Installazione dipendenze](#installazione)
  - [Running the bot](#running-the-bot)
  - [Cosa puoi chiedere al bot](#things-you-can-ask-the-bot)
  - [Esempio di conversazione](#example-conversations)
  - [Handoff](#handoff)
    - [Try it out](#try-it-out)
    - [How it works](#how-it-works)
    - [Bot-side configuration](#bot-side-configuration)
  - [Testing the bot](#testing-the-bot)
  - [Widget Chatbot](#notes-on-chatroom)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Setup

### Installazione

In a Python3 virtual environment run:

```bash
pip install -r requirements.txt
```

To install development dependencies, run:

```bash
pip install -r requirements-dev.txt
pre-commit install
```

>Come virtual environment per lo sviluppo ho utilizza Anaconda: https://www.anaconda.com/products/individual


## Running the bot

Usa `rasa train` per addestrare il modello, una volta modificati i dati.

```bash
rasa train
```

Apri un'altra finestra , entra nel virtual enviroment, per eseguire il server di azioni personalizzate:

```bash
rasa run actions
```

In un'altra finestra di terminale ancora , runna il  duckling server (per l'estrazione dientità):

```bash
docker run -p 8000:8000 rasa/duckling
```

Se vuoi parlare con il robot da terminale, digita:

```bash
rasa shell 
```

Puoi utilizzare il flag  `--debug` per capire meglio come funziona il tutto. 

Se vuoi utilizzare il widget per pagine web, apri in un'altra finestra dell'ambiente di sviluppo, la directory "Webchat-Widget". 
In seguito invia il tutto al  http/:localhost:5005. Con i seguenti comandi:

```bash
npx http-server
```

Lato Rasa invece digitare il seguente comando:

```bash
rasa run -m models --enable-api -cors="*"
```



## Cose che puoi chiedere al bot

Il bot ha due principali skill:
1. Aprire un ticket di assistenza con il proprio indirizzo e-mail.
2. Chiedere al bot lo stato della propria richiesta di assistenza inserendo l'indirizzo e-mail con cui si è fatta la richiesta. Se non esiste il bot ti chiederà se ne vuoi creare un nuovo ticket.
Per alcuni problemi, come il reset della password oppure problemi con la propria mail, non occorre dare un titolo alla richiesta, se ne occuperà il bot grazie alla propria intelligenza.



## Example conversations





## Handoff

This bot includes a simple skill for handing off the conversation to another bot or a human.
This demo relies on [this fork of chatroom](https://github.com/RasaHQ/chatroom) to work, however you
could implement similar behaviour in another channel and then use that instead. See the chatroom README for
more details on channel-side configuration.


Using the default set up, the handoff skill enables this kind of conversation with two bots:

<img src="./handoff.gif" width="200">

### Try it out

The simplest way to use the handoff feature is to do the following:

1. Clone [chatroom](https://github.com/RasaHQ/chatroom) and [Financial-Demo](https://github.com/RasaHQ/Financial-Demo) alongside this repo
2. In the chatroom repo, install the dependencies:
```bash
yarn install
```
3. In the chatroom repo, build and serve chatroom:
```bash
yarn build
yarn serve
```
4. In the Financial-Demo repo, install the dependencies and train a model (see the Financial-Demo README)
5. In the Helpdesk-Assistant repo (i.e. this repo), run the rasa server and action server at the default ports (shown here for clarity)
   In one terminal window:
    ```bash
    rasa run --enable-api --cors "*" --port 5005 --debug
    ```
    In another terminal window:
    ```bash
    rasa run actions --port 5055 --debug
    ```
6. In the Financial-Demo repo, run the rasa server and action server at **the non-default ports shown below**
   In one terminal window:
    ```bash
    rasa run --enable-api --cors "*" --port 5006 --debug
    ```
    In another terminal window:
    ```bash
    rasa run actions --port 5056 --debug
    ```
7. Open `chatroom_handoff.html` in a browser to see handoff in action


### How it works

Using chatroom, the general approach is as follows:

1. User asks original bot for a handoff.
2. The original bot handles the request and eventually
   sends a message with the following custom json payload:
    ```
        {
            "handoff_host": "<url of handoff host endpoint>",
            "title": "<title for bot/channel handed off to>"
            }
    ```
    This message is not displayed in the Chatroom window.
3. Chatroom switches the host to the specified `handoff_host`
4. The original bot no longer receives any messages.
5. The handoff host receives the message `/handoff{"from_host":"<original bot url">}`
6. The handoff host should be configured to respond to this message with something like,
   "Hi, I'm <so and so>, how can I help you??"
7. The handoff host can send a message in the same format as specified above to hand back to the original bot.
   In this case the same pattern repeats, but with
   the roles reversed. It could also hand off to yet another bot/human.

### Bot-side configuration

The "try it out" section doesn't require any further configuration; this section is for those
who want to change or further understand the set up.

For this demo, the user can ask for a human, but they'll be offered a bot (or bots) instead,
so that the conversation looks like this:


For handoff to work, you need at least one "handoff_host". You can specify any number of handoff hosts in the file `actions/hanodff_config.yml`.
```
handoff_hosts:
    financial_demo:
      title: "Financial Demo"
      url: "http://localhost:5006"
    ## you can add more handoff hosts to this list e.g.
    # moodbot:
    #   title: "MoodBot"
    #   url: "http://localhost:5007"
```

Handoff hosts can be other locally running rasa bots, or anything that serves responses in the format that chatroom
accepts. If a handoff host is not a rasa bot, you will of course want to update the response text to tell the user
who/what they are being handed off to.

The [Financial-Demo](https://github.com/RasaHQ/Financial-Demo) bot has been set up to handle handoff in exactly the same way as Helpdesk-Assistant,
so the simplest way to see handoff in action is to clone Financial-Demo alongside this repo.

If you list other locally running bots as handoff hosts, make sure the ports on which the various rasa servers & action servers are running do not conflict with each other.


## Testing the bot

You can test the bot on the test conversations by running  `rasa test`.
This will run [end-to-end testing](https://rasa.com/docs/rasa/user-guide/testing-your-assistant/#end-to-end-testing) on the conversations in `tests/conversation_tests.md`.

