# IU HPC Setup

Last Updated Jan 2022

## Setting up your [IU High Performance Computing (HPC) accounts](https://kb.iu.edu/d/aczn)

1.  Log into [One.IU](one.iu.edu)
2.  Search for "Accounts", enter the "View Accounts" app
	![Account Screen](/images/one_iu_view_accounts.png)
3.  Review your computing accounts; as of 2022 for bioinformatic purposes, we recommended using at least Carbonate (shared with Research Desktop), Scholarly Data Archive (SDA), and Slate.
4.  Click "Create Computing Account" to access further resources.  Fill out the agreement form and submit.
    - For storage resources, answer the fields
5.  You should receive a notification when your account(s) is created; you can then login and access IU HPC resources.

## Accessing Carbonate

Refer to the IU Knowledge Base (IUKB) for many of your computing needs, including [Cabonate](https://kb.iu.edu/d/aopq).

There are a couple ways to [access Carbonate](https://kb.iu.edu/d/aolp#access). You can shell into a login node by opening the terminal and using `ssh` to log in: `ssh your_username@carbonate.uits.iu.edu`. Enter your password and Duo authenicate as needed. You'll then be able to navigate the high performance file system, submit jobs, etc. from the terminal.
- Note that doing any heavy lifting on the login nodes is frowned upon - most programs will be killed after 20m of CPU time.  Some simple, low-overhead commands like `wget`, `tar`, and `gunzip` can run longer. [Submit a job](https://kb.iu.edu/d/awrz) or start an interactive session for anything more.

### Research Desktop on Carbonate

Your other option, if you prefer, is the [Research Desktop](https://kb.iu.edu/d/apum)(RED). This provides a graphical desktop for users who don't prefer the command line, or who need an easy way to run graphical programs like RStudio, Jupyter Notebooks, and so forth.  You also get the advantage of running on a dedicated (but shared) RED node; this is akin to automatically running an interactive session.
- You should still submit a job for any intensive analysis, as your memory usage is limited to \~75GB before the program is automatically killed (and there are other users on the node using resources).
- There's a "Carbonate Job Manager" helper program on the desktop to assist you in managing jobs.


## Submitting Jobs

Once you are ready, [submit a job](https://kb.iu.edu/d/awrz) using the Slurm Workload Manager. We'll cover that in a little more detail in the next chapter.
