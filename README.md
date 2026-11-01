# Mudrix integration template

A starting point for your own [Mudrix](https://github.com/kapsikkum/mudrix-web) integration repository. An
integration is a short description in a JSON file (`repo.json`): which links Mudrix should turn into cards, which
buttons and slash commands to add to the message box, which one web service it may call, and, only if you ask for it,
a widget page that runs code. The repository is signed with a key only you hold, and readers pin that key.

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

4. **Edit `repo.json`.** It has one integration, `my-service`, with a link card and a button. Change it to yours:

   | part | what it does |
   | --- | --- |
   | `name`, `description`, `version` | what people read before installing (name up to 40 characters, description up to 400). Raise `version` when you change the integration |
   | `id` | never change it after publishing: it is how Mudrix recognises an installed integration |
   | `permissions.linkCards` | up to 10 rules that turn matching `https` links into a card with a button |
   | `permissions.composerActions` | up to 6 buttons in the message box's menu that insert text |
   | `permissions.commands` | up to 6 slash commands that insert text, run an api action, or open the widget |
   | `permissions.api` | one host and up to 4 fixed POST requests, each shown to the person and confirmed every time |
   | `permissions.widget` | a web page of yours opened inside Mudrix |

   Use the smallest tier that does the job. Every field is documented in the
   [reference](https://github.com/kapsikkum/mudrix-integrations#reference).

   | tier | permissions | what the person sees |
   | --- | --- | --- |
   | Links and buttons | `linkCards`, `composerActions` | Nothing leaves their device |
   | Commands | `commands` | Same: they insert text, or trigger one of the tiers below |
   | API | `api` | The exact request, and a confirmation every time |
   | Widget | `widget` | **A red "this runs code" warning** and an explicit tick to install |

   A widget is the only tier that runs code: the page is served live by your host and can change at any time. It
   can only do what the person approves from a fixed list, and only in the one room it is opened in:
   `send.message`, `read.messages`, `read.room`, `upload.file` and `download.file`. See
   [Writing a widget](docs/writing-a-widget.md) and the sample in [`examples/hello-widget`](examples/hello-widget).
   The whole file may be at most 256 KB, with at most 50 integrations.

5. **Sign it:**

   ```
   npm run sign
   ```

   Run this after every edit. It raises `serial` and writes `repo.json.sig`.

6. **Commit and push** `repo.json` and `repo.json.sig`.
7. **Add it in Mudrix:** Settings > Integrations > Repositories, paste `https://github.com/<you>/<repo>` and press
   **Add**. Mudrix shows a fingerprint: it must be the one `init` printed. Then trust it, and **Install** your
   integration.

Mistakes are reported with the path of the field (for example
`repo.integrations[0].permissions.linkCards[0].hosts[0]: must be lowercase`). Until you run `npm run init` the GitHub
check in this repository shows a warning and passes; afterwards it fails if the signature or schema is wrong.

## Try things locally

`npm run dev` starts a mock API, a sample widget and a signed dev repository on ports 8787 to 8789, for use with a
Mudrix development build. Details in [Writing a widget](docs/writing-a-widget.md#test-it-locally).

## Updating your integration

1. Edit `repo.json` (and raise the integration's `version`).
2. `npm run sign`
3. Commit and push both files.
4. Readers press **Refresh** next to the repository in Mudrix. GitHub's cache can take a few minutes.

If your change asks for more than before (another host, button, command, api action, changed wording, or a changed
widget address or capability list), the integration stays switched off for readers until they approve it again.

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
