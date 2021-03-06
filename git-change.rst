============
 git-change
============

-----------------------------------------------
Git command to create and manage Gerrit changes
-----------------------------------------------

.. |date| date::
.. include:: version.rst

:Author: Jacob Hesch <jacob@nextdoor.com>
:Date: |date|
:Manual section: 1


SYNOPSIS
========

| `git change` [create] [<create-options>]
| `git change` update [<update-options>]
| `git change` rebase
| `git change` list
| `git change` submit
| `git change` gc
| `git change` clean
| `git change` print [<print-options>]


DESCRIPTION
===========

Use git-change to create and manage changes for the **Gerrit Code
Review** [1] tool. The default behavior is to create a new
change. There are subcommands to manage the change at later stages,
including uploading a new patch set, rebasing, and garbage-collecting
the temporary change branches this command creates.


USAGE
=====

create [-r|--reviewers=] [--cc=] [-b|--bug=] [-m|--message=] [--topic=] [--skip=] [--fetch] [--switch] [--chain] [--use-head-commit] [--merge-commit] [--ignore-owners]

    Create a new change and upload to Gerrit. Creating a change is the
    default operation, so omitting the subcommand causes `git-change`
    to behave as if `create` had been specified.

    The files that make up the change must be staged for commit. Those
    staged changes will be committed in a new temporary branch meant
    to exist locally and exclusively for this change.

    The `create` subcommand performs the following operations:

      | 1. Notes the current tracking branch (the "target" branch)
      | 2. Creates a temporary, local branch (the "change" branch)
      | 3. Commits the staged changes to the change branch
      | 4. Creates a Gerrit code review destined for the target branch

    If there are commits in the target branch not yet merged into its
    remote branch prior to step 3 above, a warning will be issued to
    explain that continuing would result in multiple new commits being
    added to the temporary branch and pushed to Gerrit, resulting in
    multiple code reviews, all of which share the reviewers, bug ID,
    etc.

    Change branches are named change-<ID>, where ID is the change ID
    generated by the commit-msg hook that ships with Gerrit. That hook
    must be enabled. Note that if the Change-Id header is not injected
    into the commit message for some reason, or if step 4 above fails,
    the temporary change branch will be named tmp-change-<TS> where TS
    is a timestamp of as a floating point number expressed in seconds
    since the epoch. In this case the change branch must be manually
    deleted and the change creation must be retried.

update [-r|--reviewers=] [--cc=] [-b|--bug=] [--skip=] [--ignore-owners]

    Update the existing Gerrit change with new changes. Staged changes
    will be automatically committed by amending the HEAD commit. The
    current branch must be a temporary change branch.

rebase

    Rebase the target and temporary change branches. The current
    branch must be a temporary change branch.

    First the target branch (the branch from which the temporary
    change branch was created) will be rebased, then the temporary
    change branch will be rebased. This subcommand can be used to pull
    upstream changes down to both branches to resolve a failed Gerrit
    submission due to a path conflict.

    If there are conflicts with either rebase operation, the process
    terminates and it is up to the user to resolve the conflicts at
    that point and retry.


list

    List all temporary change branches and display a menu to check one
    of them out.

submit

    Submit the code review associated with the current change branch
    to Gerrit.

gc

    Remove temporary change branches which are fully merged.

clean

    Remove all temporary change branches (unmerged branches included).

print [-r|--reviewers=] [--cc=] [-b|--bug=]

    Print the command to push a change to Gerrit. This can be useful
    if manaually creating a Gerrit code review is desired.


OPTIONS
=======

-r <addresses>, --reviewers=<addresses>
            Comma-separated list of reviewers.

--ignore-owners
            Boolean indictating whether to ignore the owners listed in
            OWNERS files when uploading a change branch. This is false
            by default. However, if the `include-owners` git config is
            not set, owners are always ignored, reguardless of this
            flag.

--cc=<addresses>
            Comma-separated list of addresses to copy on change notification
            mails.

-b <bug-id>, --bug=<bug-id>
            Bug ID to include in the commit message header. This
            option causes `git-change` to set the BUG_ID environment
            variable to the given ID before invoking `git-commit` so
            that a Git hook can add it as a commit message header.

-m <msg>, --message=<msg>
            Use the given message as the commit message.

--topic=<topic>
            Tag the change with the given topic name.

--skip=<checks>
            Comma-separated list of pre-commit checks to skip. Option
            values: tests, whitespace, linelength, pep8, pyflakes,
            jslint or all. This option assumes that a pre-commit hook
            runs the checks, and causes `git-change` to set the SKIP
            environment variable to the given list of checks before
            invoking `git-commit` so that the hook can skip them.

--fetch     Run `git-fetch` so that remote branch is in sync with
            the central repository.

--switch    Switch to the temporary change branch after creating it.

--chain     Chain with the previous Gerrit change. Use when this
            change depends on the previous one. Current branch must be
            a temporary change branch. Implies --switch.

--use-head-commit
            Use the HEAD commit as the change to push rather than
            committing staged changes.

--merge-commit
            Create a change for a merge commit. Implies
            `--use-head-commit`.  This flag assumes the current branch
            is a tracking branch and that the HEAD commit is an
            unreviewed merge commit for which a review is being
            created. A change branch will be created and `git-commit
            --amend` invoked in order to have the commit-msg hook add
            a change ID header. The usual check for unmerged commits
            is skipped, so be sure all of the commits being merged
            have change ID headers to avoid having Gerrit create a
            review for each one. Finally, note that the HEAD (merge)
            commit in the original tracking branch is removed after
            the change branch is created.

--remote=<remote>
            Name of the remote repository to fetch from and push to.
            Defaults to the `git-change.remote` Git config option if
            it is set, otherwise 'origin'.

--gerrit-ssh-host=<host>
            Name of the Gerrit server hosting the Git repository.
            Defaults to the `git-change.gerrit-host` Git config
            option if it is set. Required unless the config
            option is set.

CONFIGURATION
=============

color=<when>
            Color-code change branches based on their status. Status is one
            of 'Normal', 'Merged', or 'Current'. Defaults to 'auto'.
            Set 'false' to disable.

change-id-url=<host>
            When specified, the change id during the listing is preceded with
            a custom url. Example: cid-url=http://github/user/repo/

gerrit-ssh-host=<host>
            Name of the Gerrit server hosting the Git repository.
            Defaults to the `git-change.gerrit-host` Git config
            option if it is set. Required unless the config
            option is set.

remote=<remote>
            Name of the remote repository to fetch from and push to.
            Defaults to the `git-change.remote` Git config option if
            it is set, otherwise 'origin'.

include-owners=<boolean>
            Whether or not the add OWNERS file support for this
            repository. If this is not set, OWNERS files will be
            ignored.


SEE ALSO
========

git(1), git-commit(1), git-merge(1), git-rebase(1), git-fetch(1), git-config(1), githooks(1)


NOTES
=====

1. Gerrit Code Review: http://code.google.com/p/gerrit/
