# Lab Assignment: Resolving Merge Conflicts

## Overview

In this lab, you will practice resolving merge conflicts. When you merge one branch into the other, Git will be forced to ask you to resolve the conflict manually.

## Get start

> _reference: https://graphite.dev/guides/git-clone-all-branches_

Fork this repo to your own GitHub account, and remember to un-check the `Copy the main branch only` option.

Ensure your repo has all three branches `main`, `branchA`, and `branchB`.

Clone your repo to local using `git clone`

Check your local branch using `git branch`, you might find there's only one `main` branch available locally.

Check the remote branches using `git branch -r`, you will find 
```
  origin/HEAD -> origin/main
  origin/branchA
  origin/branchB
  origin/main
```

We also want to track other remote branches.

We can checkout branches locally by
```
git checkout -b <branch-name> origin/<branch-name>
```
or we can automate this process and checkout all remote branches using a loop in command line:
```
for branch in `git branch -r | grep -v '\->'`; do
    git branch --track "${branch#origin/}" "$branch"
done
```

After setting up tracking for each branch, make sure your local copies are up to date:
```
git fetch --all
```

Now we are ready to start our lab



## Lab Steps

**Merge branchB into branchA (three-way merge with conflict)**
   - Switch to **branchA** using `git checkout branchA`
   - run command ```git diff branchA branchB main.txt```, you will find the `branchA` and `branchB`'s `main.txt` are different:
```
diff --git a/main.txt b/main.txt
index 6d58650..d096ad8 100644
--- a/main.txt
+++ b/main.txt
@@ -1 +1 @@
-Hello from Branch A
+Hello from Branch B
```
   - First, make sure you are in `branchA`.
   - Merge **branchB** into **branchA** using `git merge branchB`. This merge will trigger a merge conflict since `branchA` and `branchB` diverged from the initial commit, and **the two branches changed the same part of the same file differently**
   - Open `main.txt` and look for conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
   - Manually edit the file to resolve the conflict, then stage and commit your changes.
   - You will receive a message like `[branchA 9c33a47] Merge branch 'branchB' into branchA`

**Merge branchA into main (three-way merge)**
   - Switch to **main**.
   - run command ```git diff main branchA main.txt```, you will find the `main` and `branchA`'s `main.txt` are different.
   - Merge **branchA** into **main**. Note `branchA` and `main` diverged from the initial commit, but there's **no conflict** even if the the file `main.txt` is different. Why? 

## Key Commands

- **Initialize Repository:**
  ```bash
  git init
  ```

- **Create and Switch Branches:**
  ```bash
  git checkout -b <branch-name>
  git checkout <branch-name>
  ```

- **Add and Commit Changes:**
  ```bash
  git add <file>
  git commit -m "Your commit message"
  ```
  
- **Merge Branches:**
  ```bash
  git merge <branch-name>
  ```
  
## Resolving Conflicts

1. Edit the file to remove conflict markers.
2. Stage the resolved file: `git merge <branch-name>`
3. Commit the merge resolution: `git commit -m "Resolved merge conflict"`
