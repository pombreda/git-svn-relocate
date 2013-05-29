git-svn-relocate
================

A tool for doing the equivalent of `svn switch --relocate` on a git-svn repository.
MUCH faster than `git filter-branch` on large repositories. Operates in O(commits) and does no
filesystem operations (stats, reads, etc.).

DISCLAIMER
----------
This will generate an entirely new history for your git-svn repository (i.e. 100% of your commit hashes will change), and
will change every head reference to point at the new history.

I would _strongly_ suggest doing this on a clone of your repo, or at least copying the hashes of every head on your repo.
If you operate on your only copy of your git repo and lose all your data, it's completely your fault. If you run this
tool and it turns out that it corrupted all your commits but you didn't find out right away, it's completely your fault.
**No warranties of any kind are implied**.

You have been warned.

That said! If you find bugs, I'm happy to take pull requests :)

Usage
-----

```bash
./git_svn_relocate.py --old_url <url> --new_url <url> --old_uuid <uuid> --new_uuid <uuid>
```

You can find the old url + UUID at the bottom of every commit which git-svn has introduced into your repo's history.

Why?
----
Rebuilding large (>100k revisions) git-svn repositories from the new SVN server can take weeks. Filtering over your
git repo with git filter-branch can take days.

How?
----
This simple python script loops over every commit in your repo in topological order and manipulates the raw commit
objects. This avoids a lot of undesirable extra processes and filesystem operations present in filter-branch (no need
to shell out on every commit, no need to verify tree objects. There's not even a need to fully parse the commit objects).

In particular, it replaces:
  * The `git-svn-id: <url>@rev <uuid>` line
  * UUID of the old repo on the committer/author lines.
  * The parents of the commit objects to point to their now-rewritten parent objects.
