# .github

Organization-level files for [Cats Inc](https://github.com/cats-inc).

- [`profile/README.md`](profile/README.md) — rendered on the organization page at
  <https://github.com/cats-inc>
- [`profile/README.zh-TW.md`](profile/README.zh-TW.md) — Traditional Chinese version
- [`.github/workflows/add-issue-to-project.yml`](.github/workflows/add-issue-to-project.yml)
  — reusable workflow that adds each new issue to the
  [Cats Development](https://github.com/orgs/cats-inc/projects/1) Project with
  Status Backlog. `cats-runtime`, `cats-platform`, `cats-one` and `cats-apps`
  call it on `issues: opened` and pass the `CATS_PROJECT_TOKEN` organization
  secret, a fine-grained token with organization Projects read and write.

There is no application source code in this repository.
