# Git Attribute Test

This is a contrived example of using GIT attributes.

The single biggest drawback with using `gitattributes` to modify files in git is 
that the attribute configuration cannot be part of the repository.

As a result of this, you must clone the repository, implement the configuration changes, then check the repository
out again.

In this example, the required configuration is in a file called `.gitconfig` at the root of the repository.  You will need
to configure git to include the `.gitconfig` file by adding an `include` directive to the local git configuration.

For git to implement the changes, the files will needed to be checked out again. This can be done with `git reset`, or
(depending on git version) removing the affected files (bash.1 and bash.2) and checking them out again with `git checkout -- .`

## Testing.

1. Clone this repository.

   There are two files in the repository, bash.1 and bash.2.  Both are text files creating via `man bash`

   The file `bash.1` requires `base64` to be converted back to text, while the file `bash.2` requires the linux `rev` command
   to complete conversion.

   `git status` will show a clean repository initially.

   `md5sum bash.?` will produce the following output showing that the files have not been filtered.

       0affa1cca7fa9e4ad27a7da9d48d735e  bash.1
       f2ffc3d825c429a950794fa73afda5f1  bash.2

2. Update the local git configuration to include the local `.gitconfig` file at the root of the repository.

       git config --local include.path ../.gitconfig

   or

       ./install.sh

3. At this point, git *should* immediately show that the repository is modified.
   `git status` will show that the files `bash.1` and `bash.2` are modified.

> Note: If `git status` does _NOT_ show any changes, you may need to remove `bash.1` and `bash.2` to force git to acknowledge the change.

4. Re-checkout the branch

   Either

        git checkout -- .

   or 

        git reset --hard HEAD

    At this point, the two bash files should now be identical.

    `md5sum bash.?` will now produce the following output.

        aeedc190af950a99fc0fc3430f341542  bash.1
        aeedc190af950a99fc0fc3430f341542  bash.2

## Diagnostics

To assist with understanding what is happening, simply use the following command

    export GIT_TRACE=1

With GIT_TRACE set, git commands will produce some diagnostics.

Example:

```bash
$ export GIT_TRACE=1
$ git status
11:09:00.243593 git.c:743               trace: exec: git-status
11:09:00.243630 run-command.c:667       trace: run_command: git-status
11:09:00.245254 git.c:455               trace: built-in: git status --short --branch
11:09:00.246283 run-command.c:667       trace: run_command: base64
11:09:00.252118 run-command.c:667       trace: run_command: rev
## main...origin/main
```

```bash
$ git diff
11:16:49.719979 git.c:455               trace: built-in: git diff
11:16:49.720788 run-command.c:667       trace: run_command: unset GIT_PAGER_IN_USE; LV=-c 'less -RFX -x4'
11:16:49.721352 run-command.c:667       trace: run_command: base64
11:16:49.727872 run-command.c:667       trace: run_command: base64
11:16:49.734732 run-command.c:667       trace: run_command: 'base64 -d'
11:16:49.737877 run-command.c:667       trace: run_command: cat /tmp/lgFCh1_bash.1
11:16:49.738966 run-command.c:667       trace: run_command: cat bash.1
diff --git bash.1 bash.1
index b053375..12cb285 100644
...
```
