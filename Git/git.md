# Why Git?
- Distributed
    - Whole repo is on everyones machine
    - You dont need to be connected to the internet to access older versions
- Branching
- Everyone else does it: Github
- Competition: Mercurial, SVN, Bazaar

# Intalling Git
## OSX
- Use homebrew `brew install git`
- Make sure that /usr/local/bin is before /usr/bin
- You can change the order by adding `export PATH="/usr/local/bin:$PATH"`

## Windows
- git-scm.com and then click download
- Run the installer
    - Pick the last option "Run Git and included Unix tools..."
        - Pick the first option "Checkout windows, commit unix"
            - It is important that everyone in your team commits unix style
        - Or if you are sure that everyone will be using a windows, you can use checkout and commit as-is but i do not recommend this at all
- I would highly recommmed that you use the git shell   

## Test install
`git --version` should return the right version that you installed.

# Configure git
- You can configure settings at
    - the `system` level = All users on the computer
    - the `global` level = The current user
    - the `local` level = Per repository (**This is the default**)
        - this config file is in `.git/config`  
- **Set your name, email and color**
    - `git config --global user.name "Nikunj Handa"`
    - `git config --global user.email "nikunj.sg@gmail.com"`
    - `git config --global color.ui true`
- Test if it was set:
    - `git config user.name`
    - `git config user.email`
    - `git config color.ui`

# Initialize a repository
- `mkdir project`, `cd project`, `git init`
    - Creates a .git folder, this has all the files and all the versions

# Basics
## Staging and Statuses
- Git works like a factory
    - If you are working on a file, its like a factory working on a product
    - Once the product is built you put it in an area ready to ship it (this is the staging area where you can put files by calling git add)
    -  Once you have all the final products you can send them out (in git you can commit it)
    -  At anytime you can use git push to send it to a central server
- `git status` gives the status of all the files in the repo
- `git add <filename>` copies the current version on the file in the working area to the staging area.
- Illustrate all this on the terminal using a simple text file

## Commit
- `git commit` will pop up a text file with some comments in it
-  Those comments will not be pushed to the server, you can just go to a new line without a `#` and start typing your message
-  When you save and quit, the commit continues automatically
-  If you make a mistake you can add the changes to the staging area and call `git commit --amend`
        - This will require you to re-enter the commit message
    - If you do not wish to re enter the message, call `git commit --amend -C HEAD`
        - `HEAD` is the latest commit on the current branch
        - This will copy the message and author info from the HEAD commit or any other commit whose hash you enter

## Reset
- If you want to go back to a commit, you use `git reset <commit>`
    - The default mode is `git reset --mixed <commit>` == `git reset <commit>`
        - Changes are preserved, but not put in the staging area
    - Soft mode `git reset --soft <commit>`
        - Changes are preserved AND put in the staging area
    - Hard mode `git reset --hard <commit>`
        - Changes are discarded
    
- When you do a reset to an older version, the commits after the the one you reset to can be lost, because git can garbage collect it at any time
    - However, if you want to revert in a couple of hours or so, you should be fine
    - Find the dangling commits by calling: `git reflog` or `git fsck --lost-found`
    - Then call `git reset hash` to go back to a newer commit

## Checkout (Reseting files)
- Checkout is usually used in branches but it can also be used to reset individual files
- `git reset` only works for full commits
- Roll back to version in staging area: `git checkout -- filename`
- Roll back to a commit: `get checkout HEAD filename`
    - HEAD can be HEAD~2 or a hash like ea234cd 

### Shortcuts
- `git commit -a` stages the files that are **already being tracked**and commits them, no new files will be added
- `git commit -am "Message here"` to add the message in the cmd too

