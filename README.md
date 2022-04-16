# eslint-lax

> Note: Current code scaffolded from [typescript-starter](https://github.com/bitjson/typescript-starter) for speed, nothing implemented yet

A proposed augmentation of [ESLint](https://github.com/eslint/eslint) to make pushing changes to lint rulesets less
painful for tooling engineers and source maintainers.

## Motivation

### Managing Legacy and Modern Code with a single tool

In larger projects, it is inevitable that a large amount of source code needs
to be maintained. Traditionally, linting and formatting can be a solid immediate
source of improved stability of the source code, as it provides both opinions of
how to unify the code style of the project and enforcement of generally approved
best practices.

### `lint-staged` doesn't full solve the issue

The tooling around JS linting today can often be a pain in day-to-day development
when dealing with both legacy and modern source code.
While it can be generally agreed across a project that new and commonly
touched source should have the latest and greatest enforcement, linting legacy files
can often be a significant task and fall under more of a "tech debt" effort.
One of the most common patterns to alleviate this is [`lint-staged`](https://github.com/okonet/lint-staged). Conceptually,
linting only the files that have been updated within a given change request can reduce
the amount of unrelated lint errors that a team member trying to push the change needs
to resolve. However, if a developer makes a single change to an old file, they will be
tasked with fixing everything about this file relative to ESLint, which could also
actually increase the risk of the change overall. We want to come up with a better solution
to avoid this situation.

## Solution

This project proposes that we agument the concept of `lint-staged` by labeling the 
source code with a `maintenance-status` to determine how strict lint rules should be
relative to the given file. We propose three statuses, each with a different ESLint
configuration, that will provide various lifecycles

- `legacy`: These files are rarely or minimally touched and thus do not need to be
strictly enforced with the target state rules. We should run `.eslintrc.js` on this
set of source when present in a commit.
- `low`: These files are **either** touched less often or have a relatively smaller
diff size compared to other files, so we should gradually apply changes to these files
to minimize developer toil. Run `.eslintrc.incremental.js` on these files
- `high`: These files are changed often (or are new files) and should have the highest
expectations of code quality to combat the frequent changes and promote stability. We
would run `.eslintrc.target.js` on these files.

These three ESLint configuration can be configurable and should inherit from each other
(TBD: We can actually just have them automatically inherit optionally). In more detail:

- `.eslintrc.js` can be thought of as a baseline lint configuration. This is a stable set
of rules that the team wants to migrate away from
- `.eslintrc.incremental.js` provides a way for teams to prepare a file for when it requires
the strongest rules. It would be up to the project to find the right balance between base and
target that lessens the blow of needing to update the file to the new standard
- `.eslintrc.target.js` is the "north star" configuration that the team wants to push for.
This will be explicit, strict, and push developers to produce durable code with best practices.

### Labeling Source Code as Legacy

Several data points can help us to indicate if we think the source code is "Legacy", including:

- How many times it has been involved with commits
  - Is this frequency decreasing with time?
- Amount of lines changed in the file
  - Should a one-liner change the maintenance status of a file?
- One-time: How many (non-formatting) eslint standard rules are failing on this file?
  - Files with many basic rules failing were likely generated when a different best practice existed

Suggested "caluclation" TBD

#### Manual labeling

Sometimes, even though a file hasn't been used very often, it can still be a file we want
to enforce the the latest and greatest on the be prepared for when it gets more substantial
usage. For this use case, we can annotate the file:

```js
// eslint-lax maintenance:high
```

## Pitfalls and Alternatives

TBD

### Maintenance of this tool

TBD
