# skills.cirrus.ac.uk

A curated collection of **AI agent skills** for the [Cirrus UK National Compute Resource](https://www.cirrus.ac.uk) HPC systems,
served at [skills.cirrus.ac.uk](https://skills.cirrus.ac.uk).

Agent skills are plain Markdown instruction files that teach AI coding
assistants (such as [Claude Code](https://claude.ai/code)) how to perform
specific tasks correctly in your environment. Loading a skill gives the
agent up-to-date, site-specific knowledge without you having to explain
the environment from scratch every session.

!!! Note "Based on skills.isambard.ac.uk"
    This repository draws heavily on the original [skills.isambard.ac.uk repository](https://github.com/isambard-sc/skills.isambard.ac.uk)

---

## Available Skills

| Skill | Description | URL |
|---|---|---|
| [Slurm](site/plugins/cirrus/skills/slurm/SKILL.md) | Submit, monitor and manage HPC jobs on Cirrus using the Slurm workload manager | `https://skills.cirrus.ac.uk/skills/slurm/SKILL.md` |
| [Python](site/plugins/cirrus/skills/python/SKILL.md) | Install and manage Python environments on Cirrus using Conda (Miniforge), uv, or Cray Python | `https://skills.cirrus.ac.uk/skills/python/SKILL.md` |
| [Modules](site/plugins/cirrus/skills/modules/SKILL.md) | Use the modules system, Cray Programming Environments, compiler wrappers (cc, CC, ftn), GNU and NVIDIA compilers, and profiling tools on Cirrus | `https://skills.cirrus.ac.uk/skills/modules/SKILL.md` |
| [MPI](site/plugins/cirrus/skills/mpi/SKILL.md) | Use MPI on Cirrus with Cray MPICH or OpenMPI. Covers PMI types, srun --mpi flags, Slingshot 11 performance, and why mpirun/mpiexec must not be used | `https://skills.cirrus.ac.uk/skills/mpi/SKILL.md` |

---

## Using a Skill

### Claude Code

Add the Cirrus marketplace:

```
/plugin marketplace add https://skills.cirrus.ac.uk
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
  5 skills · Space to cycle, Enter to save, / to search, t to sort, Esc to cancel

  ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
  │ ⌕ Search skills…                                                                                                         │
  ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
  ❯ 🔒 on         cirrus:docs · plugin · ~100 tok · locked by plugin
    🔒 on         cirrus:modules · plugin · ~180 tok · locked by plugin
    🔒 on         cirrus:mpi · plugin · ~170 tok · locked by plugin
    🔒 on         cirrus:python · plugin · ~150 tok · locked by plugin
    🔒 on         cirrus:slurm · plugin · ~190 tok · locked by plugin

  Plugin skills are managed via /plugin

```

---

## Repository Structure

```
.claude-plugin
  marketplace.json           # Claude Code plugin marketplace catalog
site/plugins/cirrus/         # Skills files
  skills/
    slurm/
      SKILL.md               # Slurm skill file (AgentSkills spec format)
    docs/
      SKILL.md               # User documentation skill file (AgentSkills spec format)
    ...etc..
  marketplace.json           # Simple skills index for other agent tools
.github/
  agents/
    skills-agent.md          # Instructions for AI agents on creating skills
CNAME                        # Custom domain configuration
README.md                    # This file
```

---

## Contributing / Adding a New Skill

See [`.github/agents/skills-agent.md`](.github/agents/skills-agent.md)
for the full conventions on how skills are structured, named, and
registered in the marketplace.

For full Cirrus documentation visit
[docs.cirrus.ac.uk](https://docs.cirrus.ac.uk).
