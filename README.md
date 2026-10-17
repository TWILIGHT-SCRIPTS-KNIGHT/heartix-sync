# heartix-sync

Tiny relay repo for **HeartixOSC**'s chatbox sync feature.

The host app commits a small JSON file (`heartix-sync.json`) to this repo
whenever it pushes a message. Every follower polls that file and mirrors the
message into their own VRChat chatbox. Because it rides on GitHub's API,
**it works internationally with no port forwarding, firewall rules, or public
IP** — all you need is a public repo.

## How it works

| Role       | What it does                                                        |
| ---------- | ------------------------------------------------------------------- |
| **Host**   | `PUT`s the latest message to `heartix-sync.json` (needs a token).   |
| **Follower** | conditional-`GET`s the file every few seconds and sends new ones. |

Follower polls use `ETag` conditional requests, so an unchanged file returns
`304 Not Modified` and **does not count against the GitHub rate limit**. A
follower on a *public* repo therefore needs **no token at all**.

## `heartix-sync.json` format

```json
{
  "Seq": 26,
  "Text": "the message everyone should see",
  "Sender": "host",
  "TimestampMs": 1760000000000
}
```

- `Seq` – monotonic id (the app uses the push timestamp).
- `Text` – the exact chatbox string (≤144 chars once framed).
- `Sender` – who pushed it.
- `TimestampMs` – Unix ms; followers only send messages newer than the last one they saw.

You normally never edit this file by hand — the app owns it.

## Setup

### 1. Create the token (host only)

1. GitHub → **Settings → Developer settings → Personal access tokens**.
2. Create a **fine-grained token** limited to this repo with
   **Contents: Read and write**.
3. Paste it into HeartixOSC → **Sync → GitHub relay → Token**.

### 2. Host

In **HeartixOSC → Sync → GitHub relay**:

- tick **Publish pushes to GitHub (host)**
- fill **Repo** (`TWILIGHT-SCRIPTS-KNIGHT/heartix-sync`), **Branch** (`main`)
  and **File** (`heartix-sync.json`)
- paste the token
- click **Apply GitHub**, then type a message and hit **Push**

Pushing from inside the app to the repo owner also works with the **Create
repo** button if the repo does not exist yet.

### 3. Follower

Tick **Follow GitHub pushes (follower)** and set the same **Repo / Branch /
File**, then **Apply GitHub**. No token is required for a public repo.

## Privacy

Anyone who can read this repo can read every message you push, including its
history (Git history keeps old messages). Use a **private** repo + a read token
for followers if you want the traffic to stay private.
