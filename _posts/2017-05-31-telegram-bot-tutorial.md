---
layout: post
title: "Tutorial: Telegram bookstore bot in Haskell"
description: "Recently Telegram rolled out their new update for Telegram Bot API that introduced payments. So now developers can build merchant bots. In this tutorial, we are going to create a simple bookstore bot that will sell these wonderful O'RLY books that teach as how to build better software."
category: dev
tags: [ Haskell, Telegram, Bot ]
---
{% include JB/setup %}

# Intro

Recently Telegram rolled out their new update for [Telegram Bot API][telegram-bot-api] that introduced payments. So now developers can build merchant bots.
In this tutorial, we are going to create a simple bookstore bot that will sell these wonderful [O'RLY books][orly-books] that teach as how to build better software.
We will go through the process of making a new bot from scratch, creating and debugging webhook, extending it to list books and conduct payments using test payments provider.
And all of that in Haskell!

*Note: You won't find here the detailed explanation of different Haskell features.
It would make this tutorial unneccessary bigger and there are much better explanations on the Internet.
I will try to provide some useful links you can use to understand better what's going on.*

I'm going to use [haskell-telegram-api][haskell-telegram-api], Telegram Bot API bindings based on [Servant][servant] library and Servant library itself to create a webhook for our bot.
Webhook based implementation makes our bot more responsive in comparison with polling based model but would require our bot to be accessible by Telegram servers on the Internet. This makes it harder to develop bot, but we will solve this issue using [ngrok][ngrok].
You can read more about receiveing updates from Telegram [here][receiving-updates].
In this tutorial, Telegram will call the webhook to notify our bot of user's actions and the bot will react on them.

There are two ways to interact with the Telegram servers when the webhook is called.
You can directly answer Telegram's webhook requests with an appropriate response to user action like send him a message.
But in this case you won't be able to know was your response successfully accepted by Telegram or not.
Another option is to call Telegram directly when bot received a webhook request.
See more info [here][making-requests] and [here][making-requests-visual] for visual representation.

We are going to use the later one because it makes it simpler and at the moment *telegram-api* library gives us only this option.

# Step 0: Prerequisites

In order to start we need to

1) register our bot with [@BotFather][botfather] and receive *bot token*. Keep it secret! See more information about registering your bot on official Telegram [page][reg-bot].

2) obtain test *payment token* from [@BotFather][botfather]. Keep it secret too!

3) install [ngrok][ngrok] or any other simial tool.

I assume you already have [stack][stack] and some [Haskell IDE][haskell-ide] installed.

# Step 1: Initialize project

Stack provides us with the possibility to create a new project from a template and there is already servant template suitable for our needs.
We will create a new bot project with the name `orly-bookstore-bot` from the `servant` template by running command:

```bash
stack orly-bookstore-bot servant
```

Explore newly created folder a bit to see what was created from template and add these dependencies to `orly-bookstore-bot.cabal` file into `library.build-depends` section. 

```yaml
library
  build-depends:
                     # other dependencies
                     , mtl
                     , http-client
                     , http-client-tls
                     , telegram-api >= 0.6.3.0
                     , text
                     , transformers
```

`telegram-api` should be no less than version `0.6.3.0` because payments API has been added in this version.

Now you can open `src/Libs.hs` file, remove everything that is below imports except `startApp` function since it is exported.
We will redefine `app` function which is used for the unit tests in this template, but I will skip testing aspect in this blog post.
You can take a look at `test/Spec.hs` to get more context on how to test your bot. 
But comment out it for now.

Add these imports to the `src/Libs.hs` file:

```haskell
import GHC.Generics
import Control.Monad.Reader
import Control.Monad.Except
import Data.Text (Text)
import qualified Data.Text as T
import Network.HTTP.Client (Manager)
import Network.HTTP.Client.TLS  (tlsManagerSettings)
import Data.Maybe
import Data.Monoid
import Web.Telegram.API.Bot
import System.Environment
import qualified aths_orly_booksstore_bot (version) as P
import Data.Version (showVersion)

```

Same for the language extensions. These are very useful ones:

