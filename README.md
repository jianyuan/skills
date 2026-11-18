# Skills

Agent skills I use and share. They follow the [Agent Skills](https://agentskills.io) format, so they work with Claude Code, Codex, Cursor and other agents that read `SKILL.md` files.

## Install

Install with [skills.sh](https://skills.sh):

```sh
npx skills add jianyuan/skills
```

To install one skill:

```sh
npx skills add jianyuan/skills --skill improve-makefile
```

## Skills

| Skill | What it does |
| --- | --- |
| [improve-makefile](skills/improve-makefile/SKILL.md) | Makes a Makefile self-documenting: a single-awk `help` target, `## description` comments on each target and `##@` section headings. Background: [A better self-documenting Makefile](https://www.jian.fyi/blog/a-better-self-documenting-makefile). |
