---
title: Discord Mod Bot
app_type: discord-mod-bot
wallet: 0xC9eeFDCB8Ac773A5bA168817fa637f741b942613
---

## premise

A discord bot that you drop into a server, grant manage messages and ban members on, then walk away. It handles the boring half of moderation on its own, and the slash commands cover the rest. No web dashboard, no premium tier, no telemetry, no opinion about your community.

## stack

Node 20. `discord.js` v14.16. TypeScript 5.6 with `strict` and `exactOptionalPropertyTypes` on. SQLite via `better-sqlite3` for state. `esbuild` for the production bundle. `tsx` for the dev loop. `vitest` for tests. No express, no redis, no http server at all. The process is a long lived gateway client and nothing else.

## rules that run on their own

The bot watches every message in every channel it can see. Three rules trigger automatically.

First, link control. A regex spots urls. If the author has been a server member for less than seven days and the link points outside an allow list set per server, the message gets deleted and a quiet note lands in the mod log channel. The allow list lives in the config row for that guild.

Second, mention flood. If one message tags more than five users or roles, it gets deleted and the author receives a one minute timeout. Five is the default, configurable to anywhere between three and twenty per server.

Third, raid detection. If more than ten accounts join in under thirty seconds, the bot raises the server verification level temporarily and posts an alert with the join timestamps in mod log. After fifteen minutes of quiet, verification level returns to the prior value.

Each rule is a single file in `src/automod`, each exports a name, a trigger condition, and an action. Adding a fourth rule is one new file and one import.

## slash commands

```
/warn  user reason         log a warning, dm the user, send to mod log
/mute  user minutes reason apply a timeout, dm the user, send to mod log
/kick  user reason         kick, dm before, send to mod log
/ban   user days reason    ban with message delete window, send to mod log
/unban user reason         lift a ban, send to mod log
/case  id                  fetch one moderation case by id
/cases user                list all cases for a user, paginated
/note  user text           add a private note, never dm to user
/config show               dump current guild config
/config set key value      update one config key
```

Every mod action writes a case row. The row holds id, guild id, target user id, mod user id, action, reason, duration, created at. Id is auto incremented per guild. The case command uses the id to fetch and pretty print.

## data shape

Three sqlite tables and nothing else.

```sql
create table guilds (
  guild_id        text primary key,
  mod_log_id      text,
  mention_max     integer default 5,
  link_age_days   integer default 7,
  allowed_domains text default '[]'
);

create table cases (
  id          integer primary key autoincrement,
  guild_id    text not null,
  target_id   text not null,
  mod_id      text not null,
  action      text not null,
  reason      text,
  duration    integer,
  created_at  integer not null
);

create table notes (
  id         integer primary key autoincrement,
  guild_id   text not null,
  target_id  text not null,
  author_id  text not null,
  body       text not null,
  created_at integer not null
);
```

A single `migrations/` folder holds versioned sql files. The runner is fifteen lines in `src/db/migrate.ts`. No knex, no prisma, no drizzle.

## permissions model

Every command checks the invoker has the matching discord permission. `/ban` requires ban members. `/kick` requires kick members. `/mute` and `/warn` require moderate members. `/config` requires manage guild. If the bot itself lacks the permission to perform the action, the response explains exactly which permission is missing and which role on the bot needs it.

## file layout

```
src/
  index.ts            client setup, login, graceful shutdown
  config.ts           env parsing, type safe
  commands/
    warn.ts
    mute.ts
    kick.ts
    ban.ts
    unban.ts
    case.ts
    cases.ts
    note.ts
    config.ts
    register.ts       posts the commands to discord on startup
  automod/
    links.ts
    mentions.ts
    raids.ts
    index.ts          registers the listeners
  db/
    schema.ts
    migrate.ts
    queries.ts
  mod_log.ts          builds and posts the embed
  errors.ts           friendly error replies
tests/
  warn.test.ts
  ban.test.ts
  automod_links.test.ts
  raid.test.ts
migrations/
  001_init.sql
package.json
tsconfig.json
.env.example
README.md
```

## logging and observability

A single `pino` logger streams ndjson to stdout. Every command, every automod trigger, every gateway event of note gets one log line with the guild id, user id, action, and outcome. The mod log embed is the user facing version of the same record. There is no Sentry, no OpenTelemetry, no metrics endpoint. If you want to ship logs, pipe stdout to your platform of choice.

## dev loop

```
pnpm install
cp .env.example .env  # paste your bot token
pnpm dev              # tsx watch
pnpm test             # vitest
pnpm build            # esbuild single bundle
pnpm start            # runs the built bundle
```

## deploy shape

A single Dockerfile, `node:20-alpine`, twelve lines. One process, no health check beyond the discord gateway connection, sqlite file on a volume. A systemd unit lives in `deploy/systemd.service` for a vps. A Fly.io app config lives in `fly.toml`. Pick whichever, the bot does not care.

## intentionally absent

No web dashboard. No subscription. No premium features. No multi server billing. No analytics. No anti raid honeypot channels. No verification flow with reaction roles. No music. No leveling. No starboard. Each of these is a separate bot.
