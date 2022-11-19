---
layout: ../../layouts/BlogLayout.astro
title: Writing a Discord Bot in TypeScript
description: In this article, we will be writing a Discord bot in TypeScript. We will be using the discord.js library to create slash commands and handle events.
date: 2022-11-19:13:06
---

# In this article we will

- Create a Discord application on the Discord Developer Portal
- Setup a blank TypeScript project
- Setup the discord.js library to process slash commands and events
- Create a "module"
- Add commands to the module
- Add events to the module

## Who is this article for?

Before proceeding, you should have a good understanding of JavaScript and TypeScript. Basic knowledge of how Discord works in general is also recommended.

Basically, I'm aiming for people who have written a fair bit amount of TypeScript, but are new to Discord bots.

If you're new to TypeScript and JavaScript, I recommend you learn JavaScript and do a few projects before proceeding. Then you can learn TypeScript and do a few more projects. If you do all of that, you're worthy (lol) of reading this article.

discord.js is a fairly complex libary, and if you're just a begginer, it might trip you up a lot. That's why I recommend you to do other projects first.

## Code

All of the code for this article can be found on [GitHub](https://github.com/marzeq/djs-tut-example). Each commit represents a step in the tutorial.

## Creating a Discord application

First, head over to the [Discord Developer Portal](https://discord.com/developers/applications).

Now, click on the "New Application" button.

![New Application](/discord-bot-typescript/01.png)

Give your application a name, agree to the terms, and click "Create".

Now, click on the "Bot" tab on the left side of the screen. From there, click on the "Add Bot" button and confirm.

You will now see a "Token" section. Click on the "Reset Token" button and confirm. You may need to enter your 2FA code if you have it enabled.

![Copy token](/discord-bot-typescript/02.png)

Copy the token and save it somewhere safe. You will need it later.

**IMPORTANT**: The token **ACTS LIKE A PASSWORD FOR YOUR BOT. DO NOT SHARE IT WITH ANYONE!** If you accidentally share it, you can reset it by clicking on the "Reset Token" button.

From here, go to the "OAuth2" tab on the left side of the screen and go into the "URL Generator" section. Check the "bot" checkbox and the "applications.commands" checkbox. You can then select the permissions you want your bot to have. For this tutorial, we will use the "Administrator" permission for simplicity. For deployment, you should only give your bot the permissions it needs.

![Authorize bot](/discord-bot-typescript/03.png)

Copy the generated URL and open it in your browser. Select the server you want to add your bot to and click "Authorize". This process is the same as adding any other bot to your server.

Boom! Your bot should now be in your server.

## Setting up the project

Now that we have our Discord application set up, we can start working on our bot. We will be using TypeScript for this tutorial, as mentioned in the title.

If you have your own TypeScript template, you can use your own. If not, you can use the one I made. You can find it [here](https://github.com/marzeq/typescript-template).

Clone the repository and install the dependencies.

```bash
git clone https://github.com/marzeq/typescript-template my-bot # Replace "my-bot" with the name of your desired project
# you can also use your own template if you want
cd my-bot
rm -rf .git # remove the template's git repository
git init # initialize a new git repository
yarn # install dependencies
# you can also use npm if you want

# optional: make a commit
git add .
git commit -m "Initial commit"
```

Edit the `package.json` file and change the values to your liking.

In the `src/index.ts` file, remove the `console.log` line (why would it need be there in the first place?).

You will now have a blank TypeScript project set up. We can now start working on the bot itself.

## Setting up discord.js

Now that we have our project set up, we can start working on our bot. We will be using the [discord.js](https://discord.js.org/) library to control our bot.

Install the library and other dependencies.

```bash
yarn add discord.js dotenv # or npm install
```

Now, create a new file called `.env` in the root of your project. This file will contain the token for your bot. It is a good practice to store sensitive information in environment variables.

Place your token into the file like this:

```
TOKEN="[your token]"
```

Replace `[your token]` with your token. The quotes are optional, but I recommend using them.

Edit the `src/index.ts` file and add the following code:

```ts
import { Client, Collection, Awaitable, ChatInputApplicationCommandData, ChatInputCommandInteraction } from "discord.js"
import dotenv from "dotenv"

dotenv.config()

class Bot extends Client {
  public readonly commands = new Collection<string, Command>()
}

export type BotType = Bot

export interface Command extends ChatInputApplicationCommandData {
  run: (interaction: ChatInputCommandInteraction) => Awaitable<any>
}

const main = async () => {
  const client = new Bot({
    intents: ["Guilds"]
  })

  if (!("TOKEN" in process.env)) {
    throw new Error("No token provided")
  }

  client.login(process.env.TOKEN)
}

main().catch(console.error)
```

This is the most basic setup for a Discord bot. We are creating a class that extends the `Client` class from the discord.js library. We are also creating a `commands` property that will store all of our commands. Then we just check if the token is defined in the environment variables and log in.

For more complex bots, you can add more properties to the `Bot` class and mess around with other things in here.

## Creating a module

So, you might be asking: "What the heck is a module?". A module is my own term to describe a group of commands and events that are related to each other. For example, you could have a "moderation" module that contains commands like `ban`, `kick`, `mute`, etc. You could also have a "fun" module that contains commands like `meme`, `8ball`, etc.

This term is not used in the discord.js library or community, but I find it useful to organize my code that way.

For this simple setup, a module will just be a function that takes the client as it's only argument. This function will be called when the bot class is instantiated, but before the bot logs in.

### Setting up module support

The first decision we need to make is how we want to load our modules. We can either explicitly import them in the `src/index.ts` file or we can load them automatically from a directory.

Personally I prefer the first option, as the `import(...)` function when given a string will give us the type, so we are 100% type safe. The second option is also fine, but it does not guarantee type safety.

For this tutorial, we will also be using the first option.

Append the following code to the `src/index.ts` file just above the `main` function:

```ts
export type Module = Promise<{
  default: (client: Bot) => Awaitable<any>
} | {
  module: (client: Bot) => Awaitable<any>
}>

const loadModules = async (modules: Module[], client: Bot) => {
  for (const module of await Promise.all(modules)) {
    if ("default" in module) {
      module.default(client)
    } else {
      module.module(client)
    }
  }
}
```

Now, in the `main` function below the `client` declaration, add the following code:

```ts
await loadModules([
  import("./modules/example"),
], client)
```

This will load the `example` module. Now we have to create the module.

### Creating the module

Create a new file called `src/modules/example.ts`. This will be our example module.

Add the following code to the file:

```ts
import { BotType } from "../index"

export const module = async (client: BotType) => {
  console.log("Example module loaded")
}
```

Now, if you run the bot, you should see the following message in the console:

```
Example module loaded
```

That's great! We have successfully loaded our first module.

## Creating a command

### Setting up command support

Before we can create our first command, we need to set up the command support. For the sake of this tutorial, all our commands will be global. You can also make them guild-specific, but that is out of the scope of this tutorial.

Add a `start` method to the `Bot` class in the `src/index.ts` file:

```ts
public async start() {
  await this.application?.commands.set(Array.from(this.commands.values()))

  this.on("interactionCreate", async interaction => {
    if (!interaction.isChatInputCommand()) return

    const command = this.commands.get(interaction.commandName)

    if (!command) {
      await interaction.reply({
        content: "Command not found",
        ephemeral: true,
      })
      return
    }

    try {
      await command.run(interaction)
    } catch (error) {
      console.error(error)
      await interaction.reply({ content: "There was an error while executing this command!", ephemeral: true })
    }
  })

  await this.login(process.env.TOKEN)
}
```

Now replace the `client.login` call in the `main` function with `client.start()`.

Great! Now we can start creating commands.

### Creating the command

Now that we have our module and command support set up, we can start working on our first command. We will be creating a simple ping command.

In the module's function, add the following code:

```ts
client.commands.set("ping", {
  name: "ping",
  description: "Pong!",
  run: async interaction => {
    await interaction.reply("Pong!")
  }
})
```

### Testing the command

If everything went well, you should now be able to execute the command in your Discord server.

Go to your Discord server and type `/ping`. You should see the bot reply with `Pong!`.

## Creating events

Now that we have our first command, we can start working on events. Events are triggered when something happens in the Discord API. For example, when a message is sent, the `messageCreate` event is triggered.

The `messageCreate` event is a priveleged intent, so we will use another event for this tutorial. We will be using the `reactionAdd` event instead.

In the module's function, add the following code:

```ts
client.on("reactionAdd", async (reaction, user) => {
  if (reaction.emoji.name !== "👍") return
  if (user.bot) return // important to prevent infinite loops

  await reaction.message.react("👍")
})
```

### Specifying intents

If you try to run the bot now, you notice that it does not work. This is because we are not specifying the intents that we want to use. We can do this by adding the following code to the client options in the `client` declaration in the `src/index.ts` file:

```ts
  intents: ["Guilds", "GuildMessageReactions"]
```

If you are confused about what intents are, you can read more about them [here](https://discordjs.guide/popular-topics/intents.html).

### Testing the event

If everything went well, you should now be able to test the event in your Discord server.

Go to your Discord server and add a 👍 reaction to a message. You should see the bot add another 👍 reaction to the message.

## More advanced stuff

If you want to do more advanced things, like adding command autocomplete, I will leave that as an exercise for you.

A few useful resources to learn what's possible:
- [The official Discord API documentation](https://discord.com/developers/docs/intro)
- [The discord.js guide](https://discordjs.guide/)
- [The discord.js documentation](https://discord.js.org/#/docs/main/stable/general/welcome)

If you prefer to piggyback off of my code, stay tuned, because I will also be making a more advanced tutorial in the future, along with a GitHub template repository.

## Conclusion

That's it for this tutorial. I hope you learned something new. If you have any questions, feel free to contact me through Twitter or Discord. Contact links are on my [main website](https://marzeq.codes/).

If you need help with discord.js, the guys have [their own Discord server](https://discord.gg/bRCvFy9) where you can ask questions.

## Resources

- [Official Discord API documentation](https://discord.com/developers/docs/intro)
- [discord.js documentation](https://discord.js.org/#/docs/main/stable/general/welcome)
- [discord.js Discord server](https://discord.gg/bRCvFy9)
- [discord.js GitHub repository](https://github.com/discordjs/discord.js)
- [My TypeScript template repository](https://github.com/marzeq/typescript-template)
