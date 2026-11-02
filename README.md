# Mudrix integration template

A starting point for your own [Mudrix](https://github.com/kapsikkum/mudrix-web) integration repository. An
integration is a short description in a JSON file (`repo.json`): which links Mudrix should turn into cards (and
which details to look up for them), which buttons and slash commands to add to the message box, which one web
service it may call, and, only if you ask for it, a widget page that runs code. The repository is signed with a key
only you hold, and readers pin that key.

## Make your own in five minutes

You need a GitHub account and Node 20 or newer.

1. **Use this template.** Press **Use this template > Create a new repository** on GitHub (public, any name).
2. **Clone it** and open a terminal in the folder:

   ```
   git clone https://github.com/<you>/<repo>
   cd <repo>
   ```

3. **Create your signing key:**

   ```
   npm run init
   ```

   This makes the key, writes its public half into `repo.json`, signs the file and prints a **fingerprint**. Note it
   down. The private key is saved in `~/.mudrix/keys/` (on Windows `C:\Users\<you>\.mudrix\keys\`): **back it up now**
   and never commit it. See [Signing](docs/signing.md).

4. **Edit `repo.json`.** It has one integration, `my-service`, with a link card and a button. Change it to yours
   (in VS Code you get completion and hints while you type):

   | part | what it does |
   | --- | --- |
   | `name`, `description`, `version` | what people read before installing (name up to 40 characters, description up to 400). Raise `version` when you change the integration |
   | `id` | never change it after publishing: it is how Mudrix recognises an installed integration |
   | `permissions.linkCards` | up to 10 rules that turn matching `https` links into a card with a button |
   | `permissions.linkDetails` | one host to look up a few fields for a card's link, such as a title and a star count |
   | `permissions.composerActions` | up to 6 buttons in the message box's menu that insert text |
   | `permissions.commands` | up to 6 slash commands that insert text, run an api action, or open the widget |
   | `permissions.api` | one host and up to 4 fixed POST requests, each shown to the person and confirmed every time |
   | `permissions.widget` | a web page of yours opened inside Mudrix |
   | `settings` | up to 8 options people can change, such as which facts a card shows (see [Settings](#settings)) |

   Use the smallest tier that does the job. Every field is documented in the
   [reference](https://github.com/kapsikkum/mudrix-integrations#reference), and
   [`github.json`](https://github.com/kapsikkum/mudrix-integrations/blob/main/examples/github.json) is a complete
   example with cards and link details.

   | tier | permissions | what the person sees |
   | --- | --- | --- |
   | Links and buttons | `linkCards`, `composerActions` | Nothing leaves their device |
   | Link details | `linkDetails` | The host and the fields shown, at install; their device asks that host when a card is shown (in encrypted rooms only when they press the card's button) |
   | Commands | `commands` | Same as links and buttons: they insert text, or trigger one of the other tiers |
   | API | `api` | The exact request, and a confirmation every time |
   | Widget | `widget` | **A red "this runs code" warning** and an explicit tick to install |

   A widget is the only tier that runs code: the page is served live by your host and can change at any time. It
   can only do what the person approves from a fixed list, and only in the one room it is opened in:
   `send.message`, `read.messages`, `read.room`, `upload.file` and `download.file`, plus up to 3 custom room event
   types of its own that you list in `events`. See [Writing a widget](docs/writing-a-widget.md) and the samples in
   [`examples/hello-widget`](examples/hello-widget) (sends a message) and
   [`examples/file-share`](examples/file-share) (sends a file and a custom event).
   The whole file may be at most 256 KB, with at most 50 integrations.

5. **Sign it:**

   ```
   npm run sign
   ```

   Run this after every edit. It first runs Mudrix's own checks on the file (offline) and refuses to sign one
   Mudrix would refuse, naming the field, for example
   `repo.integrations[0].permissions.linkCards[0].hosts[0]: must be lowercase`. Then it raises `serial` and writes
   `repo.json.sig`. It finds your key by itself, even after you rename the repository. `npm run check` runs the
   same checks plus the signature check without changing anything.

6. **Commit and push** `repo.json` and `repo.json.sig`.
7. **Add it in Mudrix:** Settings > Integrations > Repositories, paste `https://github.com/<you>/<repo>` and press
   **Add**. Mudrix shows a fingerprint: it must be the one `init` printed. Then trust it, and **Install** your
   integration.

Until you run `npm run init` the GitHub check in this repository shows a warning and passes; afterwards it fails if
Mudrix would refuse the file or the signature does not fit.

## Try things locally

With a released Mudrix app, push to a branch and add `https://github.com/<you>/<repo>/tree/<branch>`: you can try a
change there before it reaches your main branch.

With a Mudrix **development build** (released apps refuse `localhost`), `npm run dev -- repo.json` serves your own
file, signed with a throwaway dev key, at `http://localhost:8789/repo.json`, so you can try it without signing or
pushing. Plain `npm run dev` serves the samples instead: a mock API, link details, the sample widgets and a dev
repository on ports 8787 to 8789. Details in [Writing a widget](docs/writing-a-widget.md#test-it-locally).

## Settings

An integration can offer a few options of its own, shown under it in Settings > Integrations. They are not
permissions: they only choose between things you already ask for. A `toggle` can switch a looked-up fact or
description on and off; a `text` or `choice` value can go into a card's title or subtitle and into inserted text
as `{setting.<id>}`, and a `choice` value also into an api body. For example:

```json
"settings": [
  { "id": "showStars", "label": "Show stars", "type": "toggle", "default": true }
],
```

and in a `linkDetails` lookup: `{ "label": "Stars", "field": "stargazers_count", "format": "count", "if": "showStars" }`.

Values stay on the person's device and Mudrix never sends them anywhere itself, so never ask for passwords or
tokens. Give a new setting the default that matches what the integration did before. Widgets have no settings (your
page can keep its own). All fields are in the
[reference](https://github.com/kapsikkum/mudrix-integrations#settings-options-the-person-can-change).

## Updating your integration

1. Edit `repo.json` (and raise the integration's `version`).
2. `npm run sign`
3. Commit and push both files.
4. Readers press **Refresh** next to the repository in Mudrix. GitHub's cache can take a few minutes.

If your change asks for more than before (another host, button, command, api action, lookup or shown field, changed
wording, a changed widget address or capability list, or a setting with a new type, default or choice value),
readers keep the version they approved until they approve the new one.

An entry that Mudrix can not use (a mistake, or something only a newer Mudrix knows) is listed to readers as
unusable while the rest of your repository keeps working, but `npm run sign` and `npm run check` refuse such a
file, so it should never reach them.

## FAQ

**I lost my key.** You cannot publish updates any more, and there is no way around it: a new key is a new
repository identity. Set `"key"` in `repo.json` back to `"REPLACE_ME_RUN_npm_run_init"`, run `npm run init` (add
`--key-out <new path>` if the default file name is taken), push, and tell your readers to remove the repository and add it again, comparing the new fingerprint. This is why you back the
key up.

**Can the repository be private?** Mudrix fetches it without credentials, so it must be reachable by anyone who adds
it. A public repository (or any public `https://.../repo.json` with `repo.json.sig` next to it) works.

**Can I list it in the official repository?** Yes: open a pull request on
[mudrix-integrations](https://github.com/kapsikkum/mudrix-integrations) adding your integration to its `repo.json`.
The maintainer signs the official repository, so you do not send a signature.

**Does the signature mean my widget is safe?** No. It shows who published the file and that it was not altered. It does
not vouch for code your widget's server sends.
