---
layout: post
title: "Neater Pull Requests (PR) with git"
menutitle: "git split"
category: programming
author: kunaltyagi
tags: git rebase shell programming dev
comments: true
---

Turning my commit history from spaghetti to symphony.

Or feel free to skip to the [alias](#the-alias) and the [summary](#summary), and pretend you didn't see the chaos. I won't be offended (just mildly devastated).

Typical development advice goes that I should work on byte-sized chunks of work, be it a small feature or a well-defined bug post a long and harrowing debugging session.
This allows my commits to be *neat*, with each commit telling the story of the pull request, one step at a time.
Each commit becomes a self-contained snapshot, a glorious breadcrumb trail for my co-workers to follow and relive the adventures I undertook.

But alas, the same can not be said of a frantic debugging session, when I'm up to my shoulders in the elephant grass, sweating through the heat, swatting at the flies, trying to survive long enough to find and catch the elusive bug before I catch some terminal git disease.

Sure, I could have prepared better... maybe even a little bit, but what's done is done.
I've come too far to turn back and do it the "clean way."

And now?
Now it's all a mess.
The bug is fixed, yes, but the commit history? A dumpster fire.
A tangled mess rivaled only by a toddler left alone with a bowl of spaghetti while the babysitter went to fix a merge conflict.

# The Alias

Thankfully, I stumbled upon a gem buried deep in the [depths of internet](https://hisham.hm/2019/02/12/splitting-a-git-commit-into-one-commit-per-file/), and turned it into a git alias.
And let’s be honest here, git aliases are better than gems.
This one has saved me countless minutes of brain-numbing grunt work.
It's a little wild, a litle weird, some may call it overly comlex. But it gets the job done.

```ini
[alias]
    split = "!f() { message=\"$(git log --pretty=format:'%s' -n1)\"; if [ `git status --porcelain -uno | wc -l` = 0 ]; then git reset --soft HEAD^; fi; git status --porcelain -uno | while read status file; do if [ \"$status\" = 'M' ]; then git add \"$file\"; git commit -n \"$file\" -m \"$file: $message\"; elif [ \"$status\" = 'A' ]; then git add \"$file\"; git commit -n \"$file\" -m \"added $file: $message\"; elif [ \"$status\" = 'D' ]; then git rm \"$file\"; git commit -n \"$file\" -m \"removed $file: $message\"; else echo \"unknown status $file\"; fi; done; }; f"
```

# Using The Alias

## The Setup

Let’s say the Git history looks something like this:

```text
h7i8j9k 2025-05-02 | Implement authentication module [Alice]
l0m1n2o 2025-05-03 | Fix login bug [Bob]
p3q4r5s 2025-05-03 | Update dependencies [Charlie]
t6u7v8w 2025-05-03 | Refactor user service [Delphi]
x9y0z1a 2025-05-03 | Lint everything [Eve]
```

Hmmm... Maybe I *should* update dependencies as the first step.
That will allow me to use new stuff right away in the implementation commit, no awkward refactoring required later on.

**Note to self**: install the pre-commit hook as the first thing before starting development on a new machine.
But for now, the aim is to split the lint commit so that each commit is linted as if the pre-commit hook was there all along.

## The Action

I run `git rebase -i -x "git split" main` and I tweak the resultant buffer daily, nightly, ever-so-rightly:

```text
pick p3q4r5s Update dependencies
pick h7i8j9k Implement authentication module
pick l0m1n2o Fix login bug
pick t6u7v8w Refactor user service
exec git split
pick x9y0z1a Lint everything
exec git split
```

## The Resolution

A-bada-bing-a-bada-boom!
We now have a freshly baked history.
All commits up to the fix step are untouched, but the two `exec`-tagged commits have exploded into lovely little file-specific ones:

```text
added api/user/posts.py: Refactor user service
api/user/route.py: Refactor user service
api/user/auth.py: Lint everything
api/user/route.py: Lint everything
```

Do the messages look weird? A little, yeah.
Some file names have snuck their way into the original commit messages.
Never mind that — I can always reword the messages later.

Now I am free to shuffle the commits around, rnu another interactive `git rebase` (this time without the `exec`), and finally get the clean history I so desperately want to show the world.

The End!!

## Summary

Assuming the [alias](#the-alias) is present in either `~/.gitconfig` or the repo's `.git/config`,

1. If needed, create a new branch using `git switch -c <branch-name>` to preserve your current state.
2. Run `git rebase -i -x "git split"`.
3. Tweak the list, deleting any `exec` lines you don't want.
4. Run `git rebase -i` (again).
5. Rearrange and fixup (using `fixup` or `f` instead of `pick`) commits as needed.

Don't forget to push! And remember the name of the new branch (see step 1, optional but recommended)

## The Reveal (aka the boring part)

How does the trick work?

A few small bits on `git rebase`, a clever alias, and a whole lotta gumption.

The `-x` (aka `--exec`) on `git rebase` inserts the `exec` lines after each commit, saving me several seconds and tens of keystrokes.
`exec`, true to its name, runs a command after each commit; in this case, our fancy alias stuff.

And the alias? It does all of the explody-commit bit.

Officially The End!!

# Breaking Down the Alias

Still here?

Of cource you are. We have much to unravel in this trench-coat full of raccoons, all three of them.

Let's break it down and turn it inside out into a proper bash script:

1. Copy everything inside the quotes (after the initial `!`) into a new file. Let's ignore the [exclamation mark](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias) for now
2. Replace all `\"` with `"`. Those are just an escape sequence for nesting quoting.
3. Run a bash formatter on the file, IFF your eyes need a break

Or you know... just look at [the original](https://hisham.hm/2019/02/12/splitting-a-git-commit-into-one-commit-per-file/). You should end up with something like this:

```bash
f() {
    message="$(git log --pretty=format:'%s' -n1)";
    if [ `git status --porcelain -uno | wc -l` = 0 ];
    then
        git reset --soft HEAD^;
    fi;
    git status --porcelain -uno | while read status file;
    do
        if [ "$status" = 'M' ];
        then
            git add "$file";
            git commit -n "$file" -m "$file: $message";
        elif [ "$status" = 'A' ];
        then
            git add "$file";
            git commit -n "$file" -m "added $file: $message";
        elif [ "$status" = 'D' ];
        then
            git rm "$file";
            git commit -n "$file" -m "removed $file: $message";
        else
            echo "unknown status $file";
        fi;
    done;
};
f
```

That's a function followed by an immediate call to it. Quite close to what the cool kids use: [IIFE](https://en.wikipedia.org/wiki/Immediately_invoked_function_expression).
No arguments here.

You could trace what's happening here with a little understanding of git, bash and some Bing-fu (don't @ me).

* First, we grab the current [commit message](https://git-scm.com/docs/git-log#Documentation/git-log.txt---prettyltformatgt) (given the `%s` and the `-n1`)
* Then we do a conditional soft reset (whatever that does)
* Loop over some `MAD` files (which we get from some porcelain stuff)
* Use an `if-elif-else` ladder to create new commits based on their status: `M`odified, `A`dded or `D`eleted.

Once the curtain's lifted, the magic's all just ... plumbing. Well, *except* for:

* [`--porcelain`](https://git-scm.com/docs/git-status#Documentation/git-status.txt---porcelain)
* [`-uno`](https://git-scm.com/docs/git-status#Documentation/git-status.txt--ultmodegt) (What *is* Mattel doing here!?)
* [`--soft reset`](https://git-scm.com/docs/git-reset#Documentation/git-reset.txt-Undoacommitandredo)
* [`HEAD^`](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection).\
* [`-n`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt-code-ncode)
* The noticeable lack of [plumbing commands](https://git-scm.com/book/en/v2/Appendix-C%3A-Git-Commands-Plumbing-Commands)

And some lingering questions for my sanity.

But let's stop here. There's only so much you can dig before your `HEAD` explodes.
