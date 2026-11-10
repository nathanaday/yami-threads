# Yami Threads

Yami Threads is an Obsidian plugin for work on your code repositories that one agent does and a second agent checks. You describe an idea in your own words. The agent turns it into a spec of checkable requirements, plans the tasks, and does them with a commit for each. Then a fresh, read-only agent that did none of the work validates the result against the spec, requirement by requirement, with evidence.

All of this lives in one note, the thread. The request, the spec, the decisions, each task with its commit, and every validation round stay together in your vault, where you can read them later. A thread closes only after a round passes, so "the agent says it is done" becomes "a second agent checked each requirement and found it met."

Yami Threads runs on the [yami](https://github.com/nathanaday/yami) library and grew out of yami's former showcase plugin. It is not in Obsidian's community plugin directory.

## Requirements

- Obsidian 1.13 or later, on the desktop.
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (`claude`) or the [Codex CLI](https://github.com/openai/codex) (`codex`), installed and signed in. The plugin runs them as its agents, with your own Claude Code or Codex setup.

## Install

Download `main.js`, `manifest.json`, and `styles.css` from the [latest release](https://github.com/nathanaday/yami-threads/releases/latest) into `<your vault>/.obsidian/plugins/yami-threads/`, then turn on **Yami Threads** under **Settings → Community plugins**. [BRAT](https://github.com/TfTHacker/obsidian42-brat) can install it for you and keep it up to date: add `nathanaday/yami-threads`. To build it yourself, see [Development](#development).

The agents start in the vault folder. To let them edit and test a repository outside the vault, add its folder in your Claude Code settings (`additionalDirectories`) or Codex settings (`sandbox_workspace_write.writable_roots`). Otherwise, approve each request in the conversation as it comes.

## Your First Thread

1. Run **New conversation with claude** (or **with codex**) from the command palette. It opens a conversation note in `conversations/` with a message box.
2. Plant the thread with your idea and the repository it touches:

   ```
   /threads:stub Add a --dry-run flag to the deploy script in ~/code/shop
   ```

   The agent keeps your words, finds where the work lands, asks you which folder when the repository is unclear, and writes the thread to `threads/`.
3. Run it, naming the thread by the title the stub reply gave:

   ```
   /threads:run Deploy dry run
   ```

   The agent goes from step to step without stopping:
   - **Spec.** A goal and a few requirements, each with the check that shows it. A read-only reviewer agent reads the spec before any work starts.
   - **Plan.** The tasks in order, each serving one or more requirements.
   - **Work.** Each task done, tested, and committed. The thread records the commit beside the task.
   - **Validation.** A fresh read-only validator checks every requirement and reports a round of results and findings. The agent fixes or settles each finding and validates again.
   - **Close.** After a passing round, the agent closes the thread with a one- or two-sentence summary of what is now true.
4. Open the thread note to read the whole record. **Open the threads panel**, or the spool icon in the ribbon, shows the module, its sessions, and buttons to run, stop, and resume a session.

The agent stops to ask you only when a thread runs out of rounds, a tool refuses a step, an action reaches outside your machine, or a repository is unclear. Ask `/threads:run what should I work on?` to list the open threads, most urgent first.

## The Rules of a Round

The threads tools refuse any step that breaks these rules, so an agent cannot skip them:

- Every requirement gets a result with evidence, and a failing requirement needs a blocking finding that says what is wrong.
- Each finding gets an outcome (fixed, spec, stub, declined, or knowledge) in the next round or at the close.
- A thread may take three rounds, unless its `budget` property sets another number. When the rounds run out, the agent summarizes what passes and fails and asks you how to go on; only your words raise the budget.
- After a passing round, any change to the spec's requirements or rules, or to the plan, needs your words, which the thread keeps as a decision.

The thread's phase (stub, specified, planned, in progress, unvalidated, failed, validated, closed, or dropped) is derived from the note each time it is read, so the note and its status always agree.

## Your Own Flows

The threads module also runs kinds of work that you define in the vault, with no code. **Threads: create flow** writes a starter flow under `internal/threads/flows/` and opens it; **create agent**, **create policy**, **create promise**, and **create skill** write the other forms a flow can use. Each flow gets its own command, **Threads: new <flow>**, and **Threads: create a flow session** lets you pick one. Its sessions go in `threads/` beside the threads, and you run them with **Run this session with Claude** or **with Codex**.

## Commands

| Command | What it does |
| --- | --- |
| **Open the threads panel** | Opens the panel in the right sidebar. |
| **New conversation with claude**, **with codex** | Opens a conversation note with a message box, where `/threads:stub`, `/threads:spec`, and `/threads:run` work. |
| **Threads: create flow**, **create agent**, **create policy**, **create promise**, **create skill** | Writes a starter form under `internal/threads/` and opens it. |
| **Threads: create a flow session**, **Threads: new <flow>** | Creates a session of one of your flows in `threads/`. |
| **Run this session with Claude**, **with Codex**, **Resume this session**, **Stop this session**, **Kill this session's agent** | Act on the session note open in the editor. |

The agents get the threads tools (`threads_stub`, `threads_spec`, `threads_plan`, `threads_round`, `threads_close`, `threads_status`) and the threads skills (`/threads:stub`, `/threads:spec`, `/threads:run`).

## What It Adds to Your Vault

- `threads/`: the threads, and the sessions of your own flows.
- `internal/threads/`: the module's document, its ledger with a **Board** of every thread and its phase, the reviewer and validator agents, the skills, and your forms.
- `conversations/`: a conversation note for each conversation and run.
- `internal/yami/skills/run/SKILL.md`, which every Yami plugin writes. Nothing goes at the vault's root.
- Colors for the module's folders in the file explorer, and a status bar item that says which agents are working.

Yami Threads writes no wiki files and adds no wiki commands: the wiki belongs to [Yami Wiki](https://github.com/nathanaday/yami-wiki).

## With Duet and Yami Wiki

[Duet](https://github.com/nathanaday/obsidian-duet), [Yami Wiki](https://github.com/nathanaday/yami-wiki), and Yami Threads can all run in one vault. Each runs its own copy of yami, with its own agents and its own conversation notes, and a tool of one plugin is not available to another plugin's agents. Plant and run threads in Yami Threads' own conversations: a Duet conversation has no threads tools or skills. Likewise, a Yami Threads agent has no wiki tools, so it reads your notes as files. The known costs of two Yami plugins in one vault are in the [library README](https://github.com/nathanaday/yami/blob/main/yami/README.md#two-yami-plugins-in-one-vault).

To give Claude Code or Codex the wiki tools while they work outside the vault, see [Yami Wiki for Claude Code and Codex](https://github.com/nathanaday/yami-wiki-agent-plugin).

## Development

```sh
npm install                               # takes yami from the release that package.json names
npm run dev                               # rebuilds main.js and styles.css on every change
npm run install:vault -- /path/to/vault   # builds, and copies the plugin into the vault
```

| Command | What it does |
| --- | --- |
| `npm run build` | Type-checks the plugin and its tests, and writes a minified `main.js` and `styles.css`. |
| `npm run lint` | Runs Obsidian's review rules, with no warnings allowed. |
| `npm test` | Runs the unit tests, which check that the version files agree. The threads module's own tests are in the library. |
| `npm run test:obsidian` | Starts a separate Obsidian with a temporary vault, installs the built plugin, and drives it with real agents. Run `npm run build` first. |

Most of the behavior described here lives in the library (`yami/obsidian` and its threads module); this repository holds the plugin's entry point, its build, and its tests. To move to a new library release, change the URL of `yami` in `package.json` and run `npm install`.

Work happens on `main`, where CI runs the type check, lint, unit tests, and build. A version bump on `main` is a release: `npm run set-version X.Y.Z`, commit, and push. See [docs/releasing.md](https://github.com/nathanaday/yami/blob/main/docs/releasing.md) in the yami repository.

## License

[MIT](LICENSE)
