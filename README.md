# Git Repo Watcher

A simple bash script to watch a git repository and pull upstream changes if available.

### Requirements

* Bash with Version > 3
* Tested on Ubuntu, Debian, MacOS, [Windows Git Shell](https://git-scm.com/download/win), [Windows Subsystem for Linux (WSL)](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71)

Basically, it will work anywhere you can install bash.
If something doesn't work, please let me know.

### Usage

To start, only the file path to your git repository is needed. This will start a watcher that looks for changes every 10 seconds:
```bash
./git-repo-watcher -d "/path/to/your/repository"
```

The time interval can be changed by passing it to `-i`:
```bash
./git-repo-watcher -d "/path/to/your/repository" -i 60 #seconds
```

You can add your own logic to `git-repo-watcher-hooks`.
For example, you can start your build process in case of changes:

```bash
# $1 - Git repository name
# $2 - Branch name
# $3 - Commit details
change_pulled() {
    echo "Starting build for commit: $@"
    ./your-build-script.sh
}
```

If you have more than one repository you can pass a copy of the `git-repo-watcher-hooks` file like so:
```bash
./git-repo-watcher -d "/path/to/your/repository" -h "/path/to/your/hooks-file"
```

Make sure your local repository is tracking a remote branch, otherwise the script will fail.

### Private repositories

The script works with private repositories.

First configure a password cache with `git config --global credential.helper "cache --timeout=60"`.
Make sure that the `timeout` is greater than the time interval given to the script. Both are given as seconds.

At startup, the program will execute `git fetch` and ask for your credentials.

If you want it to run in the background as a daemon process, you have to execute `git fetch` beforehand.
For example:

```bash
cd "/path/to/your/repository"
git config --global credential.helper "cache --timeout=60"
git fetch

# Checking exit code
if [[ $? -eq 1 ]]; then
    echo "Wrong password!" >&2
    exit 1
fi

# Disown process
./git-repo-watcher -d "/path/to/your/repository" > "/path/to/your/logfile.txt" & disown
```

### Using on Windows 10

The easiest way is to install [Git Shell](https://git-scm.com/download/win), which also comes with bash. 
The only thing you have to consider are the file separators.
The Unix format should be used here:

`C:\YourGitRepository` &#8594; `/C/YourGitRepository`

It is a little more difficult with [WSL](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71). This must first be installed and configured via the Windows Store.
The file structure is also slightly different:

`C:\YourGitRepository` &#8594; `/mnt/c/YourGitRepository`

Note: In [Ubuntu 20.04 LTS](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71) there is a [bug](https://github.com/microsoft/WSL/issues/4898) that was introduced by Microsoft.
The `sleep` command does not work. However, this is used by the git-repo-watcher. A temporary fix for this can be found in the [same thread](https://github.com/microsoft/WSL/issues/4898#issuecomment-638649617).

[Ubuntu 18.04 LTS](https://www.microsoft.com/en-us/p/ubuntu-1804-lts/9n9tngvndl3q) does not have this problem.

### Tests

The test suite `git-repo-watcher-tests` is using the test framework [shunit2](https://github.com/kward/shunit2), it will be downloaded automatically to your `/tmp` folder.
The script doesn't need any other dependencies and doesn't need an internet connection. 

The tests create several test Git repositories in the folder: `/tmp/git-repo-watcher`.

A git user should be configured, otherwise the tests will fail.
Here you can see if this is the case:
```bash
git config --list
```

Otherwise you can set it as follows:
```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```


