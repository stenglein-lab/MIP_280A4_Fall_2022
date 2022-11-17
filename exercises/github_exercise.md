## GitHub exercise

MIP 280A4 
---

### In this exercise, we will learn some git (github) basics

Today, you will create a github account (if you don't already have one) and a repository (possibly your first one?).  This repository is where you will document the work you do for your final project.   


#### Open a browser, navigate to github.com, and create an account

You will need to use an email for your github account.  You may want to use an email that you expect to have for a long time (in other words, you may want to *not* use your CSU email if you plan to graduate from and leave CSU sometime.) 

Later, you can customize your github profile if you want.  For instance by putting in a profile picture, providing a location, etc.

Note that there is an option to keep your email private if you prefer.

> If you already have an account, log into to it.

#### Create a personal access token

This token will act like a password, and will enable you to interact with github from the command line.

Follow these steps to create a github access token:

- Login to github
- In the upper-right corner of any page, click your profile photo, then click Settings.
- In the left sidebar, click <> Developer settings (all the way at the bottom).
- In the left sidebar, under Personal access tokens, click Tokens (classic).
- Select Generate new token, then click Generate new token (classic).
- Give your token a descriptive name. 
- Set the expiration date to 90 days from now, or further into the future if you want.
- Set the scope of the token to repo (check the box that says repo) 
- Click Generate token. 
- The token will appear and will be some text that begins with `gph_` followed by numbers and letters.
- **Copy the token and email it to yourself or store it in some other location on your computer.** Treat this token like a password (keep it secure and private).  Note that if you lose the token you can just generate a new one, but that's extra work. 

#### Create a new repository

Let's create a repository that you will use to document your final project.

- Login to github and click on the Repositories tab.
- Click the green New button to create a new repository
- Give it a name like `2022_MIP_280A4_final_project` or something 
- For Description, enter something like "Final project for MIP 280A4, Microbial Sequence Analysis"
- Leave it as a public repository. (Or make it private but then you will have to add me as a collaborator for me to see it and help you or grade it.
- Check on `Add a README file`
- Choose a License if you want to put some copyright on your work.  People often use the MIT license for open source software.  You can also add a license later.
- Click Create Repository

#### Clone the repository

Now you've created a repo, but it's very bare bones, with a file named README.md and possible a file named LICENSE, with copyright license text.  We want to add stuff (files) to it.  We can do this from the command line.  To do this, we'll need to download (clone) a copy of the repository.  Then you'll make changes on thoth01 (local changes) that you can commit and push to github.  

[This page](https://uidaholib.github.io/get-git/3workflow.html) has a nice description of a typical git workflow.

The basic steps for working with a repository are:

- `git clone`: download a copy of a repository
- make local changes: add files, edit files, add directories, etc.
- `git add`: "stage" files to be committed.
- `git commit`: commit the changes you've made.  You'll need to supply a message that describes what you did.
- `git push`: sync the changes you made locally to the remote repository.  This uploads any new files or edited files to the remote repository.

Other relevant commands includ:
- `git status`: gives you information about your repository: what's new, what's changed, etc.
- `git diff`: tells you what's different between your local repo (what's on thoth01) and the remote repo (what's on github)
- `git pull`: sync your local repo copy with the remote repo by pulling any changes that have been pushed to the remote repo.  This is necessary when you are working collaboratively with people on a project/repository.

Let's clone your new repository so you can work with it on thoth01.  First, you need to find the URL for this repository.  This is found in the `<> Code` dropdown on the repository's main page.  This will be some URL like: `https://github.com/stenglein-lab/2022_MIP_280A4_final_project.git`  (with your username instead of stenglein-lab).  Copy this URL.  

Login to thoth01 and clone the repository using that URL and `git clone`.
```
# use the correct URL for your repository
git clone https://github.com/YOUR_GITHUB_USERNAME/2022_MIP_280A4_final_project.git
```

This will download a copy of the repository into a new directory that has the same name as the repository. 

#### Make local changes and push them to the repo

You will be documenting the work you do for the final project in a [markdown format](https://www.markdownguide.org/basic-syntax/) file.  ([Here's a page](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) on the markdown format that github supports.)

To edit this file, let's use nano.  
```
# Make sure you are in your repository's directory! 
cd 2022_MIP_280A4_final_project  # or whatever you named the repo

# create a new file
nano report.md
```

In nano, edit the file to begin your final project report by adding the following text
```
# MIP 280A4 Final Project 2022

This report documents the final project I did for MIP 280A4, Microbial Sequence Analysis, in the fall of 2022.

It is written in [Markdown format](https://www.markdownguide.org/basic-syntax/).  

## Step 1: Thanksgiving Break

I'm going to eat some delicous food

## Step 2: Describe my project

My project is about x y z and my dataset is from a b c.

```

You'll be filling this out as you complete the final project in the last weeks of class.

BTW, if you want to become a command line power user, you can learn to edit files using the [vim](https://www.vim.org) text editor.  Check out [this video](https://www.youtube.com/watch?v=y4SzAr33st0) from the OMGenomics channel (Maria Nattestad) for a tutorial.


#### Commit the local changes

OK, now that you've made local changes to your files you'll need to commit and push them up to github.  

First, let's check that git knows you made changes:
```
# see what the situation is
git status
```

You should see some text like this, that tells you that you need to add your new file.

```
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	report.md
```

So let's use `git add` to add it to the repo.  In git terminology, the file is now 'staged' and ready to be commited.

```
git add report.md
```

Run `git status` again to see how this changed things:

```
git status
```

Before we can commit the file, you need to tell git who you are (what your email and name are).  Run these commands:

```
# use the email address associated with your github account.  
# Note quotes around email address and name.
git config --global user.email "YOUR_EMAIL_ADDRESS@gmail.com"
git config --global user.name "Your Name"
```

You only need to do that once (per computer/server).

Now we are ready to commit.  Commit you changes and supply a commit message using the -m option:

```
# this will commit all the files that are staged
git commit -m "adding report.md to repo"
```

You should see some output that the commit occurred successfully, e.g.:
```
[main 5059448] adding report.md to repo
 1 file changed, 1 insertion(+)
 create mode 100644 report.md
```

Now we will push the changes to github: upload them so the repo on github.com is in sync with the changes you've made and committed locally.

```
git push origin main
```

Your username will be the email you used when creating your github account.
Your password will be the **access token** you created earlier.  Copy and paste this in.

Now, go to your repository online and confirm that your new file is there.

Great!  Now you understand how to interact with a github repository from the command line.  During your final project, your overall workflow will be:

1. Add a new file or directory to the repository directory.  Or, edit an existing file.
2. Use `git add` to stage the new or edited file.
3. Use `git commit` to commit one or more changed files.
4. Use `git push origin master` to sync the commit to github.
5. Back to step 1.



