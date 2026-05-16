# NotebookLM Source-Pack Handoff

Strict web applications are not good first targets for full browser
automation.

When a service uses strong browser verification, the useful OpenClaw pattern is
not to fight the login screen. Use the human's already-authorized session for
human-only actions, and move the repeatable preparation work into a CLI or
file-based workflow.

NotebookLM is a good example. The hard part is often not typing a prompt. The
hard part is collecting sources, adding them to the right notebook, tracking
what was added, and deciding which artifact is worth asking the operator to
generate next.

## Pattern

Use a source-pack handoff:

```text
research / repository / video / run log
  -> sanitized source pack
  -> CLI upload to NotebookLM
  -> notebook id and source manifest
  -> recommended next artifact
  -> operator clicks the generation UI
  -> result is recorded back into OpenClaw notes
```

The OpenClaw agent does the durable preparation. The operator handles
high-friction UI controls and final publication decisions.

## Why This Works

Many browser tools can open simple pages, click links, fill forms, and capture
screenshots. They become less reliable on sites that combine:

- account-specific login state
- anti-bot or human-verification checks
- file pickers
- Google Drive or account permission modals
- artifact generation buttons
- long-running UI state transitions

A CLI upload path changes the automation target. Instead of controlling the
whole NotebookLM UI, the agent produces and uploads source material through a
bounded command.

This keeps the highest-value work automated without treating a browser
challenge as something to bypass.

## Source Pack Shape

A source pack should be boring and auditable:

```text
source-pack/
  manifest.json
  overview.md
  repository-notes.md
  video-notes.md
  run-log-summary.md
  links.md
```

The manifest should record:

- source title
- source type
- public URL, if safe to publish
- local filename
- generated-at timestamp
- redaction status
- why this source is useful

The source files should be written for later retrieval, not just for one prompt.
They should include primary evidence such as public README excerpts, public
release notes, sanitized run summaries, or operator-written experiment notes.

## CLI Upload Boundary

A CLI such as `notebooklm-tui` can move source upload out of the GUI when valid
NotebookLM cookies are available.

Useful operations:

```text
list notebooks
create a notebook
upload Markdown or text sources
upload supported files
add URL sources
restore from a local backup folder
```

The cookie file is a secret. It should stay outside the public repository and
outside copied source packs.

The CLI should be treated as an adapter to an unofficial internal API, not as a
stable platform contract.

## Health Checks

Because the NotebookLM endpoint is not a public stable API, the adapter needs a
small health check before important media work.

Check:

```text
auth check
notebook list
small Markdown upload to a test notebook
source list after upload, if available
```

Classify failures before asking for action:

| Failure | Likely meaning | Next step |
| --- | --- | --- |
| missing cookie file | setup not completed | ask operator to run login flow |
| auth expired | session expired | ask operator to refresh auth |
| CSRF token missing | page or auth contract changed | inspect login/client adapter |
| RPC failure | internal API changed | update adapter notes before retrying |
| upload failure only | format, size, or notebook id issue | inspect source pack and target |
| network failure | access layer or local connectivity | check tunnel/network separately |

This should run before a Media Factory job or on a low-frequency schedule. It
does not need to become a noisy daily heartbeat.

## Operator Handoff

After upload, the agent should not pretend the job is complete.

It should report:

- notebook title
- notebook id or safe link
- uploaded source count
- failed source count
- source manifest path
- suggested artifact to create next
- short reason for the suggestion

Examples of artifact recommendations:

| Goal | Suggested artifact |
| --- | --- |
| explain a repo to readers | briefing document or FAQ |
| support a blog article | report and outline |
| teach a process | slide deck |
| compare implementation options | mind map or table |
| create audio/video companion material | audio or video overview |

The operator can then click the appropriate NotebookLM generation button in
their existing browser session.

## Redaction Rules

Do not upload raw private operations by default.

Before building a source pack, remove:

- tokens, cookies, credentials, and private keys
- private host names, private IP addresses, SSH aliases, and real domains
- raw user data and private account identifiers
- raw production logs that include secrets or attacker-controlled text
- unpublished business data that was not intended for NotebookLM

For public Media Factory work, prefer public repository URLs, public videos,
public articles, sanitized experiment logs, and operator-authored notes.

## Relationship To Browser Tools

Browser tools remain useful.

Use browser tools for:

- screenshots
- read-only extraction
- evidence gathering on simple pages
- confirming that a published page is visible
- assisting an already-open human session through paste-only input

Do not rely on browser tools for:

- solving human-verification challenges
- managing Google account permission flows
- clicking artifact generation buttons without review
- changing publication or account settings

The source-pack handoff gives the agent a strong role without needing full UI
control.

## Practical Rule

Let OpenClaw agents prepare the source material, upload it through a bounded
CLI when authentication is healthy, and recommend what should be generated.

Let the operator handle the UI-only generation step and final judgment. This
keeps NotebookLM useful in an autonomous workflow without making the agent
responsible for brittle browser automation.
