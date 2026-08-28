# skills.cirrus.ac.uk

A curated collection of **AI agent skills** for the [Cirrus UK National Compute Resource](https://www.cirrus.ac.uk) HPC systems,
served at [skills.cirrus.ac.uk](https://skills.cirrus.ac.uk).

Agent skills are plain Markdown instruction files that teach AI coding
assistants (such as [Claude Code](https://claude.ai/code)) how to perform
specific tasks correctly in your environment. Loading a skill gives the
agent up-to-date, site-specific knowledge without you having to explain
the environment from scratch every session.

!!! Note "Based on skills.cirrus.ac.uk"
    This repository draws heavily on the original [skills.cirrus.ac.uk repository](https://github.com/cirrus-sc/skills.cirrus.ac.uk)

---

## Available Skills

| Skill | Description | URL |
|---|---|---|
| [Slurm](site/plugins/cirrus/skills/slurm/SKILL.md) | Submit, monitor and manage HPC jobs on Cirrus using the Slurm workload manager | `https://skills.cirrus.ac.uk/skills/slurm/SKILL.md` |
| [Python](site/plugins/cirrus/skills/python/SKILL.md) | Install and manage Python environments on Cirrus using Conda (Miniforge), uv, or Cray Python | `https://skills.cirrus.ac.uk/skills/python/SKILL.md` |
| [Modules](site/plugins/cirrus/skills/modules/SKILL.md) | Use the modules system, Cray Programming Environments, compiler wrappers (cc, CC, ftn), GNU and NVIDIA compilers, and profiling tools on Cirrus | `https://skills.cirrus.ac.uk/skills/modules/SKILL.md` |
| [Spack](site/plugins/cirrus/skills/spack/SKILL.md) | Install, configure, and use Spack to build HPC software on Cirrus-AI and Cirrus 3, including the buildit config repository and targeting neoverse_v2 / aarch64 | `https://skills.cirrus.ac.uk/skills/spack/SKILL.md` |
| [Containers](site/plugins/cirrus/skills/containers/SKILL.md) | Run containers on Cirrus using Podman-HPC and Apptainer. Covers image management, GPU access, and multi-node MPI/NCCL workloads over Slingshot 11 | `https://skills.cirrus.ac.uk/skills/containers/SKILL.md` |
| [MPI](site/plugins/cirrus/skills/mpi/SKILL.md) | Use MPI on Cirrus with Cray MPICH or OpenMPI. Covers PMI types, srun --mpi flags, Slingshot 11 performance, and why mpirun/mpiexec must not be used | `https://skills.cirrus.ac.uk/skills/mpi/SKILL.md` |
| [NCCL](site/plugins/cirrus/skills/nccl/SKILL.md) | Use NCCL for multi-node GPU communication on Cirrus-AI over Slingshot 11. Covers the brics/nccl module, aws-ofi-nccl plugin, building from source, and NCCL in containers | `https://skills.cirrus.ac.uk/skills/nccl/SKILL.md` |
| [GPUs and CUDA](site/plugins/cirrus/skills/cuda/SKILL.md) | Use GPUs and CUDA on Cirrus-AI (NVIDIA GH200, sm_90). Covers cudatoolkit/nvhpc modules, compiling with nvcc, and CUDA forward compatibility via NGC containers or NVIDIA HPC SDK | `https://skills.cirrus.ac.uk/skills/cuda/SKILL.md` |

---

## Using a Skill

### Claude Code

Add the Cirrus marketplace:

```
/plugin marketplace add https://github.com/cirrus-sc/skills.cirrus.ac.uk
```

Install the `cirrus` plugin from the marketplace:

```
/plugin install cirrus@cirrus-skills
```

Or add to your project's `.claude/settings.json` to enable automatically:

```json
{
  "extraKnownMarketplaces": {
    "cirrus-skills": {
      "source": {
        "source": "url",
        "url": "https://skills.cirrus.ac.uk/.claude-plugin/marketplace.json"
      }
    }
  },
  "enabledPlugins": { "cirrus@cirrus-skills": true }
}
```

Reload skills to find and activate skills provided by Cirrus plugin:

```
/reload-skills
```

View skills provided by Cirrus plugin:

```
/skills
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  Skills
  9 skills · Space to cycle, Enter to save, / to search, t to sort, Esc to cancel

  ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
  │ ⌕ Search skills…                                                                                                         │
  ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
  ❯ 🔒 on         cirrus:docs · plugin · ~100 tok · locked by plugin
    🔒 on         cirrus:containers · plugin · ~200 tok · locked by plugin
    🔒 on         cirrus:cuda · plugin · ~190 tok · locked by plugin
    🔒 on         cirrus:modules · plugin · ~180 tok · locked by plugin
    🔒 on         cirrus:mpi · plugin · ~170 tok · locked by plugin
    🔒 on         cirrus:nccl · plugin · ~210 tok · locked by plugin
    🔒 on         cirrus:python · plugin · ~150 tok · locked by plugin
    🔒 on         cirrus:slurm · plugin · ~190 tok · locked by plugin
    🔒 on         cirrus:spack · plugin · ~200 tok · locked by plugin

  Plugin skills are managed via /plugin

```

### Codex

Tested on Codex CLI `0.147.0`.

Add the Cirrus plugin marketplace:

```cirrus-session
$ codex plugin marketplace add cirrus-sc/skills.cirrus.ac.uk
Added marketplace `cirrus-skills` from https://github.com/cirrus-sc/skills.cirrus.ac.uk.git.
Installed marketplace root: .../marketplaces/cirrus-skills
```

Install the `cirrus` plugin from the marketplace:

```cirrus-session
$ codex plugin add cirrus@cirrus-skills
Added plugin `cirrus` from marketplace `cirrus-skills`.
Installed plugin root: .../cirrus-skills/cirrus/1.0.0
```

### Cursor

Tested on Cursor CLI `2026.08.04-aaa8809`.

Add the Cirrus plugin marketplace:

```cirrus-session
$ agent plugin marketplace add https://github.com/cirrus-sc/skills.cirrus.ac.uk
Fetching plugins from https://github.com/cirrus-sc/skills.cirrus.ac.uk...
✓ Added marketplace cirrus-skills (1 plugin)
  cirrus - AI agent skills for Cirrus HPC systems
Tip: use /plugins in interactive mode to install plugins from this marketplace.
```

Run the Cursor `agent` CLI tool:

```cirrus-session
$ agent
```

Install the `cirrus` plugin from the marketplace:

```cirrus-session
> /plugin marketplace list
 Marketplaces

 Global
    Cursor Plugin Marketplace • 223 plugins • 0 installed • Never indexed

 User
  → cirrus-skills • 1 plugin • 0 installed • Last indexed 2026-08-11 14:14 UTC


 Enter for details • Esc to close

> [Press Enter]
```

```cirrus-session
 Marketplace details / cirrus-skills

 Scope: User
 Plugins: 1
 Installed: 1 (cirrus)
 Indexing: Last indexed 2026-08-11 14:14 UTC
 Source: https://github.com/cirrus-sc/skills.cirrus.ac.uk

  → Browse plugins
    Remove marketplace

 Enter to select • Esc to go back

> [Press Enter]
```

```cirrus-session
 Plugins Installed  Marketplace  (←/→ or tab to cycle)

 Install Plugins

 ┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
 │ ⌕ cirrus-skills                                                                                                                        │
 └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

 → cirrus (cirrus-skills) [installed]
   AI agent skills for Cirrus HPC systems


 Enter for details • Esc to clear

> [Press Enter]
```

```cirrus-session
 Plugins Installed  Marketplace  (←/→ or tab to cycle)

 Install Plugins

 ┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
 │ ⌕ cirrus-skills                                                                                                                        │
 └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

 → cirrus (cirrus-skills)
   AI agent skills for Cirrus HPC systems


 Enter for details • Esc to clear

> [Press Enter]
```

```cirrus-session
 Plugin details / cirrus

 AI agent skills for Cirrus HPC systems

 Marketplace: cirrus-skills

 Skills: 9 (containers, cuda, brics-hpc-ai-code, modules, mpi, nccl, python, slurm, spack)

  → Install for you (user scope)
    Install for all collaborators on this repository (project scope)

 Enter to select • Esc to go back

> [Press Enter]
```

### Antigravity

Tested on Antigravity CLI `1.1.12`.

Install the `cirrus` plugin from the Cirrus Skills repository: 

```cirrus-session
$ agy plugin install https://github.com/cirrus-sc/skills.cirrus.ac.uk/site
$ agy plugin enable cirrus
```

### Other agent-based tools

Paste the skill URL into the tool's context or skill configuration. The
raw Markdown content is served directly from this site.

---

## Repository Structure

```
.claude-plugin
  marketplace.json           # Claude Code plugin marketplace catalog
site/plugins/cirrus/       # All web-served content (GitHub Pages source)
  .claude-plugin/
    marketplace.json         # Claude Code plugin marketplace catalog
  skills/
    slurm/
      SKILL.md               # Slurm skill file (AgentSkills spec format)
    docs/
      SKILL.md               # User documentation skill file (AgentSkills spec format)
  index.html                 # Public site landing page
  marketplace.json           # Simple skills index for other agent tools
  CNAME                      # Custom domain configuration
.github/
  agents/
    skills-agent.md          # Instructions for AI agents on creating skills
.vscode/
  settings.json              # Word-wrap settings for Markdown/.chatagent files
README.md                    # This file
```

---

## Contributing / Adding a New Skill

See [`.github/agents/skills-agent.md`](.github/agents/skills-agent.md)
for the full conventions on how skills are structured, named, and
registered in the marketplace.

For full Cirrus documentation visit
[docs.cirrus.ac.uk](https://docs.cirrus.ac.uk).
