# Introduction
Welcome to the wonderful world of Discord bots.  Through creating my own Discord bot, I have put together this basic framework to enable the process.

The purpose of this project is to make it so that anyone can spin up their own simple Discord bot fairly easily. Simply follow the below instructions and you'll be on the way to your own Discord bot in no time.

This project provides a basic wrapper around functionality presented by [discord.js](https://github.com/hydrabolt/discord.js) project.

# Setup
## Load the Module
The first step to get started is to load the scaffolding.
```
const DiscordBot = require('discordbot-framework');
let bot = new DiscordBot();
```

## Add the configuration
Configuration for the bot is provided in the form of a basic object.

How you decide to get the object is at your discretion.
```
const config = {
    'secret_key' : 'my_discord_provided_secret_key'
};
bot.configure(config);
```
The only option that is required for configuration is `secret_key` but the full list of possible options is as follows:

|Configuration Key|Data Type|Default Value|Note|
|---:|:---:|:---:|:---|
|`secret_key`|integer|_(none)_|The client secret key/token provided on the [Discord Developer](https://discordapp.com/login?redirect_to=/developers/applications/me) page. The bot will fail to boot without this.|
|`command_prefix`|string|'!'|The prefix used for commands, e.g. `!syn|
|`allowed_channels`|array|[ ]|The channels that the bot is allowed to respond in; an empty array means all channels
|`respond_to_bots`|boolean|false|Whether or not the bot is allowed to respond to other bots|
_Note: If there is no default value, the framework will throw an Error if one isn't specified_

## Configure the Event Listeners
We can add event listeners to the bot.
```
bot.observe('message', (msg) => console.log(`${msg.author.username} sent a message in #${msg.channel.name}`));
```
The observe function takes a string for the first parameter, where the string is one of the events defined by the discordjs `Client`. The second parameter is the callback to fire when the event is triggered.

_Note: You can refer to discord.js's Client API documentation [here](https://discord.js.org/#/docs/main/stable/class/Client) for the supported events_

Two event listeners are added automatically as part of the framework; one for `'ready'` as it's required for `discord.js` to start, and the other for `message` which handles processing commands.
As event listeners can be added multiple times for the same event, these two event listeners should not affect the code you write for the bot.

## Add commands
We can also add commands to the bot.

```
bot.bind('syn', { 
    'callback' : (msg) => msg.channel.sendMessage('Ack!') 
});
```
The bind function takes in two parameters.
- The first argument is the name of the command (e.g., `syn` => `!syn`).
- The second argument is an object with the required parameters for the command.

|Parameter|Data Type|Default Value|Note|
|---:|:---:|:---:|:---|
|callback|function|_(none)_|The function to call when the command is called.|
|rate_limit|integer|3|The number of times per minute the command can be called by a user.|
_Note: If there is no default value, the system will Error if one isn't specified_

## Schedule Events
We can schedule functions to run at specific times. 

This is convenient if we want something to happen on a specific schedule.
```
bot.schedule({
    'name'      : 'server-list', 
    'frequency' : 'hourly',
    'callback'  : (instance) => {
        let servers = instance.getGuilds().reduce((list, guild) => { list.push(guild.name + "|" + guild.id); return list; }, []);
        console.log('I am connected to the following servers: ' + servers.join(', '));
    }
});
```
By default, the parameter sent in to the callback is a reference to the framework itself, but this can be specified as one of the parameters as seen in the below (fairly useless) example.
```
// Create some data we want to send in
let days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'];

bot.schedule({
    'name'      : 'hourly-notice', 
    'frequency' : 'hourly',
    'start_of'  : 'hour',
    'context'   : {bot, days}, // Short hand object notation, we still want to send the bot instance, but we want to also send in the data we created
    'callback'  : (context) => {
        context.bot.getGuilds().first().defaultChannel.sendMessage(`Hello! I am only available on the following days: ${context.days.join(', ')}`);
    }
});
```

|Parameter|Data Type|Default Value|Note|
|---:|:---:|:---:|:---|
|name|String|_(none)_|**(Required)** The name of the task for scheduling purposes. Names must be unique.|
|frequency|String|_(none)_|**(Required)** The timeframe for which to fire the event; see the supported schedules table below.|
|callback|function|_(none)_|**(Required)** The callback to trigger on the schedule.|
|begin_at|String/momentjs Timestamp|now|A timestamp at which point to start this task; can be string or momentjs instance.|
|start_of|String|_(none)_|This is used to jump your task the start of the next schedule. e.g., `hour` means start of next hour, start at `3:44 -> 4:00 -> 5:00`.  If omitted, it will just schedule for the next increment. e.g., `3:44 -> 4:44 -> 5:44`.|
|context|Any|Framework|This is the value that will be passed into the callback parameter, the default is the instance of the framework but this would allow you to pass in anything.|
|immediate|Boolean|false|This will fire the function once before scheduling it|
|once|Boolean|false|Whether or not to reschedule the task after it has run the first time (not including the `immediate` run, so `once` + `immediate` = two executions)|

The following frequencies are defined as within the limitations of NodeJS's `setTimeout` / `setInterval` maximum supported delay.
|Frequency|Definition|
|----:|:----|
|deciminute|Every ten seconds*|
|minute|Every minute|
|hourly|Every hour|
|daily|Every day|
|weekly|Every 7 days|
|biweekly|Every 14 days|
_* `deciminute` was created for testing, but the option was left because there's probably a use case for it. Highly, highly, highly recommend **AGAINST** hitting the Discord API every ten seconds._

The following `start_of` options are supported.
|`start_of` Options|
|:---:|
|year|
|month|
|quarter|
|week|
|isoweek|
|day|
|date|
|hour|
|minute|
|second|
_This is handled using the `momentjs` `startOf` function. For examples of what specifically these options mean, see the [MomentJS documentation](http://momentjs.com/docs/#/manipulating/start-of/) regarding the function._


## Connect!
Now that our bot is configured, has it's listeners, and commands added, we can start up the bot.
```
bot.connect();
```
And if everything went according to plan, your bot should log in to Discord successfully.

# Full Example
Here is a working example bot that was set up using the framework.
```
// Load the module
const DiscordBot = require('discordbot-framework');
let bot = new DiscordBot();

// Get the configuration
// Please never ever commit your secret key to a git repository
// See DotEnv in the references
const configData = {
    'secret_key'        : 'thisisreallynotasecrettoken',
    'command_prefix'    : '@',
    'respond_to_bots'   : true
};

// Load the configuration into the bot
bot.configure(configData);

// Add a command
bot.bind('syn', {
    'callback': msg => msg.channel.sendMessage('Ack!'),
    'rate_limit': 1
});

// Add a listener
bot.observe('guildMemberAdd', (guildMember) => {
    const nickname = guildMember.nickname || guildMember.user.username;
    guildMember.guild.defaultChannel.sendMessage(`Welcome to the ${guildMember.guild.name} party, ${nickname}!`);
});

// Add an event to the schedule
bot.schedule({
    'name'      : 'server-list', 
    'frequency' : 'hourly',
    'callback'  : (instance) => {
        let servers = instance.getGuilds().reduce((list, guild) => { list.push(guild.name + "|" + guild.id); return list; }, []);
        console.log('I am connected to the following servers: ' + servers.join(', '));
    }
});


// Tell the bot to connect to Discord
bot.connect();
```

# References
- [Discord.js Documentation](https://discord.js.org/#/docs/main/stable/general/welcome)
- [momentJS](http://momentjs.com/)
- [NodeJS DotEnv](https://www.npmjs.com/package/dotenv)

# Change Log
## v1.1.1
- Fix an issue in which the bot would crash if someone used a command that didn't exist

## v1.1.0
- Support for scheduling tasks to run on rotations!

## v1.0.1
- Fixed some derps in the README file

## v1.0.0
- Initial Release!
- Support for:
  - Adding commands
  - Listening for events