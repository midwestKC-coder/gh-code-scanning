# gh-code-scanning

A GitHub CLI extension for GitHub Code-Scanning!

### Dependencies

- [`gh`](https://cli.github.com/)
- `git`
- Python (version 3.5 or higher)
- Unix-like system (Linux, macOS, or BSD)

### Installation

```shell
# Disable gh's "interactive" mode, since gh-code-scanning is designed to be used in scripts.
gh config set prompt disabled

# Configure gh to use SSH instead of HTTPS for git operations. This is required.
gh config set git_protocol ssh

# Install the extension.
gh extension install https://github.com/mario-campos/gh-code-scanning
```

### Usage



#### Enable

Use the `enable` subcommand to "deploy" GitHub Code Scanning with CodeQL, by adding a GitHub Actions workflow file to a repository.

```
usage: gh code-scanning enable [-h] [-f] [--git-push] [-m MESSAGE] repos [repos ...]

positional arguments:
  repos

optional arguments:
  -h, --help            show this help message and exit
  -f, --force           overwrite existing workflow file.
  --git-push            do not create PR; push commit to HEAD of default branch.
  -m MESSAGE, --message MESSAGE
                        specify the pull-request/commit message.
```

In its most simplest form, gh-code-scanning can setup GitHub Code Scanning with CodeQL on a single repository:

```shell
gh code-scanning enable owner/repo
```

However, with the use of `xargs`, we can automate this process for every repository in an organization. For example, the following command will enable all repositories in the organization _foo_:

```shell
gh repo list foo --json nameWithOwner --jq '.[].nameWithOwner' | xargs gh code-scanning enable
```

We can take this one step further even, by applying some `jq` magic to limit our "deployments" to only some repositories. For example, if you only wanted to enable GitHub Code Scanning with CodeQL on all repositories in the organization _foo_ that have CodeQL "interpreted" languages (`javascript`, `python`, `go`, `ruby`) and none of the CodeQL "compiled" languages (`java`, `csharp`, `cpp`), run the following command:

```shell
gh repo list foo \
  --json nameWithOwner,languages \
  --jq '
  .[] | (.languages) = [.languages[].node.name] |
  select(.languages | all(. != "Java" and . != "C#" and . != "C" and . != "C++")) |
  select(.languages | any(. == "JavaScript" or . == "TypeScript" or . == "Python" or . == "Go" or . == "Ruby")) |
  .nameWithOwner' \
| xargs gh code-scanning enable
```

#### Alerts

The `alerts` subcommand is used to output the set of GitHub Code Scanning alerts of a repository.

```
usage: gh code-scanning alerts [-h] repos [repos ...]

positional arguments:
  repos

optional arguments:
  -h, --help  show this help message and exit
```

To list the alerts of a single repository, run the following command:

```shell
gh code-scanning alerts owner/repo
```

With the use of `xargs`, it is also possible to list the alerts of all repositories in a GitHub organization. By using `gh` to list all of the repositories in an organization, and then passing that list to gh-code-scanning, we can collect the alerts of every repository in an organization by having gh-code-scanning iterate through every repository. For example, the following command will output the alerts of every repository in the organization _foo_:

```shell
gh repo list foo --json nameWithOwner --jq '.[].nameWithOwner' | xargs gh code-scanning alerts
```

#### Analyses

The `analyses` subcommand is used to output or delete the GitHub Code Scanning analyses of a repository.

```
usage: gh code-scanning analyses [-h] [-d] [-r REF] repo

positional arguments:
  repo

optional arguments:
  -h, --help         show this help message and exit
  -d, --delete
  -r REF, --ref REF
```

To output a repository's set of analyses, run gh-code-scanning _without_ the delete flag (`-d` or `--delete`).

```shell
gh code-scanning analyses owner/repo
```

To delete a repository's set of analyses, run gh-code-scanning _with_ the delete flag (`-d` or `--delete`).

```shell
gh code-scanning analyses --delete owner/repo
```

You can also limit the output or deletion to a particular Git ref (e.g. branch) by specifying the ref name as an argument to the ref flag (`-r` or `--ref`):

```shell
# output analyses of the 'develop' ref
gh code-scanning analyses --ref develop owner/repo

# delete analyses of the 'develop' ref
gh code-scanning analyses --delete --ref develop owner/repo
```