```haskell
{-# LANGUAGE DeriveGeneric     #-}
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE GeneralizedNewtypeDeriving #-}
{-# LANGUAGE DeriveFunctor #-}
{-# LANGUAGE ViewPatterns #-}
{-# LANGUAGE RecordWildCards #-}
```

Some of the imports and extentions are not needed at the moment, but we will need them later.

# Step 1: Create version resource with Servant

As the first step, I'd like to focus on Servant and create simple version resource to make you familiar with Servant library.
If you already know this you can skip this step and go to Step 2.

The main idea of Servant is that your Web API can be defined and described entirely by the type.
It brings you a lot of useful features, but I'd refer you to Servant documentation to get more information on that.
We are going to define this type for version page to start with and later extend it with webhook resource.

```haskell
-- We needed some of those language extensions to make it that simple
data Version = Version
  { version :: Text
  } deriving (Show, Generic)

instance ToJSON Version

-- At the moment Bot API consists of only version resource
-- that returns Version data record as a JSON.
-- Thanks to Generic and ToJSON instance Servant knows how
type BotAPI = "version" :> Get '[JSON] Version

botApi :: Proxy BotAPI
botApi = Proxy

startApp :: IO ()
startApp = do
  putStrLn "ORLY book store bot is starting..."
  run 8080 app

app :: Application
app = serve botApi botServer

-- actual server implementation
botServer :: Server BotAPI
botServer = returnVersion
    where version' = Version $ pack $ showVersion P.version
          returnVersion = return version'
```

**Important:** You have to add `Path_orly_bookstore_bot` to the list of exposed or other modules in your cabal file!
Otherwise, it won't compile with very undescriptive message.

Run `stack build`. It should pull all dependencies and compile. 

Now you can run your bot with: 

```
$ stack exec orly-bookstore-bot-exe
ORLY book store bot is starting...
```

and see the result with:

```
$ curl http://localhost:8080/version
{"version":"0.1.0.0"}
```

# Step 2: Create Bot monad transformer and configuration

Strictly speaking part of this step is not required and a bit advanced for simple bot implementation.
We are going to create a `Bot` monad only to improve the way we read configuration and reduce the amount of boilerplate code,
but for the sake of simplicity, you can pass configuration parameter manually.

Let's define Bot monad transformer stack:

```haskell
newtype Bot a = Bot
    { runBot :: ReaderT BotConfig Handler a
    } deriving ( Functor, Applicative, Monad, MonadIO
                 MonadReader BotConfig, MonadError ServantErr)
```