## Ignoring
- Commit a `.gitignore` file
- (Github Gitignore)[https://github.com/github/gitignore]
- Folders without files are automatically ignored
- When you write something in .gitignore it matches that pattern in the file path.
- So, it you ignore `tmp`, it will ignore **both** `/tmp` and `/hello/tmp`
    - if you want to ignore only `/tmp` type `/tmp`
    - if you want to ignore only `/hello/tmp` type `/hello/tmp`
- Anti pattern can be made using `!`
    - This will ignore all `.txt` files in `tmp` except `master.txt`:
    
    ```
    tmp/*.txt
    !tmp/master.txt
    ```

## Diff
- Supposing you are working on some file and then get distracted and come back to the same file after a day or two
- You will type `git status` and see that there is a modification
- Use `git diff` to see what you changed and if you still need it
    - `git diff filename` compares the current file (working dir) to the version in the staging area OR, if file not in staging, to the last commit
    - `git diff --staged filename` or `git diff --cached filename` compares the file in staging to the latest commit
        - `staged` and `cached` are the same thing
    -  `git diff HEAD filename` compares working directory to the latest commit

- To get the difference between two branches you call `git diff master..otherBranch`

## Log
- `git log` gives a history of all the commits
- `git log --stat` gives statistics with the commit
- `git log --oneline` gives one line info, oneline is useful to get the hashes if you wanna diff or something
- `git log --graph` gives a graph which is useful if you have branches
- `git log --pretty="%h, %cn, %cr"` lets you format the log your own way
    - for a list of variables go (http://git-scm.com/docs/git-log)[http://git-scm.com/docs/git-log]
- or use `gitk` for a ui

## Branching
- You want to experiment with something or want to fix a bug but dont want to break the stuff on the master branch (a branch that git creates automatically)
- In this case, you create a branch and then merge it when you are done and sure that everything works  

### Creating and Basics
- Create branch using `git branch branchName` and switch to it using `git checkout branchName`
    - Shortcut for these two commands is `git checkout -b branchName`
- Creating a branch creates a new working directory, when you checkout, you are actually switching directories  

- `git log --all` to view a log across all branches
    - `git log --online --graph --all --decorate` is a nice way of seeing the changes  

- If you make changes, you need to commit them to a certain branch.
    - If you change a file and commit it, the working directory will be clean when you move around to other branches after commmiting
    - If you change a file (without commiting), the changes are persisted across changes in branches

### Merging
- To merge a branch with another, first checkout to that branch and then call `git merge branchYouWantToMerge`
    - Git merge actually merges the changes and then, if there are no conflicts, commits the changes to the branch you are on (merging to)
    - If this has conflicts, the commit will not be automatic. You can just resolve them and commit manually
- Once you have merged a branch, you can just delete it by calling `git branch -d branch1`
    - This is a good practice and doesnt actually delete the content of the branch... It just deletes the reference 
    - If you delete a branch that is not merged, you must use `-D` instead of `-d`
    - This will delete all the changes and remenants of that branch **if it is not merged**
        - if the branch is already merged, `-D` and `-d` have the same effect  

### Rebasing
- Rebasing is a cleaner way of merging branches. This is how it works:
    - Supposing you are merging bugfix to the master branch
    1. Rewind the master branch to the point at which bugfix branch was created
    2. Apply all the bugfix branch commits to the bugfix branch
    3. Apply all the commits in the master branch that were rewinded

- So, this removes all evidence of the branch ever existing and keeps the repository cleaner

## Remotes
- To add a new remote: `git remote add origin git@github...`
    - To send stuff over this remote `git push origin master`
    - To get the latest stuff from this server `git pull origin master`
    - `git remote -v` to list the details of a remote

## Conflict Management
```
<<<<<<< HEAD
int b = 1;
=========
int b = 2;
>>>>>>> da342fc
```

- HEAD is the changes in the latest commit of the current branch on your machine
- The other one is the change you made.

## Tagging
- Used to mark releases, etc
- `git tag -a v1.0 -m "Message"` creates a tag on the latest commit
- `git tag -a v1.0 93ed2a -m "Message"` creates a tag on the commit starting with 93ed2a
    - Tag names must be **unique**

- To view all tags, `git tag`
- To view details on the tags, `git show v1.0`

- `git describe` shows your current commit relative to the last hash
    - Sample response: `v1.0-2-g8ee0f4a` means yoiu are 2 commits after v1.0 and the current has is 8ee0f4a (g is not included, it means git)

# Best Practices
- One commit per issue/feature
    - Use the staging area, stashing to make this possible 
    - Break up your commits into granular pieces and add the using git add
    - Then commit all the files that belong to that feature with a meaningful message

# Some Theory
## Working Directories
- The directory you are making changes in is called the working directoy
- This directory has the latest uncompressed changes
- A `bare clone` is a git repo without a working directory
    - This means you only push to this directory and make no changes on through it.
    - Github is a bare clone or repositories

## What is HEAD?
- `HEAD` is the latest commit in the current branch
- `HEAD~` is the parent of the latest commit of the current branch
- `HEAD~2` is the grand-parent...
- And so on...

# GITHUB
- Create a free account
- Create a repository from scratch and another one that is an existing repository on your computer
- Talk about README.md
- Go through settings (see admin below)
- Go through the file view

## Repositories that do not belong to you
### Fork
- Creates a copy of the repository on your account
- You can make changes to it as though it was your own repository
- And then you can send a pull request with these changes

### Watch
- Just adds changes and updates in this repo to your feed

## Your repositories
### Admin
- Service hooks do something after a commit 
    - Pivotal Tracker integration is here!
    - Loads of third party apps (even twitter!)
- Deploy keys are keys that bypass ssh keys and can used used by automated scripts to send things to your github repository
    - This means that anyone using that machine can update git!

### Pull Requests
- If there is no conflict, you can just use the github GUI to accept the changes
- To merge a pull request form the branch, click on command line in github and follow the instructions
- The process is usually like so:

```bash
git checkout -b mestTest2-master master

# Isn't it possible for this to change? After the pull request the forker might add another update to the repo that might be different from the pull request commit. This might be a good solution: http://www.somethingorothersoft.com/2012/05/22/pulling-github-pull-requests-with-git/
git pull https://github.com/mestTest2/ThreeJavaFiles.git master

git checkout master

git merge mestTest2-master

# Resolve conflicts and commit if needed

git push origin master
```

## Advanced/Interesting Stuff
### Github Pages
- Host static webpages on the github
    1. You get one subdomain for your own account `username.github.io`
        - Just create a repo called `username.github.io` in your account
        - Add static content (html pages, js, css, images)
            - The main page is `index.html`
            - You can add a custom 404 page at `404.html`
        -You can use (Jekyll)[http://jekyllrb.com/] and it uses this by default but if you want to disable it, add a file called `.nojekyll` to your repo

    2. And one subdomain for each project `username.github.io/repo/`
        - This uses an orphan branch in the same repository
            - An orphan has no parent and is like a completely seperate repo in the same repo.
        - For a github page under a project you must create a orphan branch gh-pages. So: `git checkout --orphan gh-pages`
            - When you call this command, git automatically stages the files in your master branch to this new branch
            - You must **unstage and delete** all these files by calling `git rm --cached -r .` and then `rm -r ./*` and finally `rm filename` for all the files
                - Thi∫∫s just empties the working directory for THIS branch. All your files are still in the working directory of the master
                - If you `git checkout master` the files will all be there
        - Now just add the static files like before
        - push to the gh-pages branch on github like so: `git push origin gh-pages`

### Interative Add


### Stashing


### GUI


## FAQ
### Link your computers SSH key to github
- Create an ssh key
    - On windows, do this on the git bash or on cygwin

- `ssh-keygen -t rsa -C 'youemail@gmail.com'`
    - Add a passphrase, this is important for security
    - This will create `id_rsa` a private key and `id_rsa.pub` a public key
        - Private is a secret, do not share this key
        - Copy content of id_rsa.pub and paste it in github ssh keys settings
- Test by typing `ssh -T git@github.com`