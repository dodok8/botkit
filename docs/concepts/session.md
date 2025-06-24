---
description: >-
  The Session object is a short-lived object that actively communicates with
  the fediverse.  Learn how to create a session and publish messages to the
  fediverse.
---

Session
=======

The `Session` object is a short-lived object that actively communicates with
the fediverse.  It can be [created by yourself](#creating-a-session),
or you can get it when an event handler is called.


Creating a session
------------------

You can create a session by calling the `Bot.getSession()` method:

~~~~ typescript twoslash
import type { Bot } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
const session = bot.getSession("https://mydomain");
~~~~

It takes a single argument, the origin of the server to which your bot belongs.
In practice, you would have an environment variable that contains the hostname
of your server, and you would pass it to the `~Bot.getSession()` method:

::: code-group

~~~~ typescript [Deno] twoslash
import type { Bot } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
const SERVER_NAME = Deno.env.get("SERVER_NAME");
if (SERVER_NAME == null) {
  console.error("The SERVER_NAME environment variable is not set.");
  Deno.exit(1);
}

const session = bot.getSession(`https://${SERVER_NAME}`);  // [!code highlight]
~~~~

~~~~ typescript [Node.js] twoslash
import type { Bot } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
const SERVER_NAME = process.env.SERVER_NAME;
if (SERVER_NAME == null) {
  console.error("The SERVER_NAME environment variable is not set.");
  Deno.exit(1);
}

const session = bot.getSession(`https://${SERVER_NAME}`);  // [!code highlight]
~~~~

:::


Getting a session from an event handler
---------------------------------------

When an event handler is called, you can get a session from the `Session`
object that is passed as the first argument:

~~~~ typescript twoslash
import type { Bot } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
bot.onMention = async (session, message) => {
  // `session` is a `Session` object
};
~~~~

To learn more about event handlers, see the [*Events* section](./events.md).


Determining the actor URI of the bot
------------------------------------

The `Session` object has an `actorId` property that contains the URI of the bot
actor.  You can use this URI to refer to the bot in messages:

~~~~ typescript twoslash
import { type Bot, text } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
bot.onFollow = async (session, actor) => {
  await session.publish(
    text`Hi, ${actor}! I'm ${session.actorId}. Thanks for following me!`
  );
};
~~~~


Determining the fediverse handle of the bot
-------------------------------------------

The `Session` object has an `actorHandle` property that contains the fediverse
handle of the bot.  It looks like an email address except that it starts with
an `@` symbol: `@myBot@myDomain`.  You can use this handle to refer to the bot
in messages:

~~~~ typescript twoslash
import type { Bot } from "@fedify/botkit";
import { markdown } from "@fedify/botkit/text";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
bot.onFollow = async (session, actor) => {
  await session.publish(
    markdown(`I'm ${session.actorHandle}. Thanks for following me!`)
  );
};
~~~~


Getting the bot's `Actor` object
--------------------------------

The `Session` object has a `~Session.getActor()` method that returns the `Actor`
object of the bot:

~~~~ typescript twoslash
import type { Actor, Session } from "@fedify/botkit";
const session = {} as unknown as Session<void>;
// ---cut-before---
const actor: Actor = await session.getActor();
~~~~


Publishing a message
--------------------

See the [*Publishing a message* section](./message.md#publishing-a-message)
in the *Message* concept document.


Getting published messages
----------------------

See the [*Getting published messages*
section](./message.md#getting-published-messages) in the *Message* concept
document.


Following an actor
------------------

Your bot can follow an actor by calling the `Session.follow()` method.
The following example shows how to get the `bot` follow back all of its
followers:

~~~~ typescript twoslash
import type { Bot } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
bot.onFollow = async (session, followRequest) => {
  await followRequest.accept();
  await session.follow(followRequest.follower);
};
~~~~

> [!CAUTION]
> The `~Session.follow()` method just sends a follow request to the actor,
> but it does not guarantee that the actor will accept the follow request.
> The actor may reject the follow request, and your bot will not be able to
> follow the actor.
>
> If you want to know whether the actor has accepted or rejected the follow
> request, you need to register
> the [`Bot.onAcceptFollow`](./events.md#accept-follow) and
> [`Bot.onRejectFollow`](./events.md#reject-follow) event handlers.

> [!TIP]
> It takes several kinds of objects as an argument, such as `Actor`, `string`,
> and `URL`:
>
> `Actor`
> :   The actor to follow.
>
> `URL`
> :   The URI of the actor to follow.
>     E.g., `new URL("https://example.com/users/alice")`.
>
> `string`
> :   The URI or the fediverse handle of the actor to follow.
>     E.g., `"https://example.com/users/alice"` or `"@alice@example.com"`.

> [!NOTE]
> If you try to follow an actor that is already followed, the method will just
> do nothing.


Unfollowing an actor
--------------------

Likewise, your bot can unfollow an actor by calling the `Session.unfollow()`
method.  The following example shows how to make the `bot` unfollow if any of
its followers unfollow it:

~~~~ typescript twoslash
import type { Bot } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
bot.onUnfollow = async (session, actor) => {
  await session.unfollow(actor);
};
~~~~

> [!TIP]
> Like the `~Session.follow()` method, the `~Session.unfollow()` method takes
> several kinds of objects as an argument, such as `Actor`, `string`, and `URL`.

> [!NOTE]
> If you try to unfollow an actor that is not followed, the method will just
> do nothing.


Checking if the bot follows an actor
------------------------------------

The `Session` object has a `~Session.follows()` method that returns a boolean
value indicating whether your bot follows a given actor.  The following example
shows how to check if your bot follows an actor and respond accordingly:

~~~~ typescript twoslash
import { type Bot, text } from "@fedify/botkit";
const bot = {} as unknown as Bot<void>;
// ---cut-before---
bot.onMention = async (session, message) => {
  const follows = await session.follows(message.actor);
  await session.publish(
    follows
      ? text`Hi ${message.actor}, I'm already following you!`
      : text`Hi ${message.actor}, I don't follow you yet.`
  );
};
~~~~

> [!TIP]
> Like other methods, `~Session.follows()` accepts several types of arguments
> such as `Actor`, `string`, and `URL`.

> [!NOTE]
> This method returns `false` if the given actor doesn't exist or is
> inaccessible.
