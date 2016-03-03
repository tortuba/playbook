# Git Protocol

A guide for programming

## General Guidelines

* Avoid including files in source control that are specific to your development machine or process.
* Delete local and remote feature branches after merging.
* Perform work in a feature branch.
* Rebase frequently to incorporate upstream changes.
* Use a pull request for code reviews.
* Develop in a way compatible with Zero Downtime Deployment

## Branches

**Long-lived:**
* master: production
* (default) develop: staging

**Short-lived:**
* fix/<bug-name>:
  * forked from master
  * merged back into master + develop
* feat/<feature-name>: feature branch
  * forked from develop
  * merged into develop

All branches should be pushed regularly to be:
  * saved
  * visible by anyone
  * tested by our Continuous Integration service

## Working on a short-lived branch

Typically only one person works on one branch.

If we need more people to work on a feature or [if the feature must be spread over multiple deployments][1], we **break it down to the smallest possible chunk** that one person can ship.

[1] Typically to enforce Zero Downtime deployments

### Create a local branch

For production bugs:

    git checkout master
    git pull
    git checkout -b fix/<branch-name>

For features:

    git checkout develop
    git pull
    git checkout -b feat/<branch-name>

### Work (loop)

Work, rinse, repeat:

    git status # Check that you're on the correct branch
    git add --all
    git commit -m "Fix #42: Present-tense summary under 50 characters"

See [Closing issues via commit messages](https://help.github.com/articles/closing-issues-via-commit-messages/).

Rebase frequently to incorporate upstream changes and resolve conflicts.

    git fetch origin
    git rebase origin/<master(fix) or develop(feat)>

Share your work (saves, makes visible, tests with CI)

    git push (origin <branch-name>)

### Merge

#### If you're allowed to bypass the Pull-Request process

For production bugs:

    git checkout master
    git merge <branch-name> --no-ff
    git checkout develop
    git merge <branch-name> --no-ff
    git push origin -d <branch-name>
    git branch -d <branch-name>

For features:

    git checkout develop
    git merge <branch-name> --no-ff
    git push origin :<branch-name>
    git branch -d <branch-name>

#### Otherwise: Submit a pull request

See [Github documentation](https://help.github.com/articles/using-pull-requests/).

  1. Navigate to the repository
  2. Switch to your branch
  3. Click New pull request
  4. Add a message and create the pull request

**Tip:** ``base branch`` is where you think changes should be applied, the ``head branch`` is what you would like to be applied

## Reviewing a pull request

See [Github documentation](https://help.github.com/articles/merging-a-pull-request/).

It's recommended to check out the pull request locally before merging.

**Tip:** Pull requests are merged using the --no-ff option.

### Check out pull requests locally

See [Github documentation](https://help.github.com/articles/checking-out-pull-requests-locally/).

Used to resolve a merge conflict or to test and verify the changes on your local computer before merging on GitHub.

---

**References**

  * [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
  * [Our software development workflow](http://blog.codeship.com/software-development-workflow-new-feature/): they don't use a develop branch, but then again their staging only checks 500 errors
  * [Github Code Review](http://blog.codeship.com/github-code-review/)
  * [Thoughtbot Git Protocol](https://github.com/thoughtbot/guides/tree/master/protocol/git)
