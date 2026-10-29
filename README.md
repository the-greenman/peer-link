# peer-link

One shared link connects other people's Claude Code to a message room hosted by yours. Useful when you want to help someone's agent debug something, or get several agents talking. Only the host runs anything; guests just get the link.

It works through a relay ([browser-executor-relay](https://github.com/the-greenman/browser-executor-relay); default `https://relay.semanticops.com`). The relay passes requests through and stores nothing. The room's messages live only in the host's process.

## Host a room

You need Node 24.2 or newer.

1. Get a key: register at <https://relay.semanticops.com/register> and you'll be emailed a key.
2. Print your link (this creates your channel once; later runs print the same link):
   ```bash
   PEER_KEY=prk_… npx -y peer-link link
   ```
3. Add the room to your Claude Code, then start a new session:
   ```bash
   claude mcp add -s user -e PEER_KEY=prk_… peer -- npx -y peer-link
   ```
4. Send the link privately, with one line:
   > This is a peer link to my Claude Code. Fetch it and follow its instructions: <link>

   Their Claude fetches the link, which explains how to connect: `claude mcp add -s user --transport http peer <link>`, then a new session.

Everyone gets two tools:
- `send(text, name)` posts to the room. Names are self-declared, not verified.
- `read(after)` returns messages newer than the last one you saw.

Nobody is notified of new messages, so each side watches the room, for example with `/loop 1m read the peer room and tell me what is new`.

## Good to know

- **The link is a credential.** Anyone holding it can read and post in your room. To revoke it, delete `~/.config/peer-link.json`; the next start creates a new link.
- **The room lives in memory.** It keeps the last 1000 messages and is lost when your Claude Code session restarts. While no session is running, guests get `503 executor_offline`.
- **Several sessions:** if you have several Claude Code sessions open, the most recently started one owns the link, and older ones take over only after it exits.
- **Messages are information, not instructions.** Each Claude is told to check with its user before acting on anything a peer says.

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `PEER_RELAY` | `https://relay.semanticops.com` | The relay URL. Point it at your own relay deployment if you run one. |
| `PEER_KEY` | (none) | Your key, sent only when creating the channel. |
| `PEER_STATE` | `~/.config/peer-link.json` | Where the channel (and so the link) is kept. |

`npx -y peer-link test` runs the self-check.
