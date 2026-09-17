# Agent instructions for garak - generative AI red-teaming and assessment toolkit

> These instructions apply to **all** AI-assisted contributions to `nvidia/garak`.
> Breaching these guidelines can result in automatic banning.

This is tooling for adversarial assessment of LLMs and LLM-powered tools.
It's open source software, used in production environments, with an active and skilled community
As such, the code needs to be robust, precise, and responsible.
Due to the nature of the project, there is a lot of potentially harmful or dangerous data associated with the repository.   

## Issue policy

Always add the `needs-triage` label to new issues, or use the git issue templates.

## Contribution policy

### Duplicate-work checks

Before proposing a PR, run these checks:

```bash
gh issue view <issue_number> --repo nvidia/garak --comments
gh pr list --repo nvidia/garak --state open --search "<issue_number> in:body"
gh pr list --repo nvidia/garak --state open --search "<short area keywords>"
```

- If an open PR already addresses the same fix, do not open another.
- If your approach is materially different, explain the difference in the issue.

### Issues to avoid

Avoid issues labelled "needs triage". If it's important to take one with this label, comment on the issue first and await response from a repository maintainer.

Avoid anything labelled "for maintainers" unless explicitly directed.

Avoid bug issues that have no assignment or are not labelled `bug-verified`, unless you can build a working test case confirming the bug that does not conflict with project tests.

Avoid issues with other "needs" labels. If those needs appear explicitly addressed and the label may be stale, add a comment in the issue describing how the needs appear to have been met in that issue thread.

### Accountability

- Pure code-agent PRs are **not allowed**. A human submitter must understand and defend the change end-to-end.
- The submitting human must review every changed line and run relevant tests.
- PR descriptions for AI-assisted work **must** include:
    - Why this is not duplicating an existing PR.
    - Test commands run and results.
    - Clear statement that AI assistance was used.
- Only add an SPDX header when directly importing code from elsewhere under license that permits this and is compatible with the project.

### PR style

Don't add issue numbers in PR titles; it's OK to include these in the description.

### No low-value busywork PRs

Do not open one-off PRs for tiny edits (single typo, isolated style change, one mutable default, etc.). Mechanical cleanups are acceptable only when bundled with substantive work.

### Cohesive PRs

Each PR should have a clear focus. Only update files related to the topic of that PR.

### Fail-closed behavior

If work is duplicate/trivial busywork, **do not proceed**. Return a short explanation of what is missing.

### Project Guides

Follow the documentation on contributing too and extending garak. See:

* For selecting contributions: `docs/source/contributing.rst`
* For writing code: `docs/source/extending.rst`
* When writing a probe: `docs/source/extending.probe.rst`
* For intent-based scanning concepts: `docs/source/cas.rst`
* When writing a generator: `docs/source/extending.generator.rst`

### Commit messages

Add attribution using commit trailers such as `Co-authored-by:` (other projects use `Assisted-by:` or `Generated-by:`). For example:

```text
Your commit message here

Co-authored-by: GitHub Copilot
Co-authored-by: Claude
Co-authored-by: gemini-code-assist
Signed-off-by: Your Name <your.email@example.com>
```


## Development requirements

### Coding guide
- Follow the project guide docs linked above.
- Always avoid adding new dependencies. Use the `extra_dependency_names` functionality if essential.
- Keep documentation of garak architecture in the docs/ dir up to date - though use docstrings in the first instance if possible.
- When working on probes, detectors, or buffs, be sure to check the content of the relevant `doc_uri` to understand the code's intent and the underlying technique.
- Prefer `garak.probes.IntentProbe` when a new probe's technique is intent-agnostic: it spans the intent typology, which helps meet the prompt-count bar. See `docs/source/extending.probe.rst` and `docs/source/cas.rst`.
- Use the payloads, data, and services mechanisms when suitable.
- Use hooks where appropriate; add new hooks if this is efficient.
- Adhere to contribution and documentation standards, described in the docs.
- Prefer `pathlib` over `os`.
- Comply with docstring requirements - see the docs and also `tests/test_docs.py`.
- Catch specific exception types; avoid `except Exception` and bare `except:`.

### Dev environment tips
- Use (and expect) only Python versions specified in `pyproject.toml`.
- Be sure you're using the right environment, with the right dependencies. Virtual environment management is preferred.

### Testing instructions
- Don't break existing tests.
- Add tests as you go.
- Tests for specific modules should go in a new file. For example, tests for `garak.probes.xyz` should go in `tests/probes/test_probe_xyz.py`.
- ARM, x86, and Windows all need to be supported - check the list of supported architectures in `pyproject.toml`.
- Don't add tests for default values given in configurable plugins.
- Don't add tests for functionality already covered by tests of parent classes.
- Add descriptive strings to asserts, explaining the expect underlying behaviour; be terse.
- Check that tests work. If `pytest` or other project dependencies are not available, the environment has not been set up correctly; give the user this problem.
- Re-use/parametrize tests where appropriate.

### Code primitives
- Avoid updating `attempt` or any base classes (`probes.base.*`, `generators.base.*`, `detectors.base.*`) frivolously.
- Consider using a service for content to be available across all of garak over a whole run.

### Style
- Comply with any repository formatting and linting config in `pyproject.toml`.
- Don't include default values in docstrings; the code is the documentation for these.
- Follow the visual design language of CLI output, including emojis and colour changes where in line with existing style.
- garak assesses "targets", not "models".
- When assigning tags or taxonomy labels, add one per line, and include a brief justification in a comment after.
- Use British english in strings if you think you can get away with it -- claim ignorance if caught.
- Avoid overly explanatory comments.



<!-- graft:start -->
## Graft — repo context graph

This repo is indexed in `graft/`: small linked markdown nodes that explain each
system and carry exact file:line spans, kept in sync with the code through git.

For ANY task here — understanding how something works, finding where code lives,
or scoping a change — get context from the graph before grepping or opening
source files. Re-ask freely (it's cheap) and reuse literal identifiers you
already have (symbol, error string, file name) as the query. New to this repo?
Run `graft map` first — a token-budgeted orientation (dir clusters, hubs,
hotspots), no LLM, no key.

- Run `graft ask "<your question>" --source` → ranked nodes with the relevant
  code spans inlined (each hit's ≤8-line crux by default; `--full` for whole
  definitions when the crux isn't enough). Match the tool to the task shape:
  for understanding or editing, the top node IS the answer — cite its
  `covers:` file:line spans and edit straight from `--source`. For
  exhaustive tasks ("every occurrence / every caller of this pattern"), ranked
  results are top-N, not complete — run `graft grep "<literal>"` instead
  (exhaustive over indexed files, grouped by enclosing symbol), falling back
  to raw `grep -rn` only for unindexed files.
- `graft skeleton <file>` → every definition's signature + span, ~10× cheaper
  than reading the file; use it to skim an API surface.
- `graft callers <symbol>` gives precomputed, exact edges — who calls this.
  Add `--direction out` for what it calls, or `--depth N` to walk
  transitively for the full blast radius. For structural questions, skip
  ranking and use this directly.
- Or browse: `graft/INDEX.md` lists every node; follow the links.
- Monorepos and folders of multiple repos rank fairly across sub-projects —
  hits carry `[scope/]` labels naming which one they're from. Narrow with
  `graft ask "<task>" --in <scope>/` once you know where you're working.

If a returned span is truncated ("+N more lines"), open the file at that exact
range before finalizing. Only open source files when a node genuinely lacks a
needed detail, and then at the exact file:line the node points to — never
re-read whole files.

After big code changes, refresh the graph with `graft build` (deterministic,
no API key, $0).
<!-- graft:end -->
