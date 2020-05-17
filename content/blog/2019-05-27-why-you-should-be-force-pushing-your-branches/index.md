+++
title = "Why you should be force pushing your branches"
description = "Force pushing has been getting a bad wrap ever since it was invented, but it's actually the best way to push!"
aliases = ["/blog/2019-05-27-why-you-should-be-force-pushing-your-branches/"]
[taxonomies]
tags = ["git","beginner"]
+++

A lot of people hate force pushing.
I hear the same argument over and over again, that needing to force push to a branch means you've done something wrong.

I detest this!

In fact, I'd argue that needing to force push to a branch means you've done everything right!

## Fixup your mistakes

For a lot of people, Git is like the save button in Word.
You'll press it every time you make a change, you'll undo a change then save again, maybe reapply the same change and then save again.

In the end, you'll end up with a massive log that achieves basically nothing!

Looking through the log of this website I found five commits where I was trying to update the avatar!

```text
Author: Bennett
  Update avatar

Author: Bennett
  Fix avatar

Author: Bennett
  Fix avatar for good this time

Author: Bennett
  Nah actually this time

Author: Bennett
  FINALLY!
```

Now this is all well in good in a personal project.
But when you start working on a codebase with others, it gets a little bit annoying.

Imagine that one of your coworkers introduced a bug and used commit messages like that.
When they go through with a [`git bisect`](https://git-scm.com/docs/git-bisect) to find out what commit has an error, they'll be greated with the amazing message _"Nah actually this time"_!

What does this mean? Who knows! What if you don't even work at the company anymore, how will they know what the author was thinking?

Enter `--fixup` and `--amend`.

Who would have known that all this time, there's been features build straight into Git that allow you to edit commits that you've made previously.

Here's the same workflow, but instead of writing a new message every time you use `git commit --amend`.

```text
Author: Bennett
  Update avatar
```

Now when someone is looking to see why the avatar is broken, they'll be able to find this commit and see.

`--fixup` works much the same, except instead of automatically amending the previous commit, you can provide a commit id for it to be amended to. This allows complex work-flows and updates without ever polluting the log.
Just run `git rebase -i --autosquash` before you push your branch.
There's a great [stackoverflow answer](https://stackoverflow.com/questions/3103589/how-can-i-easily-fixup-a-past-commit) on how to use fixup if you're keen on learning more.

Okay, now `git push`, what? Hey? What's going on!

Well don't distress, it's simply because the commits on your local branch are different to the one on remote
(I can feel a force push coming on...) But just wait! It's important to ask yourself a few questions before you push.

1. Are there any changes on the remote that I can't afford to lose?
2. Are there any other people working this branch?
3. Is this an important branch like master / production?

If you answered yes to the first one, don't force push just yet. Instead, you need to `rebase` your current work onto those changes.
This can easily be done with `git pull --rebase`. Now, with the latest changes, you're good to go!

If you answered yes to the second, hold on just a second. If there's other people working on this branch, it's probably a good idea to let them know you're intending to force push. This way, they'll know not to force push and overwrite your changes!

Finally, if you answered yes to the third question. Force pushing probably isn't a good idea - at all.

## Working with branches

At this point I should probably mention that you should be very careful force pushing to any branches that you share with other people.
Up until now, everything I've spoken about only applies to your individual feature branches.

This is super important, because force pushing (as the name, and notoriety suggest), is an incredibly destructive action.

There's a saying among dentists, "only brush the teeth you want to keep", and I think something similar rings true for software development - you should only protect the branches you can't afford to lose!

You can [read up on branch protection here](https://help.github.com/en/articles/about-protected-branches) if you've never heard of it.
If you have, make keep those important (like master, development, production, etc..) protected so that you won't accidentally force push by mistake!

## Get rid of "Merge" commits

Merge commits are the bane of my existence! And the `--fixup` / `--amend` work-flow does a lot to remove them.

Unfortunately, Github can still add them by mistake.

Don't worry though! It's actually super easy!
In the settings of your Github repository, simply enable the "Allow squash merging" option and you're good to go!
If you can't find the option, you can read [a bit more about squash merging here](https://help.github.com/en/articles/configuring-commit-squashing-for-pull-requests);

---

In the end there's no _real_ right way to use Git - it's a tool with many users with many opinions.

What matters most is that you're using some kind of method to ensure that your not wasting time recreating things you forgot to save.

However, the next time you're about to commit another lazy message, save yourself the effort `--amend` or `--fixup` instead.
