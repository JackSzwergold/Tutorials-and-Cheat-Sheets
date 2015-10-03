# Cheat Sheet - Git - Rewinding to a Specific Commit

By Jack Szwergold, September 15, 2015

#### Rewinding to a specific commit hash in the repository.

Check the log for the commit hash.

    git log

Find a commit to roll back.

	commit b331419ad4cc4397a3138d3fc1166be014debc4f
	Author: Firstname Lastname <email_address@example.com>
	Date:   Sat May 16 21:31:20 2015 -0400
	
	    Setting a short name for the application; useful for deployments.

Reset the repo to that commit.

    git reset --hard b331419ad4cc4397a3138d3fc1166be014debc4f

Now push that commit back to GitHub.

    git push -f

***

*Cheat Sheet - Git - Rewinding to a Specific Commit (c) by Firstname Lastname*

*This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC-BY-NC-SA-4.0).*