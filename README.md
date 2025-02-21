# Lab Assignment: Resolving Merge Conflicts

## Overview

In this lab, you will practice resolving merge conflicts. When you merge one branch into the other, Git will be forced to ask you to resolve the conflict manually.

## Lab Steps

**Merge branchB into branchA (three way merge)**
   - Switch to **branchA**.
   - Merge **branchB** into **branchA**. This will trigger a merge conflict.
   - Open `main.txt` and look for conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
   - Manually edit the file to resolve the conflict, then stage and commit your changes.

**Merge branchA into main (fast forward merge)**
   - Switch to **main**.
   - Merge **branchA** into **main**. At this stage, `branchA` is ahead of `main`. Therefore, no conflicts are triggered and git will move the `head` pointer for main ahead.

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