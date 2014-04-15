clonepwn
========

The purpose of this repository is to demonstrate arbitrary code
execution after cloning a repository when using vulnerable versions of
`__git_ps1` in two- or three-argument mode.

Instructions
------------

If you are using bash:

  1. Get [`git-prompt.sh`](https://git.kernel.org/cgit/git/git.git/tree/contrib/completion/git-prompt.sh)
     from the [`git.git` repository](https://git.kernel.org/cgit/git/git.git)
     (in the `contrib/completion` subdirectory).

  2. Follow steps 1, 2, and 3b at the top of the `git-prompt.sh` file.

  3. Clone this repository:

        git clone https://github.com/richardhansen/clonepwn.git

  4. `cd` into the new `clonepwn` directory.
