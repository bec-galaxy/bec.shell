<img align="right" width="22%" src="https://raw.githubusercontent.com/patbec/zsh-console-icons/master/zsh-console-auto.svg" alt="zsh console icon"/>

# Shell Role for Ansible

This is a simple role to install and set up the **zsh-shell** on Linux.

The following steps are supported:
- Install zsh with custom packages
- Set zsh as default shell for the specified users
- Optional distribution of configuration files

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

> **Molecule:** Unit test fails if a symbolic link like `/bin/zsh` is specified for `zsh_executable`.

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

## Examples

Here are a few examples of how this role can be used.

  1. [Basic](#basic)<br>Install zsh and set it as default shell for the ansible user.
  2. [Multiple users](#multiple-users)<br>Install zsh and set it as default shell for a list of specified users.
  3. [User defined config file](#user-defined-config-file)<br>Distribute a custom `.zshrc` file for a list of users.
  4. [Custom dependencies](#custom-dependencies)<br>Distribute a custom `.zshrc` file with a dependency.
  5. [Extra functions](#extra-functions)<br>Distribute a custom `.zshrc` file with `autosuggestions` and `syntax-highlighting` functions.
  6. [Complete](#complete)<br>Full example of a zsh shell.<sup>[Sample Preview](#preview)</sup>

### Basic

Install zsh and set it as default shell for the [ansible_user](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#connection-variables).
- Install the zsh package.
- Set zsh as default shell for the `ansible_user`.

```yaml
- name: zsh
  hosts: all
  roles:
    - bec.shell.zsh
  tasks:
    - ansible.builtin.debug:
        msg: "ZSH has been installed and set as default for the user {{ ansible_user }}."
```

### Multiple users

Install zsh and set it as default shell for a list of specified users.
- Install the zsh package.
- Set zsh as default shell for the specified users.

```yaml
- name: zsh
  hosts: all
  vars:
    zsh_users:
      - lorem
      - ipsum
  roles:
    - bec.shell.zsh
  tasks:
    - ansible.builtin.debug:
        msg: "ZSH was installed and set as default for 2 users."
```

### User defined config file

Distribute a custom `.zshrc` file for a list of users.
- Install the zsh package.
- Set zsh as default shell for the specified users.
- Distribute a custom `.zshrc` for each user.

```yaml
- name: zsh
  hosts: all
  vars:
    zsh_config_backup: false
    zsh_config_overwrite: true

    zsh_users:
      - lorem
      - ipsum

    zsh_users_config:
      - template: "zshrc.j2"
        filename: ".zshrc"

  roles:
    - bec.shell.zsh
  tasks:
    - ansible.builtin.debug:
        msg: "ZSH was installed and a custom file was distributed."
```

`zshrc.j2` file in your **templates folder**:

```jinja
{% if zsh_config_overwrite is true %}
#
# {{ ansible_managed }}
#
{% endif %}

# Add your content here.

```

### Custom dependencies

Distribute a custom `.zshrc` file with a dependency.
- Install the zsh package.
- Set zsh as default shell for the specified users.
- Distribute a custom `.zshrc` for each user.
- Install a dependency.

```yaml
- name: zsh
  hosts: all
  vars:
    zsh_config_backup: false
    zsh_config_overwrite: true

    zsh_users:
      - lorem
      - ipsum

    zsh_users_config:
      - template: "zshrc.j2"
        filename: ".zshrc"

    zsh_dependencies:
      - exa
  roles:
    - bec.shell.zsh
  tasks:
    - ansible.builtin.debug:
        msg: "ZSH has been installed and exa is used to list files."
```

`zshrc.j2` file in your **templates folder**:

```jinja
{% if zsh_config_overwrite is true %}
#
# {{ ansible_managed }}
#
{% endif %}

alias ls='exa --group-directories-first'
alias ll='exa --group-directories-first --all --long --binary --group --classify --grid'
alias la='exa --group-directories-first --all --long --binary --group --header --links --inode --modified --blocks --time-style=long-iso --color-scale'
alias lx='exa --group-directories-first --all --long --binary --group --header --links --inode --modified --blocks --time-style=long-iso --color-scale --extended'
```

> [exa](https://the.exa.website/) is an improved file lister with more features and better defaults. It uses colours to distinguish file types and metadata.

### Extra functions

Distribute a custom `.zshrc` file with [autosuggestions](https://github.com/zsh-users/zsh-autosuggestions#zsh-autosuggestions) and [syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting#zsh-syntax-highlighting-) functions.
- Install the zsh package.
- Set zsh as default shell for the specified users.
- Distribute a custom `.zshrc` for each user.
- Install [autosuggestions](https://github.com/zsh-users/zsh-autosuggestions#zsh-autosuggestions) and [syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting#zsh-syntax-highlighting-).

```yaml
- name: zsh
  hosts: all
  vars:
    zsh_config_backup: false
    zsh_config_overwrite: true

    zsh_users:
      - lorem
      - ipsum

    zsh_users_config:
      - template: "zshrc.j2"
        filename: ".zshrc"

    zsh_dependencies:
      - zsh-autosuggestions
      - zsh-syntax-highlighting
  roles:
    - bec.shell.zsh
  tasks:
    - ansible.builtin.debug:
        msg: "ZSH was installed and additional functions were enabled."
```

`zshrc.j2` file in your **templates folder**:

```jinja
{% if zsh_config_overwrite is true %}
#
# {{ ansible_managed }}
#
{% endif %}

autoload colors && colors

source  /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
source  /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

### Complete

Full example of a zsh shell.
- Install the zsh package.
- Set zsh as default shell for the specified users.
- Distribute a custom `.zshrc` for each user.
- Install [autosuggestions](https://github.com/zsh-users/zsh-autosuggestions#zsh-autosuggestions), [syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting#zsh-syntax-highlighting-) and [exa](https://github.com/ogham/exa).
- Set `history`, `color` and `cd` settings.

```yaml
- name: zsh
  hosts: all
  vars:
    zsh_config_backup: false
    zsh_config_overwrite: true

    zsh_users:
      - lorem
      - ipsum

    zsh_users_config:
      - template: "zshrc.j2"
        filename: ".zshrc"

    zsh_dependencies:
      - exa
      - zsh-autosuggestions
      - zsh-syntax-highlighting

    zsh_additional_lines: ""
  roles:
    - bec.shell.zsh
  tasks:
    - ansible.builtin.debug:
        msg: "ZSH was installed."
```

`zshrc.j2` file in your **templates folder**:

<details>
  <summary><b>Click here to see the content</b></summary>

  ```jinja
  {% if zsh_config_overwrite is true %}
  #
  # {{ ansible_managed }}
  #
  {% endif %}

  autoload colors && colors

  PROMPT="%(?:%{$fg_bold[green]%}➜ :%{$fg_bold[red]%}➜ )"
  PROMPT+=" %{$fg[cyan]%}%c%{$reset_color%} "

  source  /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
  source  /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

  alias ls='exa --group-directories-first'
  alias ll='exa --group-directories-first --all --long --binary --group --classify --grid'
  alias la='exa --group-directories-first --all --long --binary --group --header --links --inode --modified --blocks --time-style=long-iso --color-scale'
  alias lx='exa --group-directories-first --all --long --binary --group --header --links --inode --modified --blocks --time-style=long-iso --color-scale --extended'

  HISTFILE="$HOME/.zsh_history"
  HISTSIZE=50000
  SAVEHIST=50000

  setopt INC_APPEND_HISTORY
  setopt AUTOCD

  # set PATH so it includes user's private bin if it exists
  if [ -d "$HOME/bin" ] ; then
      PATH="$HOME/bin:$PATH"
  fi

  # set PATH so it includes user's private bin if it exists
  if [ -d "$HOME/.local/bin" ] ; then
      PATH="$HOME/.local/bin:$PATH"
  fi

  {% if zsh_additional_lines is defined and zsh_additional_lines | length %}
  #
  # Host specific additions
  #
  {{ zsh_additional_lines }}
  {% endif %}
  ```
</details>

#### Preview

A preview of this configuration:
![zsh console](https://raw.githubusercontent.com/patbec/zsh-console-icons/master/zsh-console.svg)
<sub>You can find the used [ANSI console colors here](https://github.com/patbec/zsh-console-icons#colors).</sub>

#### Apply local

This last example can be found in the [docs](../../docs/example) folder - in slightly modified form, you can apply it with the following local command:
```shell
ansible-galaxy collection install bec.shell --force
ansible-playbook ./docs/example/playbook.yml --ask-become-pass
```

<sup>Note: Playbook in the docs folder <u>does not</u> overwrite any existing zsh configuration files.</sup>

---

&uarr; [Back to top](#)