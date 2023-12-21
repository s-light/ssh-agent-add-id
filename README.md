# ssh-agent-add-id
Checks whether an identity file has already been stored by the SSH agent. If not, it adds this identity by prompting for its passphrase.

## Description
`ssh-agent-add-id` was primarily created to address a [pending issue](https://github.com/microsoft/vscode-remote-release/issues/2369) in the `VS Code` [WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) when authenticating with an SSH key that requires a passphrase, such as for a remote Git repository. If this key has not been previously added to the `SSH agent` accessible from the [WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl), `VS Code` does not prompt for the passphrase, causing operations like pushing to the remote repository to get stuck.

`ssh-agent-add-id` serves as a wrapper for the [ssh-add command](https://man.openbsd.org/ssh-add). However, unlike the latter, it does not prompt again for the passphrase and exits quietly if the key has already been added to the SSH agent. It can thus be easily executed in a `VS Code` task when a project is opened. 

And beyond remote Git repositories and WSL, `ssh-agent-add-id` can be also be used with cloud services that rely on SSH key authentication and thus reduce the number of times you need to enter your passphrases.

## Installation
### SSH agent
[ssh-add](https://man.openbsd.org/ssh-add) and [ssh-agent](https://man.openbsd.org/ssh-agent) must be installed. On Debian/Ubuntu distros, they are part of `openssh-client` package. `ssh-agent-add-id` should run smoothly on `macOS`, but it's currently not supported on `Windows` (without WSL).

[ssh-agent](https://man.openbsd.org/ssh-agent) must be running. `SSH_AUTH_SOCK` environment variable needs be set and accessible from `VS Code` environment. For Linux/WSL, in order to share a single `ssh-agent` process for all your shells, it is highly recommended to either run `ssh-agent` as an [user-level systemd service](https://gist.github.com/alexisbg/12102035851c2d0555878cfd865fac75) or to install and setup [Keychain](https://github.com/funtoo/keychain).

### ssh-agent-add-id
`ssh-agent-add-id` can be installed using `pip`:
```
pip install ssh-agent-add-id
```

### VS Code task
Add a task to your VS Code project (in `.vscode/tasks.json`). **Do not forget** to update the `"args"` key with the actual path of your private key file:
```json
    {
      "label": "Add Git SSH key to agent",
      "type": "shell",
      "command": "ssh-agent-add-id",
      "args": ["${userHome}/.ssh/<PRIVATE_KEY_FILE>"],
      "presentation": {
        "panel": "new",
        "revealProblems": "onProblem",
        "close": true
      },
      "runOptions": {
        "runOn": "folderOpen"
      },
      "problemMatcher": []
    }
```
Thanks to ["runOptions"/"runOn": "folderOpen"](https://code.visualstudio.com/docs/editor/tasks#_run-behavior), this task runs every time your project is opened, launching a new dedicated terminal. If the identity/key was already added to the `SSH agent`, this terminal closes immediately. Otherwise, it prompts for the key passphrase.

## Command line usage
```
usage: ssh-agent-add-id [-h] [--version] priv_key_path [pub_key_path]

positional arguments:
  priv_key_path  the path of the private key file
  pub_key_path   the path of the public key file in case its filename is not <priv_key_path>.pub

optional arguments:
  -h, --help     show this help message and exit
  --version      show program's version number and exit
```

## License
This project is licensed under the terms of the MIT license.
