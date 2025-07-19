+++
title = "Deploy your targets rapidly"
author = ["Abhimanyu G"]
description = "Deploying your cross-compiled targets to remote board via Emacs magic"
date = 2025-07-19T00:00:00+05:30
tags = ["software", "emacs", "embedded-systems"]
draft = false
+++

I am an _Embedded Software Engineer_ by profession. That means, I develop code for custom hardware (think on the lines of raspberry pi). Usually, the machine I develop code on is faster than the machine we develop code for.

More often than not, the machine I develop code on is a variant of _x86 architecture_ and the machine I want to run my code is a variant of _ARM architecture_. The developed code is [cross compiled](https://en.wikipedia.org/wiki/Cross_compiler) on the host (x86) machine to run on the target (ARM) machine.

The built binaries/libraries are then copied to the remote target board and executed there.

My workflow looks something like this

{{< figure src="/ox-hugo/embedded-workflow.svg" >}}

However, as a developer building multiple binaries and/or libraries through out the day, it is tedious to constantly copy the built artifacts to target board.

I began to wonder,

> Can I streamline this better with Emacs ?

The answer is almost always **Of course!**. Here I preset the approach and the thought process that went in for this implementation


## Requirements {#requirements}

Based on my workflow, I had the following requirements


#### Project specific build targets {#project-specific-build-targets}

Each Project will have a specific artifact. It could be a executable, library, configuration file etc.. I have to figure out a way to have project specific deploy-able assets. The assets should auto-switch when I change a project


#### Target address generation {#target-address-generation}

I copy the assets over network. So, I need a _&lt;user&gt;_ and an _&lt;ip-address&gt;_ to reach the board over network. The _ip-address_ might change when the target board is rebooted. So, I need a way to store this _ip-address_ and use it dynamically when transferring the assets.


#### Clean and easy UI {#clean-and-easy-ui}

If a project generates more than 1 artifact, I need to choose which artifact to push to the remote target. So, it helps to have a _choose-able_ option to transfer


## Implementation {#implementation}


#### Project specific build targets {#project-specific-build-targets}

One of the requirement is _project specific build targets_. Emacs has a strong notion of what a "project" is. Here is a quote from the [documentation](https://www.gnu.org/software/emacs/manual/html_node/emacs/Projects.html)

> A project is a collection of files used for producing one or more programs. Files that belong to a project are typically stored in a hierarchy of directories; the top-level directory of the hierarchy is known as the project root.
>
> Whether a given directory is a root of some project is determined by the project-specific infrastructure, known as project back-end. Emacs currently supports two such back-ends: VC-aware (see Version Control), whereby a VCS repository is considered a project; and EDE (see Emacs Development Environment). This is expected to be extended in the future to support additional types of projects.

In summary, any root folder that has a `.git` (for VC aware) and/or `.dir-locals.el` (for Emacs Dev env) is considered a project. They need not be mutually exclusive.

I added a `.dir-locals.el` file at the root of my project with following content

```emacs-lisp
;;; Directory Local Variables            -*- no-byte-compile: t -*-
;;; For more information see (info "(emacs) Directory Variables")

((prog-mode
  . ((eval
      .(setq custom/project-deploy-alist
           '(("item-name-1" "/path1/on/host" "/tmp/")
             ("item-name-2" "/path2/on/host" "/path/to/item-2")
             ("item-name-3" "/path3/on/host" "/path/to/item-3")))))))

```

What this alien looking code effectively does is

```text
When in 'prog-mode', assign variable 'custom/project-deploy-alist'
with a list-of-lists that is of the format
((<name> <local-path-on-host> <path-on-remote>)
 (<name> <local-path-on-host> <path-on-remote>))
```

This variable is defined in my `init.org` / `init.el` file like so

```emacs-lisp
(defvar-local custom/project-deploy-alist
    '(("dummy" "/tmp" "/tmp/")
      ("item-name-2" "/path/on/host" "/path/to/item-2"))
    "Project specific deploy alist set from '.dir-locals.el' file at project root.")
```


#### Pre-requisite {#pre-requisite}

In the previous section, I took care of "telling emacs what and where to push" but, I have not yet defined the remote target. In my case, the username of my embedded board is same always, but the IP address might change due to DHCP (dynamic IP allocation). Overtime, I realised, it is convenient to store the IP address of the board in an _Emacs register_ and use it everywhere. So, that's what I did

When you execute `C-x r s b` on the IP address (ex: 192.168.255.252), Emacs stores the IP address in register labeled **"b"**

Additionally, I prefer using a non-interactive shell to make the transfers happen. That implies, I cannot enter _password_ when transferring. A workaround would be to use `ssh-copy-id <user>@<ip>`. This is a one-time configuration


#### Transfer of build artifacts {#transfer-of-build-artifacts}

Checklist

-   Define a variable that lists what needs to be transferred and where it needs to be put... ☑️
-   Store IP address in a Emacs register **"b"**... ☑️
-   Configure password less transfer with `ssh-copy-id`... ☑️

The heavy lifting is done by my custom elisp function described in my _init.org_ file

```emacs-lisp
(defun custom/project-rsync-deploy-to-target (&optional user ip)
  "Prompt user to select an item to be deployed to target.
 ;; Remember to 'ssh-copy-id user@ip' before this command.
 ;; This command does not accept any passwords by default
  - target board 'ip' to be stored in @register b
  - 'user' defaults to 'root' if not provided"
  (interactive)
  (let* ((remote-target
        (format "%s@%s"
                (if (null user) "root" user)
                (if (null ip) (get-register ?b) ip)))
       (names (mapcar #'car custom/project-deploy-alist))
       (choice (completing-read "Select deploy item: " names nil t))
       (paths (cdr (assoc choice custom/project-deploy-alist)))
       (local-path (nth 0 paths))
       (remote-path (nth 1 paths))
       (rsync-command-string
        (format "rsync -qavz %s %s "
                (shell-quote-argument local-path)
                (shell-quote-argument (format "%s:%s" remote-target remote-path)))))
    (message "Executing shell command %s" rsync-command-string)
    (async-shell-command rsync-command-string nil nil)))

```

During implementation, I learnt about `rsync` a quicker transfer utility than standard `scp`. Well, that is what happens in a day on Emacs world 😀

The code is essentially doing the following,

-   Construct the remote address in the form `user@ip-addr` and assign it to _remote-target_
-   When this elsip function is executed, provide choice of _names_ as defined in the first element of the list in _.dir-locals.el_
-   When the user chooses an option, get the 2nd element (a.k.a local-path to get the artifact from) and 3rd element (a.k.a remote-path where to place this artifact on remote target)
-   Construct the command
    ```shell
    rsync -qavz /path/on/local-machine user@remote-ip:/path/on/remote-machine
    ```
-   Inform user the final command that would be executed
-   Finally, execute the command in `async shell`

This is how it looks in practice. Transferring dummy files to a rigged up docker setup
![](/images/remote-deploy-demo.gif)

It was a fun implementation. Learnt a bit more 😃

If you like my work, consider sharing. Cheers 🍻