where `Handler` is Servant's Handler which is nothing more than just a type alias for `ExceptT ServantError IO`.
The `Bot` type is accompanied by the classic set of deriving instances (don't forget to put `GeneralizedNewtypeDeriving` language extension on the top of the file).
You can read more about monad transformers on [wiki books][wikibooks-transformers], about deriving mtl typeclasses [here][mtl-transformers], and more about `ReaderT` [here][readert].

`BotConfig` is a read-only configuration that will be used by the bot.
It will be data record that we will build on application start up.

```haskell
data BotConfig = BotConfig 
  { telegramToken :: Token
  , paymentsToken :: Text
  , manager :: Manager
  }
```

In order to initialize our bot with configuration let's change `startApp` and `app` functions. 
We will read bot settings from environment variables using `getEnvironment` function from `System.Environment` module,
build `BotConfig` and initialize our bot with it.

```haskell
startApp :: IO ()
startApp = do
  putStrLn "ORLY book store bot is starting..."
  env <- getEnvironment
  manager' <- newManager tlsManagerSettings
  let telegramToken' = fromJust $ lookup "TELEGRAM_TOKEN" env
      paymentsToken' = fromJust $ lookup "PAYMENTS_TOKEN" env
      config = BotConfig
        { telegramToken = Token $ T.pack $ "bot" <> telegramToken'
        , paymentsToken = T.pack paymentsToken'
        , manager = manager'
        }
  run 8080 $ app config

app :: BotConfig -> Application
app config = serve botApi $ initBotServer config
```

Now we need to change our `botServer` function signature to return `ServerT BotAPI Bot` because we want to work with `Bot` monad and implement an `initBotServer` function that will do natural transformation from our `Bot` monad to to Servant's `ExceptT ServantErr IO` and initialize server.

```haskell
botServer :: ServerT BotAPI Bot
botServer = returnVersion
    where version' = Version $ pack $ showVersion P.version
          returnVersion :: Bot Version
          returnVersion = return version'

initBotServer :: BotConfig -> Server BotAPI
initBotServer config = enter (transform config) botServer
    where transform :: BotConfig -> Bot :~> ExceptT ServantErr IO
          transform config = Nat (flip runReaderT config . runBot)
```

Compile and run. At that point, your bot should be returning version page as before.
Note that even though we used unsafe functions `fromJust` to get tokens our application is working fine.
Haskell is really lazy and it saves our application from crashing on startup.

# Step 3: Add webhook

It's time to create a webhook for our bot!

At first, we will extend `BotAPI` type to have webhook resource defined there.
For the webhook we need to make sure that our bot accepts request only from Telegram's servers.
So as it's suggested in their [documentation][telegram-webhook] it's fine to use bot's token itself as the path parameter.
They will send POST request with `Update` object (from [telegram-api][haskell-telegram-api]) to the webhook and we will validate secret token in the path parameter to authorize Telegram.
If validation was successful our bot will handle `Update` message.
Servant library provides us `Capture` combinator that we will use to read the secret token from the path parameter.

The final version of the `BotAPI` looks like this:

```haskell
type BotAPI = "version" :> Get '[JSON] Version
         :<|> "webhook" -- maps to /webhook/<secret_token>
              :> Capture "secret" Text
              :> ReqBody '[JSON] Update
              :> Post '[JSON] ()
```

And we have to change `botServer` function again to add a handler for the webhook.
Our bot will not compile before we do that.
Type safety! We are going to use the same wiered operator `:<|>` here that we used to add the webhook to `BotAPI`.

```haskell
botServer :: ServerT BotAPI Bot
botServer = returnVersion :<|> handleWebhook
    where version' = Version $ T.pack $ showVersion P.version
          returnVersion :: Bot Version
          returnVersion = return version'
          handleWebhook :: Text -> Update -> Bot ()
          handleWebhook secret update = do
              Token token <- asks telegramToken
              if EQ == compare secret token
                 then handleUpdate update
                 else throwError err403

handleUpdate :: Update -> Bot ()
handleUpdate update = do
    case update of
--      Update { ... } more cases will go here
        _ -> liftIO $ putStrLn $ "Handle update failed. " ++ show update
```

Compile and run your bot with `stack`. It's time to test it.

# Step 4: Test your bot with ngrok

In order to test the bot from our local machine, we would need some tunnel to make our bot that is running locally accessible from the Internet. 
Personally, I use [ngrok][ngrok].
It's free for 30 requests per minute, easy to use and records interaction with your endpoint.
But you can use anyone you like.

So our bot is running, start ngrok with the command:

```
ngrok http 8080
```

Now you can see HTTP and HTTPS URLs that ngrok assigned to us and all requests that go to these URLs will appear here.
You can also open `http://localhost:4040` and see full information about requests and responses with their headers, bodies, and ability to replay them again to test you server.

We need to use HTTPS URL for our webhook. Copy it and run the `curl` command similar to what you see below:

```
curl "https://api.telegram.org/bot$TELEGRAM_TOKEN/setWebhook?url=https://<ngrok_id_here>.ngrok.io/webhook/bot$TELEGRAM_TOKEN"
```

Now you can go to Telegram client and send any message you want to your bot.
The bot will print it to output and you can see it in your terminal window, but won't respond to the client.

# Step 5: First interaction (help command)

Our bot does not do much at the moment. 
It does not even send any messages yet to the client. 
It's time to change it.
Define help message request and go to the `handleUpdate` function. 

```haskell
helpMessage userId = sendMessageRequest userId $ T.unlines
    [ "/help - show this message"
    , "/books - show list of all books"
    , "/find title - find book by title"
    ]

handleUpdate :: Update -> Bot ()
handleUpdate update = do
    case update of
        Update { message = Just msg } -> handleMessage msg
        _ -> liftIO $ putStrLn $ "Handle update failed. " ++ show update

handleMessage :: Message -> Bot ()
handleMessage msg = do
    BotConfig{..} <- ask
    let chatId = ChatId $ fromIntegral $ user_id $ fromJust $ from msg
        messageText = text msg
        sendHelpMessage = sendMessageM (helpMessage chatId) >> return ()
        onCommand (Just (T.stripPrefix "/help" -> Just _)) = sendHelpMessage
        onCommand _ = sendHelpMessage
    liftIO $ runClient (onCommand messageText) telegramToken manager
    return ()
```

`handleUpdate` uses pattern matching to match on `Update` object when `handleMessage`, in turn, matches on message content. You can see trick with pattern matching on text prefix in the implementation of the `onCommand` function: `(Just (T.stripPrefix "/command" -> Just args)) = ...`.
`onCommand` returns operations that `runClient` will execute, so now we can extend list of supported commands by defining new patterns.

# Step 6: List books and send invoices

Finally we came to the point when we will send books to the users with special button to pay for them.

Define the list of books with prices:

```haskell
allBooks :: [(Text, (Text, Text, Int))]
allBooks =
  [ ("Copying and Pasting from Stack Overflow",
        ("http://i.imgur.com/fawRchq.jpg", "Cutting corners to meet arbitrary management deadlines", 7000))
  , ("Googling the Error Message",
        ("http://i.imgur.com/fhgzVEt.jpg", "The internet will make those bad words go away", 4500))
  , ("Whiteboard Interviews",
        ("http://i.imgur.com/oM9yCym.png", "Putting the candidate through the same bullshit you went through", 3200))
  ]
```

Now we need to add a function to build invoice messages from books data using `sendInvoiceRequest` function.
It's the function from Telegram API library that creates invoice request with default parameters for optional fields, but you can easily set these fields like `snd_inv_photo_url` in example below.

```haskell
buildBuyBookInvoice (ChatId chatId) token (title, (image, description, price)) =
    (sendInvoiceRequest chatId title description payload token link code prices)
        { snd_inv_photo_url = Just image }
        where code = CurrencyCode "USD"
              payload = "book_payment_payload"
              link = "deep_link"
              prices = [ LabeledPrice title price
                       , LabeledPrice "Donation to a kitten hospital" 300
                       , LabeledPrice "Discount for donation" (-300) ]
```

The last change is to extend `handleMessage` function.
For demonstration purposes, we will add new commands to list all books with `/books` command and `/find <book_name_part>` to find books by title.
To build invoice messages we would need to `map` with `buildBuyBookInvoice` function over the books we want to send to the user.
Then we will `mapM_` with `sendInvoiceM` function over invoice requests we created before and let `runClient` send them eventually to the end user.

```haskell
sendInvoices books = mapM_ sendInvoiceM $ map (buildBuyBookInvoice chatId paymentsToken) books
byTitle title book = T.isInfixOf title $ fst book
onCommand (Just (T.stripPrefix "/help" -> Just _)) = sendHelpMessage
onCommand (Just (T.stripPrefix "/books" -> Just _)) = sendInvoices allBooks
onCommand (Just (T.stripPrefix "/find " -> Just title)) = sendInvoices $ filter (byTitle title) allBooks
```

Build and run it now, check that ngrok is still running.
If not, start it and update your webhook URL.
Open your Telegram client and type `/books` in your bot's chat.
Your bot should send you messages with *Pay* button with the price on it for every book.

You should see something like this.

![Telegram Client - Books]({{ BASE_PATH}}/assets/posts/2017-05-31-bookstore-bot/books_invoices.jpg)

And like this if you send `/find Whiteboard` and `/help` or any other unrecognized message to the bot.

![Telegram Client - Find and Help]({{ BASE_PATH}}/assets/posts/2017-05-31-bookstore-bot/find_and_help.jpg)

# Step 7: Accept payments

Step by step process to send invoices and confirm payment described in details on official [Telegram documentation page][payments-step-by-step].

We are not going to add shipping related funcitonality, but we do need to extend out bot to compete payments.
In order to do so we will extend `handleUpdate` function to handle different type of `Update`s. 
Telegram will send us `pre_checkout_query` and our bot must reply with `answerPrecheckoutQueryM` within 10 seconds.
If everything is OK, Telegram will call webhook with an update containing `Message` with `successful_payment` to confirm payment and complete transaction.
Now we can send user's order to him.

So we need `handlePreCheckout` function to answer pre checkout query from Telegram:

```haskell
handlePreCheckout :: PreCheckoutQuery -> Bot ()
handlePreCheckout query = do
    BotConfig{..} <- ask
    let chatId = ChatId $ fromIntegral $ user_id $ pre_che_from query
        queryId = pre_che_id query
        okRequest = AnswerPreCheckoutQueryRequest queryId True Nothing
    liftIO $ runClient (answerPreCheckoutQueryM okRequest) telegramToken manager
    return ()
```

And `handleSucccessfulPayment` function where will "trigger" book shipment to the client.

```haskell
handleSuccessfulPayment :: SuccessfulPayment -> Bot ()
handleSuccessfulPayment payment = do
    let totalAmount = T.pack $ show $ (suc_pmnt_total_amount payment) `div` 100
        CurrencyCode code = suc_pmnt_currency payment
    liftIO $ print $ "We have earned " <> code <> totalAmount <> ". Shipping book to the client!"
    return ()
```

And last step is to extend `handleUpdate` function to use these functions.

```haskell
handleUpdate update = do
    case update of
        Update { message = Just Message
          { successful_payment = Just payment } } -> handleSuccessfulPayment payment
        Update { pre_checkout_query = Just query } -> handlePreCheckout query
        -- the rest of the method
```

Build and run. Now our bot can do payments!

Try to buy any book from our bot and after completing all steps
using test credit cart `4242 4242 4242 4242` with arbitrary CVV
you should see something like this:

![Telegram Client - Result]({{ BASE_PATH}}/assets/posts/2017-05-31-bookstore-bot/result.jpg)

The bot will print something like this in the output:

```
ORLY book store bot is starting...
"We have earned USD45. Shipping book to the client!"
```

# Conclusions

We went through the several steps in this tutorial,
started with the simple web application based on [Servant][servant] that returns its version,
added webhook that the Telegram servers will use to send updates to our bot,
added handlers for different types of commands our bot can understand,
such as `/help`, `/books`, `/find title`, 
and added support for payment mechanism to allow users to buy books.

You can take a look at the complete [source code][src] of the bot on Github and implement your own bot.

[telegram-bot-api]: https://core.telegram.org/bots/api
[orly-books]: http://imgur.com/gallery/vqUQ5
[receiving-updates]: https://core.telegram.org/bots/api#getting-updates
[making-requests]: https://core.telegram.org/bots/api#making-requests
[making-requests-visual]: https://core.telegram.org/bots/faq#how-can-i-make-requests-in-response-to-updates
[botfather]: https://telegram.me/botfather
[reg-bot]: https://core.telegram.org/bots#3-how-do-i-create-a-bot
[stack]: https://docs.haskellstack.org/en/stable/README/
[haskell-ide]: https://wiki.haskell.org/IDEs
[haskell-telegram-api]: https://github.com/klappvisor/haskell-telegram-api
[servant]: https://haskell-servant.readthedocs.io/en/stable/
[wikibooks-transformers]: https://en.wikibooks.org/wiki/Haskell/Monad_transformers
[mtl-transformers]: https://lexi-lambda.github.io/blog/2017/04/28/lifts-for-free-making-mtl-typeclasses-derivable/
[readert]: http://dev.stephendiehl.com/hask/#readert
[telegram-webhook]: https://core.telegram.org/bots/api#setwebhook
[ngrok]: https://ngrok.com/
[payments-step-by-step]: https://core.telegram.org/bots/payments#step-by-step-process
[src]: https://github.com/klappvisor/orly-bookstore-bot
