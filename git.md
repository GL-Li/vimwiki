## Outline

- Major reference
- Workflow and SOP
- Minimal examples
- QA - quick answer to specific questions
- Raw notes

### Major references

- Udemy course: [The Git & Github Bootcamp](https://www.udemy.com/course/git-and-github-bootcamp/)
- Reference: https://git-scm.com/docs
  - exemples and discussion down to the bottom of the page for each git command.
- Book: https://git-scm.com/book/en/v2
  - this should be the read after the Udemy course.

## Workflow and SOP ===========================================================


### SOP: config a specific repo user.name, user.email, and credentials
- **use case**: my global configuration is for Bitbucket. Now I have a repo for Github and I want to set config for this repo, what should I do?
- steps to take:
    - `$ git config --list` to check current configuration
    - `git config user.email "xxxx"` to set user.email for this repo
    - `git config user.name "yyy"` to set user.name for this repo
    - `git config credential.helper store` to store credentials if not already set. It is most likely has been set globally so no need to run it.
    - credentials are stored in file `.git-credentials` in which one line for each credential, for example
        ```
        https://GL-Li:W46gARaFSAGRSfsadsag4E@github.com
        https://liguanglai:XsR36yf7hsredfsaf@bitbucket.org
        ```
    - git will automatically search for the correct credentials in above file when credentials are needed, for example, for `git push`.

,
### SOP: force line ending to work across Windows and Linux

**Line ending is a headache** when switching between Windows and Linux, but especially painful when using WSL on Windows. In WSL, the automatic line ending is Windows' crlf instead of Linux's lf. So when you clone a git repository in WSL, all line ending will changed to crlf. Some file will not run in WSL, such as `.sh` files. 

**A real example**: when install vim plugins in WSL, the plugin code files are first git cloned to the computer and then installed. Some plugins fail to install due the line ending issues.

**solution to line ending issue**: 
    - `$ git config --global core.autocrlf=input` to stop git from changing line ending when switching between Windows and Linux. Instead, the line ending will be enforced by file `.gitattribute`.
    - When creating a new git repository, create file `.gitattribute` file to force line ending of selected filetypes. (See https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings), for example: 

        ```
        * text=auto                # all text file automatic line ending
        *.bat text eol=crlf        # Windows .bat files have crlf ending
        *.sh text eol=lf           # Linux bash scripts have lf ending
        ```

**if `.gitattribute` is added after files created**, force existing file with line ending defined in `.gitattributes` by running
    ```sh
    git add . -u               # -u to update all files
    git commit -m "xxxxx"
    git add . --renormalize    # change line ending to match .gitattribute
    git status                 # or git diff to see the difference
    git commit - m "yyyyy"
    ```
## QA =========================================================================

## QA: how to push, delete, and modify a tag in remote?

- `$ git tag v0.1.23` to add a tag to current commit
- `$ git push origin v0.1.23` to push a tag to origin
- `$ git push --delete origin v0.1.23` to delete tag from origin
- `$ git tag -d v0.1.23` to delete a local tag
 

### QA: how to check edit history of a file?

- `$ git log -p path/to/file`
    - It will print out commit info and changes in that commit of all commits when changes were made to that file.
    - search the changes as if they are in vim.

### QA: how to delete a file or directory from git history and from remote repo?

This happens when we do not need a large fies or directory anymore and we want to remove it from .git, in order to reduce the repo size.

**Solution**: using BFG repo-cleaner
    - prepare to use bfg repo-cleaner
        - Make sure Java is installed with `$ java -version`, if not install with `$ sudo apt install default-jre`.
        - Download `.jar` file from `https://rtyley.github.io/bfg-repo-cleaner/` and save it to, for example, `~/Downloads/`.
    - delete files from a repo
        - git clone a bare repo to `~/Downloads/`. **Do not** use your the repo in your computer you are working one.
            ```sh
            git clone --mirror xxxx.git
            ```
        - from `~/Downloads/` run  to delete file `renv.loc`. Do not run the command inside the repo.
            ```sh
            java -jar bfg-1.14.0.jar --delete-files renv.lock xxxx.git
            ```
            - current commit is protected so `renv.lock` will not be deleted from the current commit.Mannually delete it before commit and push to make sure `renv.lock` is deleted from current and past commit.
        - go to the directory of the bare repo and run to update the remote repo
          ```sh
          # delete the file identified by bfg
          git reflog expire --expire=now --all && git gc --prune=now --aggressive
          # push to github
          git push
          ```
    - delete the bare repo and go to local repo and git pull to update local repo.
    - other ways to delete
        ```sh
        java -jar bfg-xxx.jar --delete-folders dir1 xxx.git
        java -jar bfg-xxx.jar --strip-blobs-bigger-than 50M xxx.git
        ```

### QA: how to search commits by string abcd in commit message

- `$ git log --grep=abcd` to show full log message
- `$ git grep abcd` to show the lines with abcd
- `$ git grep -E "abc|xyz"` use regex to search any of multiple keywords
- `$ git grep abc | grep xyz` search for all of multiple keywords

### QA: how to git difftool to show the same file in two commits or branches?

- `$ git difftool HEAD..master file1`

### QA: how to use git show to view a file in other branch or commit?

- `$ git show de304f08:R/pe_dfclass.R` where the path/to/file start from the git project root no matter where the current directory is.


### QA: how to exclude a specific subdirectory by name?

For example, a large `target/` dicectory is created in each Rust project after each run. We have no reason to git track this subdirectory in any Rust project.

To exclude this subdirectory, we can add a line `**/target/` in `.gitignore`. Unlike wildcard `*`, `**` includes `/` in the path. It will exclude all `target/` in the project.

To be more specific, if we only need to exclude those in directory `code/`, we can change it to `**/code/**/target/`.

## Raw notes

### git .git directory is the repo

The other two spaces are working tree/diretory and staging area.

- `git add`: move changed files from working tree to staging area
- `git commit`: move files from staging area to repo
- `git restore`:
  - `git restore --staged file1 file2` move staged files to working tree
  - `git restore file1 file2` move commited file to working tree

- how to move from repo to staging area / working tree? This move does not make sense as staging area is a temporary area for changes to be commited. No reason to move commited files into staging area.

### git commit rules and git log --oneline for long message

- keep commit atomic, that is, a commit for one change.
- a commit should not break a project, that is, a working project should be working after any new commit.
- describe changes in imperative mood, e.g. "make xyz do abc" instead of "makes xyz do abc"
- officially recommend present tense, but past tense is ok. Just follow company rules.
- give good description to a commit
- git log --oneline to see the first line only.
- `.gitignore`
    - `abc/` for directory
    - without `/`, git treat it as file

### git working with branches

- HEAD vs working tree?
- git switch vs git checkout? new replacemenet for checkout regarding branches
- git branch -d abcd: # delete branch abcd from another branch
- git branch -m new-name   # rename current branch

### git restore

- git restore file 1 file2           # undo un-commited changes even if staged
- git restore --staged file1 file2   # undo git add
    - equivalent to git reset .
- git restore --source 3f35tfa file1 # restore file1 to version in commit 3f35tfa
    - equivalent to git checkout 3f35tfa file1

### git merge

- fast forward merge, the easist one. This happens in the situation when we merge branch B in to branch A and B is branched out from A's tip. It is impossible to have any conflicts in the merge.
- three-way merge: when branch A has new commits from where B branched out and B has new commit, the merge cannot be completed in fast forward way. This merge is called 3-say merge as there are 3 ways looking from the branch-out point. Three ways merge may have merge conflicts.

### git comparing changes with git diff

- basic use
    - `git diff`   view changes in unstaged files compared to staged files (or HEAD if staging area is empty.)
    - `git diff HEAD`   view changes since last commit
    - `git diff --staged`  view changes in staged files comparing to the last commit
    - `git diff [HEAD][--staged] file1 file2`  # view changes in a specific files
    - `git diff branch1..branch2`    # compare across two branches
    - `git diff commit1..commit2`    # compare across two commits.

- How to read the message, an exemple
    ```
    gl@xps:~/aaaaa/gitclass (main)$ git diff
    diff --git a/file1 b/file1
    index 13447e2..e5d24f9 100644         # internal number for commit and file, not commit number
    --- a/file1                           # old file where deletion marked by -
    +++ b/file1                           # modified file where addition marked by +
    @@ -1,3 +1,5 @@                       # line number where the change occurs, not important
     This is the first line
    -This is the second line              # deleted from old file in red starting with - signe
    +This is the second line, no!         # new lines added to the modified file, starting with +
    +That is not second line
    +The next is the second line
     This is the third line
    ```

### git stash when having uncommited changes in branch A but need to swtich to branch B

- git stash and git stash pop are sufficient for daily work.

- git stash to temporarily save uncommited changes to stashed area. The stashed changes can be applied to any branches.
- git stash pop to retrieve the stashed changes and clear stashed area
- git stash apply to retrieve stashed the change and keep the stashed area.
- git stash drop to delete stashed changes.
- git stash list to check any stash area.

### git undo changes

- git checkout
    - HEAD~1, HEAD~5, HEAD^, HEAD^^ shows how many commit before HEAD
    - `git switch -` swtich to previous branch, `-` refers to a branch, not a commit that is not the HEAD of a branch.
    - `git checkout HEAD`  delete all changes and restore to files in the last commit

- git restore to undo changes, do the same thing as git checkout
    - `git restore file1`
    - `git restore --source commit# file1`  to replace file1 in working tree with that in commit#.
    - `git restore --staged file1`  remove file from staging area.

- git reset to go back to an early commit
    - `git reset commit#`  delete commits history after the commit# but keep all the changes. All the changes now are in the working tree. This command can be used to combine the most recent commits.
    - `git reset --hard commit#`  delete commits after the commit# and also delete all changes.

- git revert commit# to discard all changes made in that commit
    - `git revert commit#`  better for the collabration by keeping history if other people has the history.
    - may need to resolve conflicts just like resolving merge comflicts.

### git remote, git push, git push -u

- `git remote -v`    list remote origin url
- `git remote rename old new`    origin can be changed to any names but most people use it
- `git remote remove <name>`     a git can have multiple remote so you have option to remove some
- `git push -u origin branchname`    # set the upstream of the current branch to origin branch with -u so we can use `git push` from the branch without typing the full `git push origin branchname`.
- `git push --set-upstream origin branchname`  # same as `git push -u origin branchname`.

### git fetch, git pull

- `git branch -r`  # view the remote tracking branches
- `git checkout origin/branchname`  # you can checkout to a remote tracking branch that is not in local yet and create one locally.
- `git fetch origin master`  # update origin/master tracking branch but does not change my current master HEAD
    - `git status` will compare local branches to the latest remote branches, otherwise the remote tracking branches are those in the last fetch or pull or push
    - `git checkout origin/master` to view the fetched changes
- `git pull`  # update local branch, equals to `git fetch` + `git merge`. It works on current branch.

### git github gist and page

Should use them for main portofolio.

- github pages
    - personal pages
    - project pages
        - create a branch for github pages which contains a index.html file.

### git rebase, use with caution, do not rebase commits shared with others

- why use rebase: 1) as an alternative to merging, 2) as a cleanup tool
- git rebase vs git merge
    - after 3-way git merge, we often get a commit with message simply called "merge branch xxxx". This commit does not have any changes in it. There can be many of them and pretty annoying. And the commit history is complicated when viewed with `git log --graph`.
    - git rebase insetead will create a linear structure and rewrite the commit history.
        - (branchA)$ git rebase branchB    merges branch B into A and places all new commits of branch A at the top even a commit is later than those in branch B. Rebasing means rebase branch A on the tip of branch B.
    - after rebase branch A on B, we can switch to branch B and fast forward merge A to B as A is already rebased on B.
