# Trent — security advisor plugin

Trent, an AI security advisor. Review code, plans and configs for security problems, run threat models over a repo or website, and track remediation — without leaving the editor.

## Installing

Add the marketplace by repository name — `trnt-ai/trent-agent-plugin` is the only public
marketplace that carries Trent, and adding it by name is what ties the install
to us. (A plugin manifest carries no signature, so the name is what you are
trusting; never add a marketplace from a URL to a file.)

Claude Code:

```
/plugin marketplace add trnt-ai/trent-agent-plugin
/plugin install trent@trent
```

Codex CLI:

```
codex plugin marketplace add trnt-ai/trent-agent-plugin
codex plugin add trent@trent
```

Check what you installed with `claude plugin details trent`. It prints
`Source: trent@trent` and the manifest's identity fields.

## Signing in

The plugin declares a remote MCP server and no credentials. Authentication
happens in the browser the first time a tool runs — there is no API key to paste
and none is shipped here.

## Verifying where a release came from

Each release is one commit, and its message carries the revision it was built
from:

```
Published from source revision <sha>
```

Run `git log -1` in a clone of this repository to read it. The `<sha>` is an
identifier, not a link — there is nowhere to follow it to. What it gives you is
an exact name for the build you are running: quote it to support and they can
say which release you have. If a copy of this plugin does not carry a message in
that form, it was not published by us.

The plugin manifest names `https://github.com/trnt-ai/trent-agent-plugin` — this repository — as
its `repository`, and `trent.ai` as its `homepage`. A plugin claiming to be Trent
from anywhere else is not ours.

## Pinning a release

Every release is tagged `plugin-v<version>`, on the commit that release
published. `git tag -l` in a clone lists them. A tag here never moves: the
repository's rules refuse it, so a pin keeps resolving to the same files.

On a Team or Enterprise plan an admin can register this marketplace for everyone
in `managed-settings.json`, pinned to a release. `ref` is the tag and `sha` is
the commit it names; with both set the `sha` is the pin and the `ref` says which
release it came from:

```json
{
  "extraKnownMarketplaces": {
    "trent": {
      "source": {
        "source": "github",
        "repo": "trnt-ai/trent-agent-plugin",
        "ref": "plugin-v0.1.0",
        "sha": "<the 40-character commit plugin-v0.1.0 names>"
      }
    }
  },
  "enabledPlugins": { "trent@trent": true }
}
```

Read the `sha` off the tag rather than copying one from anywhere else:

```
git ls-remote https://github.com/trnt-ai/trent-agent-plugin refs/tags/plugin-v0.1.0 'refs/tags/plugin-v0.1.0^{}'
```

One line back means the tag names that commit — pin it. Two lines back means
the tag is annotated: the line ending `^{}` names the commit, and that is the
one to pin.

Upgrading is then a deliberate edit of those two fields, not something that
happens when someone reinstalls.

## Getting help

Questions, bugs and feature requests reach us at <https://trent.ai> or through
your usual support channel — that is where they get answered.

This repository is published from Trent's build on each release; every file in
it is generated, so it is not the place to file or fix anything.

## Licence

These files are Trent AI's property. See [LICENSE](LICENSE) for the terms, and
<https://trent.ai> for anything the terms do not answer.
