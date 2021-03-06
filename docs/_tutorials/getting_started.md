---
title: Getting started
order: 1
slug: getting-started
layout: tutorial
permalink: /tutorial/getting-started
redirect_from:
  - /getting-started
---
# Getting started

<div class="section-content">
This guide is meant to show you how to get up and running with your first Bolt app. Along the way, we’ll create a new Slack app, set up your local environment, and develop an app that listens and responds to messages from a Slack workspace.
</div> 

<div class="supporting-content">
- [Create an app](#create-an-app)
- [Tokens and installing apps](#tokens-and-installing-apps)
- [Setting up your local project](#setting-up-your-local-project)
- [Setting up events](#setting-up-events)
- [Listening and responding to a message](#listening-and-responding-to-a-message)
- [Sending and responding to actions](#sending-and-responding-to-actions)
- [Next steps](#next-steps)
</div>

---

### Create an app
First thing's first: before you start developing with Bolt, you'll want to [create a Slack app](https://api.slack.com/apps/new). 

> 💡 We recommend using a workspace where you won't disrupt real work getting done — [you can create a new one for free](https://slack.com/get-started#create).

After you fill out an App Name (_you can change it later_) and picking a workspace to install it to, hit the `Create App` button and you'll land on your app's **Basic Information** page.

This page contains all of the information about your app in addition to important credentials you'll need for development later, like the `Signing Secret` under the **App Crendentials** header. 

![Basic Information page](../assets/basic-information-page.png "Basic Information page")

Look around, add an app icon and description, and then let's start configuring your app 🔩

---

### Tokens and installing apps
First things first: Slack apps use the industry standard [OAuth to manage access to Slack's APIs](https://api.slack.com/docs/oauth). When an app is installed, you receive a token that your app uses to call various API methods. 

There are two token types your app can use: user (`xoxp`) tokens and bot (`xoxb`) tokens. User tokens allow you to call API methods on behalf of the people who are part of your workspace, and by default, your app receive an `xoxp` token associated with the person who installs the app. Bot tokens require adding a bot user to your app, which is granted a default set of permissions.

[Tokens also require one or more scopes](https://api.slack.com/docs/oauth-scopes), which define the actions the token can perform. Every API method has a corresponding scope and in order to be able to call that method, the token must have been granted that scope when the app is installed. 

You can learn more about the different token types [on our API site](https://api.slack.com/docs/token-types). The type of token your app needs depends on the actions you want it to perform. For brevity, we're going to use bot tokens for this guide.

To add a bot user, click **Bot Users** on the left sidebar and then **Add A Bot User**. Give it a display name and username and then click **Add Bot User**.

Now that you have a bot user with permission to send messages to Slack, let's install the app to your workspace.

Click **Install App** on the left sidebar and click the big **Install App to Workspace** button at the top of the page. If you've never installed a Slack app before, you'll see a screen that details what permissions this app is requesting. This is what determines the scopes that get applied to your app's OAuth token(s).

Once you authorize the installation, you'll land on the **OAuth & Permissions** page.

![OAuth Tokens](../assets/bot-token.png "OAuth Tokens")

You'll see two tokens. For now, we'll just use the `xoxb` bot token. (If you scrol down this page to the **Scopes** section, you'll see the various scopes you can add to the `xoxp` token.)

> 💡 Treat your token like a password and [keep it safe](https://api.slack.com/docs/oauth-safety). Your app uses it to post and retrieve information from your Slack.

### Setting up your local project
With the initial configuration handled, it's time to set up a new Bolt project. This is where you'll write the code that handles all of the logic for your app. One important thing to note is that Slack doesn't host your code, you do.

If you don’t already have a project, let’s create a new one. Create an empty directory and initalize a new project:

```shell
mkdir first-bolt-app
cd first-bolt-app
npm init
```

You’ll be prompted with a series of questions to describe your project, and you can accept the defaults if you aren’t picky. After you’re done, you’ll have a new `package.json` file in your directory.

Before we install the Bolt package to your new project, let's save the bot token and signing secret you generated when you configured your app. These are stored as environment variables and should not be saved in version control -- remember, your tokens have access to your Slack workspace, treat them just like you would a password.

1. **Copy your Signing Secret from the Basic Information page** and then store it in a new environment variable. The following example works on Linux and MacOS; but [similar commands are available on Windows](https://superuser.com/questions/212150/how-to-set-env-variable-in-windows-cmd-line/212153#212153).

```shell
export SLACK_SIGNING_SECRET=<your-signing-secret>
```

2. **Copy your bot (xoxb) token from the OAuth & Permissions page** and store it in another enviornment variable.

```shell
export SLACK_BOT_TOKEN=xoxb-<your-bot-token>
```

Now, lets create your app. Install the `@slack/bolt` package and save it to your `package.json` dependencies using the following command:

```shell
npm install @slack/bolt
```

Create a new file called `app.js` in this directory and add the following code:

```javascript
const { App } = require('@slack/bolt');

// Initalizes your app with your bot token and signing secret
const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET
});

(async () => {
  // Start your app
  await app.start(process.env.PORT || 3000);

  console.log('⚡️ Bolt app is running!');
})();
```

Your token and signing secret are all that's required to create your first Bolt app. Save your `app.js` file then, back at the command line, run the following:

```script
node app.js
```

Your app should let you know that it's up and running.

---

### Setting up events
Your app behaves similarly to people on your team -- it can respond to things that happen, post messages, and more. To listen to events happening in a Slack workspace (like when a message is posted or when a emoji reaction is posted to a message) you'll use the [Events API to listen for specific events](https://api.slack.com/events-api).

To enable events for your app, start by going back to your app configuration page (click on your app [from your app management page](https://api.slack.com/apps)). Click **Event Subscriptions** on the left sidebar. Toggle the switch labeled **Enable Events**. 

You'll see a text input labeled "Request URL". This Request URL is a public endpoint where Slack will send HTTP POST requests about just the events you specify.

When one of these events occurs, Slack will send your app some information about the event, like the user that triggered it and the channel it occured in. Your app will process the JSON and can then respond accordingly.

<details>
<summary markdown="0">
<h4>Using a local Request URL for development</h4>
</summary>

If you’re just getting started with your app development, you probably don’t have a publicly accessible URL yet. Eventually, you’ll want to set that up, but for now a development proxy like [ngrok](https://ngrok.com/) will do the job. We've written a separate tutorial about [using ngrok with Slack for local development](https://api.slack.com/tutorials/tunneling-with-ngrok) that should help you get everything set up.

Once you’ve installed a development proxy, run it to begin forwarding requests to a specific port (we’re using port 3000 for this example, but if you customized the port used to intialize your app use that port instead):

```shell
ngrok http 3000
```

![Running ngrok](../assets/ngrok.gif "Running ngrok")

The output should show a generated URL that you can use (we recommend the one that starts with `https://`). This URL will be the base of your request URL, in this case `https://8e8ec2d7.ngrok.io`.

---
</details>

Now you have a public-facing URL for your app that tunnels to your local machine. The Request URL that you use in your app configuration is composed of your public-facing URL combined with the endpoint your app is listening on. By default, Bolt apps listen on the `/slack/events` endpoint so our full request URL would be `https://8e8ec2d7.ngrok.io/slack/events`.

Under the **Enable Events** switch in the **Request URL** box, go ahead and paste in your URL. As long as your Bolt app is still running, your endpoint should become verified.

---

### Listening and responding to a message
Your app is now ready for some logic. Let's start by using the `message()` method that listens to messages in channels your bot user is a member of.

The following example listens to all messages that contain the word "hello" and responds with "Hey there @user!"

```javascript
const { App } = require('@slack/bolt');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET
});

// Listens to incoming messages that contain "hello"
app.message('hello', ({ message, say }) => {
  // say() sends a message to the channel where the event was triggered
  say(`Hey there <@{message.user}!`);
});

(async () => {
  // Start your app
  await app.start(process.env.PORT || 3000);

  console.log('⚡️ Bolt app is running!');
})();
```

If you restart your app, you should be able to add your bot user to a channel, say "hello", and it will respond.

This is a basic example, but it gives you a place to start customizing your app based on your end goal. Let's try something a little more interactive by sending an interactive button rather than plain text.

---

### Sending and responding to actions

To use features like buttons, select menus, datepickers, dialogs, or message actions, you’ll need to enable interactivity. Similar to how you set up your app to listen for events, you'll need to specify a URL for Slack to send the action (such as, someone clicked a button) to.

Back on your app configuration page, click on **Interactive Components** on the left side. You'll see that there's another **Request URL** box.

By default, Bolt is configured to use same endpoint for interactive components that it uses for events, so use the same request URL as above (in the example, it was `https://8e8ec2d7.ngrok.io/slack/events`). Press the **Save Changes** button in the lower right hand corner, and that's it. Your app is now all set up for interactivity!

![Configuring a Request URL](../assets/request-url-config.png "Configuring a Request URL")

Now, let's go back to your app's code and add our own interactivity. This will consist of two steps: first, your app will send a message that contains a button. Then, your app will set up a listener for when that button is clicked and do something in response.

Below, I've modified the app code we've been writing to send a message with a button instead of a just sending a string:

```javascript
const { App } = require('@slack/bolt');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET
});

// Listens to incoming messages that contain "hello"
app.message('hello', ({ message, say }) => {
  // say() sends a message to the channel where the event was triggered
  say({
    blocks: [
	    {
		    "type": "section",
        "text": {
          "type": "plain_text",
          "text": `Hey there <@{message.user}!`
        },
        "accessory": {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Click Me"
          },
          "action_id": "button_click"
		    }
	    }
    ]
  });
});

(async () => {
  // Start your app
  await app.start(process.env.PORT || 3000);

  console.log('⚡️ Bolt app is running!');
})();
```

The value inside of `say()` is now an object that contains an array of `blocks`. Blocks are the building components of a Slack message and can range from text to images to datepickers. In this case, your app will respond with a section block that includes a button as an accessory.

You'll notice in the same `accessory` object as the button, there is an `action_id`. This will act as a unique identifier for the action so your app knows what action you want to respond to.

> 💡 The [Block Kit Builder](https://api.slack.com/tools/block-kit-builder) is an easy way to prototype your interactive messages. The builder lets you, or anyone on your team, mockup what a message should look like and then generates the corresponding JSON. You can then copy and paste the message payload JSON directly into your Bolt app.

Now, if you restart your app and say "hello" in a channel your app is in, you'll see a message with a button. But if you click the button, nothing happens.

Let's add a handler to send a followup message when someone clicks the button:

```javascript
const { App } = require('@slack/bolt');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET
});

// Listens to incoming messages that contain "hello"
app.message('hello', ({ message, say }) => {
  // say() sends a message to the channel where the event was triggered
  say({
    blocks: [
	    {
		    "type": "section",
        "text": {
          "type": "plain_text",
          "text": `Hey there <@{message.user}!`
        },
        "accessory": {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Click Me"
          },
          "action_id": "button_click"
		    }
	    }
    ]
  });
});

app.action('button_click', ({ action, say }) {
  say(`@{action.user} clicked the button`);
});

(async () => {
  // Start your app
  await app.start(process.env.PORT || 3000);

  console.log('⚡️ Bolt app is running!');
})();
```

You can see that we used the `action_id` to add a listener for our button action. If you restart your app and click the button, you'll see a new message from your app that says you clicked the button.

---

### Next steps
You just built your first Bolt app! 🎉

Now that you have a basic app up and running, you can start exploring the parts of Bolt that will make your app stand out. Here are some ideas about where to look next:

* Read through the [Basic concepts](https://slack.dev/bolt#basic) to learn about the different methods and features your Bolt app has access to.

* Explore the different events your bot can listen to with the [`events()` method](https://slack.dev/bolt#event-listening). All of the events are listed [on the API site](https://api.slack.com/events).

* Bolt allows you to [call Web API methods](https://slack.dev/bolt#web-api) with the client attached to your app. There are [over 130 methods](https://api.slack.com/methods) on our API site.
