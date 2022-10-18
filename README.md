## What this is

This is a guide, for how I setup my WSL (Windows Subsystem for Linux) 2 as a development backend.

## Why?

Windows has a couple of problem when it comes to using it for software development.
One of the things which pushed me over the edge are the inefficient file watchers and the missing native support for a container runtime.

Another pretty helpful aspect is that you can have one WSL machine for each project, to keep them isolated.
Additionally you can keep them at different versions, depending on your project.

I could not switch to linux directly since my company does not support a linux distribution as operating system or dual boot.
If your company / setup supports this then you probably want to use it directly for an even better integration.

## Caveats

I use Webstorm running on Windows as frontend, this is a specific solution for this IDE and has some limitation in regard to how it uses the host.

If your IDE / workflow does not support development over SSH then you can use the virtual directory and use the IDE / workflow on that.

## Prerequisites

Your Windows computer needs to support WSL 2, this guide wont work if it does not.

## Goal

The goal is, that you can start an IDE on your Windows computer and utilize Windows applications such as the Office suite, special VPN settings, or network drives and authentication and still utilize the full functionality and performance of a unix system for development applications and workflows.

The windows IDE therefore connects via SSH or via the virtual network directory on the development files and edits them.
Your native linux workflow then processes those files after which you can inspect them.

The linux system is in many cases much faster and more suitable for many development tasks which improves your development cycle.

## Setup

### SSH Setup

1. (optional) Install the [Terminal application](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701) for a better terminal
2. Open the Windows PowerShell (in the Terminal) to generate ssh secrets via `ssh-keygen -t rsa -b 4096 -f C:\Users\<YOUR-WINDOWS-USERNAME>\.ssh\id_rsa`
4. Create a ssh config by creating the config file `C:\Users\<YOUR-WINDOWS-USERNAME>\.ssh\config`
    ```text
    Host ubuntu
    Hostname localhost
    User <YOUR-LINUX-USERNAME>
    PubKeyAuthentication yes
    IdentityFile ~/.ssh/id_rsa
    ```
    Remark: If you already got another WSL instance you need to enter the actual IP adress of your Linux instance instead of localhost. 
    This IP changes on each restart and can be found via `wsl hostname -I`.
    
    Remark: You can use whatever name you want instead of `ubuntu`.
3. Install the wsl via `wsl --install -d Ubuntu`
4. Wait for the installation to progress, enter your new <YOUR-LINUX-USERNAME>
5. Update your installation via `sudo apt-get update` and `sudo apt-get upgrade`
6. (optional) Create or login to the user you want to ssh into later on
7. Create the .ssh dir `mkdir -p ~/.ssh` `chmod 700 ~/.ssh`
8. Insert the generated public key into the authorized keys file with `cp /mnt/c/Users/<YOUR-WINDOWS-USERNAME>/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys` and `chmod 711 ~/.ssh/authorized_keys`
9. (Re-) Start your ssh daemon via `sudo service ssh stop` and `sudo service ssh start` (You need to start it after every Linux start!)
10. Open another PowerShell instance
11. Connect to the WSL via ssh by using `ssh ubuntu`
12. You should be connected to the WSL via ssh
13. (optional) Install all additional packages e.g. Node.js, Yarn, Docker

### Data and file transfer

You have multiple options here, if your IDE / workflow does not support the SSH approach then you can use the virtual network drive to access the project files.

To access Windows files from Linux you can use the auto mount points which are under `/mnt` followed by the drive letter e.g. `/mnt/c/Users/Flo/my-file.txt`
To access Linux files from Windows you can use the virtual network drive located at `\\wsl$` followed by the installatio name e.g. `\\wsl$\Ubuntu\home\Flo\my-file.txt`

A heads up: Those translations of file systems (NTFS <-> ext4) are not without cost or loss of information.
Only do them sparingly and only when needed.
For example only do them when you use the IDE to modify a file or want to export a build artifact but *not* with webpack to read and bundle all files from another file systme type.

Rule of thumb: Use the file system from the operating system which also runs your build tasks.

### IDE setup

If your IDE can not use SSH as dev backend then you can just open the network drive as your project directory.
But be aware that this might be rather slow for integrated language servers (e.g. for TypeScript) only use this approad if you havily profit from the time gained by using Linux to build your project.

You can also use rsync to rsync from `/mtn/Users/<YOUR-WINDOWS-USERNAME>/projects/example/` to `/home/<YOUR-LINUX-USERNAME>/projects/example` and then build from there, to have the benfit of the faster language server and the faster build pipeline.

I used Webstorm as IDE, which supports the SSH backend.

1. Open WebStorm
2. Select the Remote Development from the possible project sources
3. Select SSH - New Connection and click on the cogwheele next to the connection dropdown
4. Enter the `Host` and `User` strings you used in the ssh config file
5. Select OpenSSH config and authentication agent
6. Click ok and then select the newly created connection then select the `Check Connection and Continue`
7. Use your terminal SSH session to checkout / create a directory for your project
8. Use this path for the project directory
9. The newest WebStorm is automaticly downloaded, installed and started on the (WSL-) Host

If everything went as it should you should not be presented with a normal WebStorm IDE windows which is connected to the WSL as a dev backend.
You can check this by opening a terminal in the WebStorm instance and check if you are on the WSL host.

All Plugins are executed on the Linux system and can therefore nativly interact with other tools, like `git` and `docker`.
The latter one is a big plus for my project.

### Troubleshooting

There are many things which might not work as discribed.
I would recommend that you start by checking the installation state of your Windows system, then the WSL abstraction and then the subsystem.
If thats all in order, then you can continue to debug / reinstall the IDE and then the project / tool configurations.

You can stop the Linux system *and* the WSL VM by running `wsl --shutdown` after that just run `wsl` to start the default distribution.
Remember to start the ssh service after each Linux start.
