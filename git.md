# `git`

## alias

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```

## config

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

## tagging

### Create tags

* **Lightweight tags**: basically the commit checksum stored in a file — no other information is kept. no `-a` or `-m` or `-s` options.

  ```bash
  git tag v1.4-lw
  ```

* **Annotated tags**: stored as full objects in the Git database. They’re checksummed; contain the tagger name, email, and date; have a tagging message; and can be signed and verified with GNU Privacy Guard (GPG). It’s generally recommended that you create annotated tags so you can have all this information unless you want a temporary tag

  ```bash
  git tag -a v1.4 -m "my version 1.4"
  ```

* tagging a previous commit:

  ```bash
  git tag -a v1.2 9fceb02
  ```

### Show tags

* list all tags:

  ```bash
  git tag
  ```

* show a specific tag:

  ```bash
  git show v1.4
  ```

### Push tags

* To push a single tag:

  ```bash
  git push origin v1.4
  ```

* or to push all tags:

  ```bash
  git push origin --tags
  ```

### Delete tags

* delete locally:

  ```bash
  git tag -d v1.4
  ```

* delete from a remote server:

  ```bash
  git push <remote> :refs/tags/<tagname> # option 1
  git push origin --delete <tagname> # option 2
  ```

## Misc

* Find root directory for a git repo

  ```bash
  git rev-parse --show-toplevel
  ```
