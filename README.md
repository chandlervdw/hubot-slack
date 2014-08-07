A fork of [hubot-slack](https://github.com/tinyspeck/hubot-slack) adapter for Hubot, with some useful scripts referenced and slightly modified instructions.

--------

# hubot-slack

This is a [Hubot](http://hubot.github.com/) adapter to use with [Slack](https://slack.com).  
[![Build Status](https://travis-ci.org/tinyspeck/hubot-slack.png)](https://travis-ci.org/tinyspeck/hubot-slack)

## Getting Started

#### Creating a new bot

- `npm install -g hubot coffee-script`
- `hubot --create slackbot`
- `cd slackbot`
- `npm install hubot-slack --save`
- Initialize git and make your initial commit
- Check out the [hubot docs](https://github.com/github/hubot/tree/master/docs) for further guidance on how to build your bot

#### Deploying to Heroku

This is a modified set of instructions based on the [instructions on the Hubot wiki](https://github.com/github/hubot/blob/master/docs/deploying/heroku.md).

From the same directory (above):

- `npm install` to install the dependencies listed in `package.json`
- Install [heroku toolbelt](https://toolbelt.heroku.com/) if you haven't already.
- `heroku create my-company-slackbot`
- `heroku addons:add redistogo:nano`
- Activate the Hubot service on your ["Team Services"](http://my.slack.com/services/new/hubot) page inside Slack.
- Add the [config variables](#adapter-configuration). For the minimum working version:

        % heroku config:add HEROKU_URL=http://soothing-mists-4567.herokuapp.com
        % heroku config:add HUBOT_SLACK_TOKEN=dqqQP9xlWXAq5ybyqKAU0axG
        % heroku config:add HUBOT_SLACK_TEAM=myteam
        % heroku config:add HUBOT_SLACK_BOTNAME=hubot

- Deploy and start the bot:

        % git push heroku master
        % heroku ps:scale web=1

- Profit!

- To get some of the other scripts working, you'll want the following config variables, just like you did above. Some of these, like `BREWERYDB_API_KEY`, need you to sign up for services and request API access. If a particular command for hubot returns an error, see the individual scripts I've referenced in `hubot-scripts.json` and look them up in the [hubot scripts catalogue](http://hubot-script-catalog.herokuapp.com).

        % heroku config:add BREWERYDB_API_KEY=your_key_here
        % heroku config:add HEROKU_URL=your_key_here
        % heroku config:add HUBOT_ANNOUNCE_ROOMS=your_key_here   // these are the room ids
        % heroku config:add HUBOT_GIPHY_API_KEY=your_key_here
        % heroku config:add HUBOT_MEMEGEN_PASSWORD=your_key_here
        % heroku config:add HUBOT_MEMEGEN_USERNAME=your_key_here
        % heroku config:add IMGUR_CLIENT_ID=your_key_here
        % heroku config:add WORDNIK_API_KEY=your_key_here

## Adapter configuration

This adapter uses the following environment variables:

#### HUBOT\_SLACK\_TOKEN

This is the service token you are given when you add Hubot to your Team Services.

#### HUBOT\_SLACK\_TEAM

This is your team's Slack subdomain. For example, if your team is `https://myteam.slack.com/`, you would enter `myteam` here.

#### HUBOT\_SLACK\_BOTNAME

Optional. What your Hubot is called on Slack. If you entered `slack-hubot` here, you would address your bot like `slack-hubot: help`. Otherwise, defaults to `slackbot`.

#### HUBOT\_SLACK\_CHANNELMODE

Optional. If you entered `blacklist`, Hubot will not post in the rooms specified by HUBOT_SLACK_CHANNELS, or alternately *only* in those rooms if `whitelist` is specified instead. Defaults to `blacklist`.

#### HUBOT\_SLACK\_CHANNELS

Optional. A comma-separated list of channels to either be blacklisted or whitelisted, depending on the value of HUBOT_SLACK_CHANNELMODE.

#### HUBOT\_SLACK\_LINK\_NAMES

Optional. By default, Slack will not linkify channel names (starting with a '#') and usernames (starting with an '@'). You can enable this behavior by setting HUBOT_SLACK_LINK_NAMES to 1. Otherwise, defaults to 0. See [Slack API : Message Formatting Docs](https://api.slack.com/docs/formatting) for more information.

## Under the Hood

#### Receiving Messages:

The slack adapter adds a path to the robot's router that will accept POST requests to:

`/hubot/slack-webhook`

Source: [https://github.com/tinyspeck/hubot-slack/blob/2.1.0/src/slack.coffee#L149-L165](https://github.com/tinyspeck/hubot-slack/blob/2.1.0/src/slack.coffee#L149-L165)

Expected parameters:

- text
- user_id
- user_name
- channel_id
- channel_name

If there is a message and it can deduce an author from those paramters, it'll create a new [TextMessage](https://github.com/github/hubot/blob/v2.7.2/src/message.coffee#L14) object and have the robot receive it, from there proceeding down the regular hubot path.

#### Sending Messages

When a script calls `send()` or `reply()` this adapter makes a POST request to your team's specific URL webhook:

`https://<your_team_name>.slack.com/services/hooks/hubot`

with a JSON-formatted body including the following dictionary:

- username
- channel
- text
- link_names (optionally)
