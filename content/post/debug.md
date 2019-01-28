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
a bug with `pip`, `pip install . -U` is very slow with a local `.git` directory (because `pip` basically needs
to copy everything to a temporary directory) so we have to figure out something to assist the debug.

## Tool 1: in-place installation.

First, run


```
% pip uninstall sos
```

to remove system installed `sos`, which will do nothing but interfere with locally installed sos.

Second, run

```
% pip install . -e

```

The `-e` option installs `sos` inplace. It will install a `sos` command but the `sos` command
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

This will save the need to run `pip install . -U` repeatedly during development.


## Too 2: debuggin with VS code

There can be multiple platform for debugging but I am just showing what works on my mac.

1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install [Python extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
3. Follow the tutorial to install a linter.
4. C-S-P (open command pallete), search shell, and install `code` command to `$PATH`, which is helpful to start
  `code` in the project path.


Now, let us assume that we always want to debug the execution of a workflow defined in the local
directory with name `test.sos`.

1. Start VS Code with command `code .` under the `sos` directory.
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


Note that:

1. Debug, toggle break point is useful
2. You can use function `break_point()` to trigger a break point in the code.