- when **NOT** rebase: do not rebase any commit that has been shared with others. That is, you can only rebase your own local branch that no one else use. Never rebase branches other people use.

### git interactive rebase to clean up history before sharing

- $ git rebase -i HEAD~4  to change history back to HEAD~4
    - reword: change commit message
    - fixup: combine two or more commits
    - drop: delete a commit, including changes

### git tags to mark version and releases

- semetic version rules v2.4.3
    - the first digit 2 for major release that may not backward compatible
    - the second digit 4 for minor release that include new features but is backward compatible
    - the third digit 3 for patches for bug fix and code improvement. No new features.

- work with tag
    - git tag    # view tags
    - git checkout v1.2.5    # checkout to the commit of a tag
    - git diff v1.2.3 v1.2.2  # compare two tags
    - git tag v1.0.0     # create a light weight tag
    - git tag -a v2.3.4   # create annotated tag with more information
        - git show v2.3.4  # to view more information about the tag
    - git tag v2.1.2 6g4r356  # add tag to a commit
    - git tag v2.1.2 u87t673 -f  # move tag to another commit
    - git tag -d v2.1.2   # delete a tag
    - git push --tags  # push including all tags
    - git push v2.3.4  # push a particular tag

### git behind senes

- `.git/config` file: local setting for this repo. This file can be edited to achieve effect like command `git config --local user.name "xxxxx"` and `git config --local user.email "yyy"`.
    ```
    [core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
    [remote "origin"]
        url = https://github.com/GL-Li/totalcensus.git
        fetch = +refs/heads/*:refs/remotes/origin/*
    [branch "master"]
        remote = origin
        merge = refs/heads/master

    [user]
        name = xxxxx
        email = yyyyy

    ```

