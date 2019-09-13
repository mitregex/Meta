---
title: Source Control Management Concepts - intermediate
layout: post
date: '2019-09-10 23:24:05 +0530'
categories: blogs
author: Regex Events
---

In the last post, I covered why Google drive isn't a very attractive version control system. I mentioned three points supporting that argument. The way a version control system like git solves those issues was dependent on the rest of the post and could not be explained at that point in time. However, since the basic concepts of git are clear by now, here are the reasons :-

- Since commits contain only the changes, and are atomic units which are stacked (as opposed to every folder upload being considered a fresh folder), developer A can upload a commit C1 which contains the changes in the file F1, and it will be added to the commit history, making the changes represented by C1 a part of the final state of the codebase. Since a commit contains information only about changes in code and not the entire repository, if another developer B commits another commit C2, it will get applied on top of the commit C1. I.e. C2 will become the child commit of C1 and the changes contained within C2 will get applied on the repository. The order of these commits could have been either ways as long as they don't act on the same file -- if they do act on the same file there is a possibility of conflicts arising. Conflicts will be explained in detail in a later part of this post.
- Due to the above, developers are now able to work parallelly and not sequencially, which increases the workflow productivity by a huge margin (clearly). It will also not result in overwrites because only changes are tracked, as explained.
- A commit's size is negligible when compared to the size of the repository. If one isn't able to work with the size of the `.git` folder because of network issues, they probably won't be able to work in any case.

The first point in the above list is rather important. The ability to atomically apply git commits on top of an existing commit history makes it a very powerful software for version control. It makes merging possible.

## Merging

This is perhaps the most important concept of git. A scenario for its usage could be: Developer A and developer B want to collaborate with each other on a common project. They want to implement different features. They can't work on the same commit history, because it would get very confusing for each of the developers if random commits keep appearing in their working directory which they themselves didn't implement and are in no way related to the feature they're developing. 

<img src="/git 2.1.png" style="display: block; margin: auto;" />

While looking through `git log` developer A wouldn't know where the third commit from the bottom has come from and what it does, and consequently if it interferes with the changes they're making in the codebase with their commits. It is also important to note that because they're each using the main branch for their testing during implementation, the project might have a lot of bugs during development due to which many complications might arise for both the developers, and they will be able to work on only one feature at a time. Most developers are working on 2-3 features and more than 3 bugs at the same time, so it would become quite cumbersome.

This is where branches are useful, and also where I introduce feature branches.
Feature Branches are a special category of branches (not really special because they're the most common reason for creation of branches). If a developer wants to implement a feature they wouldn't directly start committing to the branch they have currently checked out. They will create a `branch` which specifically tracks *their* progress with the feature they're trying to implement. No one else will commit to their branch because everyone will have their own feature branches to work with. The next immediate question would be, if everyone is working on their own branch, how do we combine those individual features in order to finalise what the product which is to be shipped to users?

In git, it is possible to merge two different branches such that their functioning is combined. Git essentially stacks the commits of one of the branch on top of the commit history of the other branch. In a usual case, the branch whose commit history is being added to is the master branch (for understanding what master is, please refer to the previous post). Merge is usually done by [`git merge`](https://git-scm.com/docs/git-merge), and occasionally through the code hosting provider's (eg. Github) UI. Just a note: merging is a concept; there are two ways to implement it. 1 - using the UI/git command, 2 - using `git rebase` to physically move commits from one branch to another.

`git merge` essentially takes all the commits in branch 1 which are not present in branch 2's commit history and combines the changes in all of them to form the code changes section of a newly created 'mega commit' of sorts, and pastes that commit on top of the topmost commit in branch 2.

## Conflicts

A conflict in git may occur due to two reasons.

1. Code conflict. This is slightly tricky to understand. Consider developer A pushing a commit C1 which consists of the following code changes segment.
<img src="/git 2.2.png" style="display: block; margin: auto;" />
Here, A is removing line 1 and adding line 1.1 to one of the files in the repository (which has not been mentioned for the sake of brevity). Now consider developer B pushing a commit C2 who's code changes are as follows.
<img src="/git 2.3.png" style="display: block; margin: auto;" />
Here, B is removing the same line 1 and adding line 2. This commit is applied to C1 which was sitting on top of the commit history prior to this. At this point, git doesn't know what to do, because it sees that it has to remove the line 1 from C2 but doesn't see that line in the file because it has been removed by C1. Git throws a conflict at this point, asking us for advice on what it should do to resolve it.

2. Commit conflict. Consider a situation where there is a remote repository, and two developer's local repositories.
<img src="/git 2.4.png" style="display: block; margin: auto;" />
Here we see a normal git repository being initialised on the remote and developers A and B cloning them onto their own local machines. Now, consider A and B developing for it in parallel.
<img src="/git 2.5.png" style="display: block; margin: auto;" />
A pushes the code for their update.
<img src="/git 2.6.png" style="display: block; margin: auto;" />
Now, B tries to push their code. However, this time, it is rejected. I would encourage you to stop here and look at the problem and think about what could have caused an issue based on both the previous post and this one before reading the next paragraphs.

A commit (C2) tracks what its parent commit was. Since the remote repository would have `init commit` --> C1 as it's state, trying to push C2 which has its parent set to `init commit` will try to apply C2 on C1, which is not the parent of C2 as listed in its data (called a patch). This will cause a conflict, which could be broadly classified as a commit conflict.

To solve this problem we would have to execute `git push --force` or a more advanced command, `git pull --rebase`. The force option on `git push` forcefully changes the entire history of the remote branch with the commit history of our local branch. So, after force pushing developer B's branch, you would result in something like this.

<img src="/git 2.7.png" style="display: block; margin: auto;" />

If you would like to learn more about git, I believe these blogs have covered enough concepts to build a strong foundation for git, and reading the documentation of git would be no problem to you now.

Extras: [Git's own Book](https://git-scm.com/book/en/v2).

Author's notes: There are many more advanced concepts to cover, but I believe this will suffice for now considering the point put forth in the previous paragraph.