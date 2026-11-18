---
name: improve-makefile
description: Make a Makefile self-documenting with a single-awk `help` target, `## description` comments on each target, `##@` section headings and `make` printing the help by default. Use whenever the user asks to improve, tidy, clean up, document or add help to a Makefile, wants `make help` or `make` to list targets, mentions a self-documenting Makefile, or is adding targets to a Makefile that has no help yet. Also use when they ask whether to move from Make to just or Task.
metadata:
  author: Jian Yuan Lee
  url: https://www.jian.fyi/blog/a-better-self-documenting-makefile
---

# Improve a Makefile

Make a Makefile explain itself: running `make` (or `make help`) lists each target people run, with a one-line description, grouped under section headings, in the order they appear in the file.

## The help target

Use this recipe. It's one `awk` process, it works on stock macOS (GNU Make 3.81, BSD awk) as well as Linux, and it handles `include`d files:

```make
.DEFAULT_GOAL := help

##@ General
help: ## Show this help
	@awk 'BEGIN { FS = ":.*## " } \
	  /^##@ / { printf "\n\033[1m%s\033[0m\n", substr($$0, 5); next } \
	  /^[a-zA-Z0-9_.\/-]+:.*## / { printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2 }' $(MAKEFILE_LIST)
```

It prints:

```
General
  help            Show this help

Development
  build           Build the binary
  test            Run the tests
```

How it's built, so you can adapt it without breaking it:

- The target pattern is anchored to the start of the line. Only `name: ... ## text` lines match, so a recipe line with ` ## ` in it, such as a trailing shell comment, stays out of the help.
- There's no `sort`, so targets come out in file order. That lets the author group related targets and put the important ones first.
- `##@ Heading` lines become bold section headings.
- `$(MAKEFILE_LIST)` covers every `include`d makefile, and each file's sections appear in the order Make reads them.
- `$$` escapes `$` for Make. Recipe lines start with a tab. The continuation lines after `\` start with a tab, then spaces for alignment.
- `%-15s` fits names up to 15 characters. If the longest documented target is longer, raise it to the longest name plus 2.
- The class `[a-zA-Z0-9_.\/-]` covers names like `docs/serve`. Widen it if the file uses other characters in target names, but leave pattern rules (`%.o: %.c`) undocumented.

## Steps

1. **Read the whole Makefile and any files it `include`s.** Work out which targets people actually run (build, test, lint, run, deploy and so on) and which are internal helpers, file targets or pattern rules. Only the targets people run get a `##` description.

2. **Replace any existing help target.** Many Makefiles use the version from Victoria Drake's [How to create a self-documenting Makefile](https://dev.to/victoria/how-to-create-a-self-documenting-makefile-2b0e):

   ```make
   help: ## Show this help
   	@egrep -h '\s##\s' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'
   ```

   Swap it for the recipe above and tell the user in a sentence or two what changes: only target lines are matched now, so recipe lines with ` ## ` no longer appear, and targets keep their file order and can be grouped under headings. The existing descriptions keep working because they use the same `## ` format.

3. **Describe each target people run** with a short `## Verb phrase` on the target line: `test: build ## Run the tests`. Use the imperative mood and sentence case, with no full stop. Keep each one under about 50 characters so the help fits on one line. Base the text on what the recipe does, not on the target's name.

4. **Group targets under `##@` headings** when there are more than about five, for example `##@ General` (help), `##@ Development`, `##@ Testing` and `##@ Release`. Move target blocks into those sections only if that doesn't change behaviour. Make doesn't care about rule order except for the default goal, which step 5 handles.

5. **Make `help` the default goal, carefully.** `.DEFAULT_GOAL := help` changes what a bare `make` does. That's right for a task-runner Makefile. If the file is a real build (a C project where `make` builds `all`, or a repo whose docs or CI run plain `make`), ask before changing the default. Check the README, CI config and scripts for bare `make` calls first. If you keep the old default, `make help` still works.

6. **Declare task targets as `.PHONY`** if they aren't already, so a file called `test` or `build` can't stop them from running. Leave real file targets alone.

7. **Verify.** Run `make help`, and `make` on its own if you changed the default goal. Check that:
   - every documented target appears once, under the right heading
   - no recipe or comment lines appear
   - the columns line up

   Run one or two of the real targets (or `make -n <target>`) to confirm nothing else changed.

## Keep the change focused

Only change what makes the file document itself. Don't rename targets, rewrite recipes, add tools or switch indentation, because other people's scripts, CI and habits depend on them. If you notice a real bug in a recipe, mention it instead of fixing it silently.

Optional extras, only if the user wants them:

- Work out the column width from the longest target name.
- Skip the colours when `NO_COLOR` is set or the output isn't a terminal (`[ -t 1 ]`).

## When Make is the wrong tool

If the Makefile is only a task runner (no file targets, every target `.PHONY`), you can mention in one sentence that some tools have help built in:

- [just](https://github.com/casey/just) turns the comment above a recipe into its description in `just --list`, and `[group('dev')]` gives sections.
- [Task](https://taskfile.dev) lists each task's `desc:` with `task --list`.

Don't migrate unless asked. Keep Make wherever `make` is what people expect, for example when the README, CI or contributors' habits already rely on `make <target>`.