- `.git/objects` has all the repo data

    - `git cat-file -p 34f5te7` print out information of a commit

        ```
        tree afea54534fb51f30d9e26db3287c47d5f7088050
        parent bc9b93d4cae220ee393acda4d4105506819079ef
        author GL-Li <liguanglai@gmail.com> 1675771322 -0500
        committer GL-Li <liguanglai@gmail.com> 1675771322 -0500

        fix bug in read_decennial(..., year = 2010, summary_level = "block")

        ```

### git reflogs retrive lost work and fix big mistakes

Use `git reflog show` to find the lost commits and then `git checkout xxxx` to the commit.

- reflog stands for reference log. `.git/logs/` has file HEAD and directory logs/.
    - HEAD has the history of changes of HEAD, they are local and expires in 90 days by default. In the exemples below, code like 1675866590 are unix timestamp.
        ```
        4993976964cdbcc4517e45a03d16019401a9b501 a3401891ce0733e2fddc95565be95cf35030dd86 GL-Li <liguanglai@gmail.com> 1675866590 -0500	rebase (fixup): add aaa bbb
        a3401891ce0733e2fddc95565be95cf35030dd86 a3401891ce0733e2fddc95565be95cf35030dd86 GL-Li <liguanglai@gmail.com> 1675866590 -0500	rebase (finish): returning to refs/heads/test
        a3401891ce0733e2fddc95565be95cf35030dd86 a1d2efc68c569e0577ed4cdc428a7e636c62ff28 GL-Li <liguanglai@gmail.com> 1675866900 -0500	checkout: moving from test to main
        a1d2efc68c569e0577ed4cdc428a7e636c62ff28 c2490afd483cdcaab8ae7efef1f6d4cd1d230be1 GL-Li <liguanglai@gmail.com> 1675866904 -0500	merge test: Merge made by the 'ort' strategy.

        ```
    - logs/ contains the HEAD change history of each branch

