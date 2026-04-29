# agent-fleet

The Pondside household's specialist Claude agents, distributed as a Claude Code plugin marketplace.

## What this is

A single repo that plays two roles:

1. **Marketplace** — `.claude-plugin/marketplace.json` at the root catalogs the agents available to install across the Pondside fleet.
2. **Plugins** — each agent lives in its own subdirectory at the root, structured as a self-contained Claude Code plugin (`<name>/.claude-plugin/plugin.json` + `<name>/agents/<Name>.md`).

Co-locating the marketplace and the plugins means one git remote, one editing surface, one place to find everything.

## Layout

```
agent-fleet/
├── .claude-plugin/
│   └── marketplace.json     ← the catalog
├── edgar/                   ← Postgres DBA, lives on memorybanks
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── agents/
│       └── Edgar.md
└── (future agents as sibling subdirectories)
```

## Adding an agent

1. Create a subdirectory: `mkdir <name> && cd <name>`
2. Add the plugin manifest at `.claude-plugin/plugin.json` (see existing examples)
3. Add the agent file at `agents/<Name>.md` with frontmatter (`name`, `description`, `model`, `memory`)
4. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./<name>"`
5. Commit, push. Subscribed Claude Code instances pick up the new plugin via marketplace update.

Plugin manifests deliberately omit a `version` field — every commit is the live version. For an internal fleet under active development, this is the canonical pattern.

## Subscribing to this marketplace

On a Claude Code instance:

```
/plugin marketplace add Pondsiders/agent-fleet
/plugin install <agent>@pondsiders-agents
```

Then in `/plugin → Marketplaces`, flip on auto-update for `pondsiders-agents` (third-party marketplaces default OFF). Subsequent commits to this repo propagate at the next Claude Code startup.

## Memory

Agents distributed via plugin write their persistent memory to a per-machine path under the user's home directory (`~/.claude/projects/<project>/memory/`). On Pondside VMs, `/home` is mounted as a separate filesystem backed by the host's ZFS, so agent memory survives VM rebuilds and is captured by host-side ZFS snapshots. No additional memory configuration required at the agent level.

## Current agents

- **Edgar** — Postgres database administrator on memorybanks. Owns the Postgres software, filesystem health for `/var/lib/postgresql`, WAL archiving, replica health, basebackups, and database lifecycle. Does not read application data.
- **Lazlo** — object-storage administrator on warehouse13. Owns the Garage software, the object-store filesystem, cluster topology, bucket and key lifecycle, and the `rclone sync` to Backblaze B2. Does not read object contents.
- **Mac** — the resident technician on Jeffery's MacBook Pro. Owns macOS, Homebrew, application lifecycle, disk space, container runtimes, local model serving, and Apple Silicon tooling. Operates with cold-tool register: direct, factual, no honorifics. Different shape from the facility specialists — lives on a personal device, serves one human.
- **Operator** (he goes by **Link**) — resident operator of Primer, the household's hypervisor and Docker host. Owns the host substrate (hardware, OS, ZFS, libvirt, Docker, networking, GPU passthrough) and per-cluster Postgres / per-bucket object-store administration on Primer's Docker daemon. Does not read database contents or object data. Operates the substrate; defers to Jeffery and the household's AIs on architecture.

## Related

- `Pondsiders/Pondside-Ops` — VM cloud-init, libvirt domain definitions, systemd units, and other operational substrate
- `alphafornow/Alpha` — Alpha's identity plugin (loaded via `JE_NE_SAIS_QUOI` env var, separate from this fleet)
- Abe lives at `/opt/abe/.claude/agents/Abe.md` on Primer today; planned migration into this marketplace once Edgar validates the pattern
