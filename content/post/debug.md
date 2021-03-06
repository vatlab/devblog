---
type: "post"
draft: false
author: "Bo Peng"
title: "How to debug SoS"
date: "2017-11-29"
tags: ["debug", "pip", "vs-code"]
---

## The problem

SoS command line is generated from an entry point so there is no `sos` command if the command is not installed, 
and if we install `sos` then changes in local repository will not be reflected in the installed version. Due to
a bug with `pip`, `pip install . -U` is very slow with a local `.git` directory (because `pip` basically 
copies everything to a temporary directory before building) so it can be tedious to install `sos` to the system
directory for debugging.

## Trick 1: in-place installation.

Run


```
% pip uninstall sos
```

to remove system installed `sos`, which will do nothing but interfere with locally installed sos.

Then run

```
% pip install . -e

```

to install `sos` inplace. It will install a `sos` command but the `sos` command
will point to the local directory for package source. More specifically, it will create a file
such as

```
/Users/bpeng1/anaconda3/envs/dev/lib/python3.6/site-packages/sos.egg-link
```
with content

```
/Users/bpeng1/dev/sos/src
```

so that `sos`, when executed, will load `.py` files from local `sos/src` directory.

Now, when you make changes to the source tree of SoS, you can run it directly without having
to install `sos` again.


## Trick 2: Debugging with VS code

There can be multiple platform for debugging but I am just showing what works for me on my mac.

1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install [Python extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
3. Follow the tutorial to install a linter.
4. C-S-P (open command pallete), search shell, and install `code` command to `$PATH`, which is helpful to start
  `code` in the project path.


Now, let us assume that we always want to debug the execution of a workflow defined in the local
directory with name `test.sos`.

1. Start VS Code with command `code .` with the current directory as the project folder.
2. Click debug, add configuration, and insert the following into the configuration file

   ```
        {
            "name": "Python: SoS ",
            "type": "python",
            "request": "launch",
            "module": "sos.__main__",
            "console": "integratedTerminal",
            "args": [
                "run",
                "test.sos",
            ]
        },
   ```

   This will let `code` start sos with command `python -m sos.__main__` with the provided options. Of course
   you will need to edit the configuration file if you need to add more options.


Another tricky is that in addition to using `Debug` -> `toggle break point` from the manual, you can use
function `break_point()` to trigger a break point in the code.