- git reflog show
    - `git reflog show main` lists HEAD history of main branch from new to old
        ```
        c2490af (HEAD -> main) main@{0}: merge test: Merge made by the 'ort' strategy.
        a1d2efc main@{1}: merge test: Fast-forward
        8c5bd86 main@{2}: commit: add AAA BBB
        e741f5e main@{3}: commit: update 1
        1685275 main@{4}: commit (initial): initial commit

        ```
    - `git reflog show HEAD` shows HEAD history of all branches
       ```
        c2490af (HEAD -> main) HEAD@{0}: merge test: Merge made by the 'ort' strategy.
        a1d2efc HEAD@{1}: checkout: moving from test to main
        a340189 (test) HEAD@{2}: rebase (finish): returning to refs/heads/test
        a340189 (test) HEAD@{3}: rebase (fixup): add aaa bbb
        4993976 HEAD@{4}: rebase (fixup): # This is a combination of 2 commits.
        a1d2efc HEAD@{5}: rebase (start): checkout 8c5bd86

       ```
    - `git reflog show main@{1.week.ago}` shows HEAD history of main branch a week ago.

- recover lost commits in `git reset --hard HEAD~3`
    - `git log` will not see the deleted commits anymore
    - but `git reflog show HEAD` still has the record of the deleted commits
    - `git reset --hard HEAD@{1}` to reset the branch to a commit.

- recover lost commit in `git rebase -i HEAD~4`
    - git rebase changes history and renames commits
    - git reflog show still keeps those history and can be used to recover lost commit
    - git reset HEAD@{1} to go back to the last HEAD
    - git reset --hard HEAD@{2} to reset back to a lost commit or
    - get checkout 3rdwq2t to the lost commit as a detached HEAD so we can view the changes in the commit.

### global git config file `~/.gitconfig`

- `vim ~/.gitconfig` to edit configuration for all repositories.
    ```
    [credential]
        helper = cache
    [init]
        defaultBranch = main
    [core]
        editor = vim
    [user]
        email = liguanglai@gmail.com
        name = GL-Li
    [diff]
        tool = vimdiff

    ```

### git alias

Add a section to `~/.gitconfig` file. good reference:
    - https://www.durdn.com/blog/2012/11/22/must-have-git-aliases-advanced-examples/
    - https://github.com/GitAlias/gitalias
    - just use at `git ll` where ll stands for the long code below.
        ```
        [alias]
           ll = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --numsta
        ```
