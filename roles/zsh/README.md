# Shell Role for Ansible

This is a simple role to install and set up the **zsh-shell** on Linux.

The following steps are supported:

* Install zsh with custom packages
* Set zsh as default shell for the specified users
* Optional distribution of configuration files

This role is kept simple, uses the standard package manager and contains minimal overhead. If you have any problems, feel free to create an issue.

## Variables

Default variables in this playbook.

| Name                 | Description                                            | Default Value        |
| -------------------- | ------------------------------------------------------ | -------------------- |
| zsh_config_mode      | Default permissions for distributed files.             | `0644`               |
| zsh_config_backup    | Creates a backup before overwriting.                   | `true`               |
| zsh_config_overwrite | Overwrites the existing file if there are differences. | `false`              |
| zsh_users_config     | A list of zsh files to be distributed for a user.      | `[]`                 |
| zsh_system_config    | A list of zsh files to be distributed globally.        | `[]`                 |
| zsh_users            | A list of users who should get zsh as default shell.   | `{{ ansible_user }}` |
| zsh_dependencies     | A list of additional packages to install.              | `[]`                 |
| zsh_global           | Target for global zsh configuration files.             | `/etc/`              |
| zsh_executable       | Path to the executable.                                | `/usr/bin/zsh`       |

The `zsh_users_config` property is a dictionary:

```yaml
zsh_users_config:
  - template: "zshrc.j2"
    filename: ".zshrc"
```

This basic example distributes a `.zshrc` for a user. Ansible looks for the `zshrc.j2` file in your `template` folder.

### Paths

Here is a list of possible filenames from the [zsh documentation (5.2 Files)](https://zsh.sourceforge.io/Doc/Release/Files.html#Files-1).

#### Users

```yaml
zsh_users_config:
  - template: "zshrc.j2"
    filename: ".zshrc"
  - template: "zshenv.j2"
    filename: ".zshenv"
  - template: "zprofile.j2"
    filename: ".zprofile"
  - template: "zlogin.j2"
    filename: ".zlogin"
  - template: "zlogout.j2"
    filename: ".zlogout"
```

<i>The target path is the HOME directory of the respective user.</i>

#### System

```yaml
zsh_system_config:
  - template: "zshrc.j2"
    filename: "zshrc"
  - template: "zshenv.j2"
    filename: "zshenv"
  - template: "zprofile.j2"
    filename: "zprofile"
  - template: "zlogin.j2"
    filename: "zlogin"
  - template: "zlogout.j2"
    filename: "zlogout"
```

<i>The target path is the `/etc/` directory, this can be changed in the variable `zsh_global`.</i>

[Click here](https://github.com/bec-galaxy/bec.shell/wiki) to show examples.
