# Automated version management ([semantic-release](https://github.com/semantic-release/semantic-release))

## Step 1. Create a GitHub Token
Go to your GitHub profile **Settings** > **Developer settings** > **Personal access tokens**
- Create **GITHUB_TOKEN** token
- Select **repository**
- Allow **Read** and **Write** access to actions

## Step 2. GitHub Repository settings

### Set Up GitHub Workflow Permissions

Go to the repository **Settings** > **Actions** > **General**

**Workflow permissions**

- Select **Read** and **write** permissions then save

### Set Up GitHub Repository Environments

Go to the repository **Settings** > **Environments**

- Environment name: **GH_TOKEN**

Scroll down to **Environment secrets** then `Add environment secret`

| Name | Value |
| --- | --- |
| `GH_TOKEN` | GITHUB_TOKEN's value |

## Step 3. Integrate [semantic-release](https://github.com/semantic-release/semantic-release) Into Repository
### Install semantic-release and plugins
```
npm install --save-dev semantic-release @semantic-release/changelog @semantic-release/commit-analyzer @semantic-release/git @semantic-release/github @semantic-release/release-notes-generator
```

### Configurate [semantic-release](https://github.com/semantic-release/semantic-release)

In your `package.json`, set the `private` field to `true`. This indicates that your package is not intended for publishing to the npm registry.
```
{
  "name": "your-project-name",
  "version": "1.0.0",
  "private": true,
  // other fields...
}
```

Create a `release.config.js` file in your project's root directory with the following configurations.
```
module.exports = {
  branches: ['main'],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    '@semantic-release/changelog',
    '@semantic-release/github',
    [
      '@semantic-release/git',
      {
        assets: ['package.json', 'CHANGELOG.md'],
        message: 'chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}',
      },
    ],
  ],
};
```

### Set up GitHub Actions Workflow

Create a workflow file at `.github/workflows/release.yml` with the following content:
```
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20.8.1'

      - name: Install dependencies
        run: npm ci

      - name: Run semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: npx semantic-release
```

## [Semantic Versioning](https://semver.org/) (SemVer)

`MAJOR.MINOR.PATCH`

### [1.6.2]
- **MAJOR** version changes: Break backward compatibility or introduce breaking changes.
- **MINOR** version changes: Add new features in a backward-compatible manner.
- **PATCH** version changes: Bug fixes or minor improvements that do not break compatibility.

**MAJOR Changes**

Terms: `BREAKING CHANGE`, `major`, `refactor`, `breaking`

These commits represent changes that **break backward compatibility**, such as removing a feature, changing the API, or making other significant alterations that will require users or other developers to modify their code.

**MINOR Changes**

Terms: `feat`, `feature`, `enhancement`, `add`

These commits introduce **new features or functionalities** in a backward-compatible manner. The change does not affect existing features or cause any disruptions to current users.

**PATCH Changes**

Terms: `fix`, `bugfix`, `patch`, `test`, `chore`, `docs`, `correct`, `resolve`, `ci`

These commits address **bugs or issues** in the codebase without introducing new features or breaking any existing functionality. They are used for minor fixed or improvements.

## [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) (example)

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

| [Release type](https://nitayneeman.com/blog/understanding-semantic-commit-messages-using-git-and-angular/) | Commit Message |
| --- | --- |
| ~~Patch~~ Fix Release | `fix(pencil): stop graphite breaking when too much pressure applied` |
| ~~Minor~~ Feature Release | `feat(pencil): add 'graphiteWidth' option` |
| ~~Major~~ Breaking Release (Note that the `BREAKING CHANGE:` token must be in the footer of the commit) | `BREAKING CHANGE: The graphiteWidth option has been removed. The default graphite widht of 10mm is always used for performance reasons.` |