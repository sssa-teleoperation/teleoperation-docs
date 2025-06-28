# Welcome to HRI Teleoperation Tutorial

If you wish to develop cool algorithm or experiments using our teleoperation platform, you are in the right place!

## Code Editor

You will need [vscode](https://code.visualstudio.com/), so download it and install it before diving in!

### Extensions

To get the best from VS Code, we suggest that you install the following extensions

- [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python): fundamental coding languages support (previewing function arguments and a lot more);
- [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack): very useful to work remotely on powerful shared working stations.
- [Container Tools](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-containers): similarly, but for programming "virtual machines".
- [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph): basic software versioning visualization client.

## GitHub

If you don't have a [GitHub](https://github.com) account yet, you can obtain one, it's free and it's a nice web platform where to host your code. For now you may just want to acknowledge that GitHub hosts your code in form of `git` repositories, and it's extremely comfortable to track changes on your code.

Then you need to setup your pc to be able to access your repositories on GitHub, for instance to download or to sync them whenever needed.

**Note:** The following setup procedure **must** be repeated for each pc you may want to work on, and of course you need an [installation](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) of `git`.

##### SSH authentication

So first let's set up a pair of encrypted keys, one for the pc, one for GitHub, so that GitHub knows your pc is allowed to acces your private assets on the cloud. 

Open a terminal and type

    ssh-keygen -t ed25519 -C "your_email@example.com"

this will generate the keys. We suggest to leave default path and empty passphrase. Then add them to the authentication agent on your pc

=== "Linux"

    ```bash
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519
    ```

=== "Windows"

    ```powershell
    Get-Service -Name ssh-agent | Set-Service -StartupType Manual
    Start-Service ssh-agent
    ssh-add c:/Users/YOU/.ssh/id_ed25519
    ```

Then follow [this guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#adding-a-new-ssh-key-to-your-account) to add the public key of the pair in GitHub. 

Finally test the authentication

    ssh -T git@github.com

##### Packages authentication

Not all assets are hosted on GitHub as repositories, sometimes you stor a large amount of data into packages. Follow Step 1 and Step 2 [here](https://medium.com/devopsturkiye/pushing-docker-images-to-githubs-registry-manual-and-automated-methods-19cce3544eb1).