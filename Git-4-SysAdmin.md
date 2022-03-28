# Configuring Git
Global configuration for git is stored in /etc/gitconfig
Project configuration for git is stored in /path/to/project/.git/config 

We can make changes to the config on the system level with
	 
	 $ git config --system

Changes to the user level are made with
	
	$ git config --global

Changes to the local project are made with
	
	$ git config (--local)

We will first setup user configuration, and if you are pushing to GitHub, your username and email should match what you use on that site.

	$ git config --global user.name “AErck”
	$ git config --global user.email “aerck42@gmail.com”
	$ git config --global core.editor “vim”
	$ git config --global color.ui true

# Manage Linux System confit files with etckeeper 
The software etckeeper uses git to track system configurations and tracks file metadata.
	
	$ sudo yum install -y epel-release
	$ sudo yum install -y etckeeper

Let’s edit the conf file for etckeeper
	
	$ sudo vim /etc/etckeeper/etckeeper.conf

By default, etckeeper commits once a day and before package installations automatically. You will need to enable this feature.
	
	$ sudo systemctl enable etckeeper.timer

Once done, start up the etc repo from the /etc directory.
	
	$ sudo etckeeper init
	
It is recommended to .gitignore vim swap files by adding an entry for "*.swp" files, typically in default config already.
	
	$ sudo vim .gitignore
	
Make the first commit, this is required to get etckeeper running automatically
	
	$ sudo etckeeper commit “Initial commit of /etc”
	
Pushing to a remote repo, make sure it is private or sanitized if repo is public. First we need to add the remote repo

Login as root, then from the /etc/.git directory, add your remote Github repository:

	# git remote add origin https://github.com/user/repo.git
	
Next configure etckeeper to use provided web hook
Edit the PUSH_REMOTE option in /etc/etckeeper/etckeeper.conf, with the name of the remote repository you want etckeeper to push to. For example:

	PUSH_REMOTE="origin"

# Creating a local repository
We will start with a new directory for the project and initialize the repository from that directory
	
	$ mkdir ~/GitProjectOne
	$ cd GitProjectOne
	$ git init
	
Once the repository is started, we get a new directory called .git/ and this is where all of the changes in the repository are stored.

# Committing and adding files
Here we create some test files, modify them, and commit them.
	
	$ vim ourfirstfile.txt
	
Check what git has tracked
	
	$ git status
	
Git has noticed a new file that is not being tracked, which we will fix
	
	$ git add
	
Now commit the file changes to the disk
	
	$ git commit -m “initial commit”
	
The git status command should now be happy
Now make a few more changes to our file. Git status should now return a warning that it has noticed a change.
This time we will use a different commit option and skip the add step because no new files need to be tracked.
	
	$ git commit -a
	
This will open up into the default editor for you to type in your commit message.
The best practice for commit messages is to use the present tense, the top line should be a short summary, and the rest should be an optional verbose message about the commit.

# Analyzing Git commit history 
First we can look at the git log
	
	$ git log
	
For only the most recent commit, modify the command
	
	$ git log -n 1
	
For all commits since a specific date modify this way
	
	$ git log --since=2019-05-14
	
Changing “since” to “until” gives you all commits up to that date
For all commits from a specific author
	
	$ git log --author=Adam
	
For all commits with a piece of text in the commit message
	
	$ git log --grep=“text”
	
All of these options can be combined into one search of the log.
Additionally, multiple “grep”s can be used in one search, just add --all-match to filter by commits with matches to all “grep”s specified.
Add the --invert-grep option to get commits without the grepped text.

# Ignoring files
Sometimes it is useful to not track all of the files in the files in your git project directory. To have git ignore some files, create the .gitignore file.
	
	$ vim .gitignore
	
We can specify which files to ignore in this file. Typically we don’t want to track/commit .exe files or .tar files. 
You can have additional .gitignore files in project subdirectories that overwrite the parent .gitignore files in case you need a more advanced configuration. To do this add a line with the file or file types preceded with a “!”.

# Rolling back changes
To see the differences between the current file on the disk and the version in the repository use the diff command.
	
	$ git diff
	
We can overwrite local uncommitted changes by getting the repository version of our file
	
	$ git checkout -- ourfirstfile.txt

If the changed file has been “staged” or “added” with git add already, you will need to reset the files first.
	
	$ git reset HEAD ourfirstfile.txt
	
To “undo” a changed file that has been committed to the repository, you must make a new commit that changes the target file to match an older version of the file. We cannot undo any of the commits, but in this way we make the file like it was.
First you will need to grab the commit number or ID of the version you want to revert to.
Then you will use the revert command.
	
	$ git revert <commit-ID>
	
You will then be taken to your editor

# Branches in Git
Using branches is useful for testing new features on deployed software without changing the original software. If the test features are good, then the branches can be merged into the main branch. This reduces the need to rollback the main branch while it is in production.

You can check to see which branch you are in with
	
	$ git branch
	
Create a new branch to switch to
	
	$ git branch testconfig
	
You can now see the new branch with
	
	$ git branch
	
The asterisk is still showing that we are using the master branch
Use the checkout command to switch between branches
	
	$ git checkout testconfig
	
Since these branches are still the same, you won’t notice much different.
To view the differences between two branches, utilize the diff command as follows
	
	$ git diff <branch-1>..<branch-2>
	
To rename a branch you would move it just like a normal Linux file like this
	
	$ git branch --move <old-branch-name> <new-branch-name>
	
To delete a branch
	
	$ git branch --delete <branch-name>
	
To Fast Forward merge the branches you would use this command from the original branch
	
	$ git merge <branch-to-merge-in>
	
Conflicts that Git cannot auto-merge must be manually fixed.

# Using a remote repo with Github
If you created a local repo before the remote repo you want to store this repo on Github, you will need to link the remote repo to your local repo
	
	$ git remote add origin <github-repo-url>
	
Make sure to push local commits to the remote repo
	
	$ git push -u origin master
	
To pull a remote repo onto a different machine use
	
	$ git clone <github-repo-url>
	
Enter your GitHub credentials when prompted

# Hosting Your Own Git Repo
This is a great option for sensitive data, or system administration for security conscience people.
Additionally, LANs are faster and Local Storage is cheap.
From the “Git Server” we will create a user called “git”
	
	$ sudo useradd git
	
Give the git user a password
	
	$ sudo passwd git
	
It is highly recommended to set the git user’s shell to “git-shell” to prevent interactive logins from this user. You should set this in the .bashrc for this user.
Make a directory for the repository and move to it
	
	$ sudo mkdir -p /srv/git/GitProjectTwo.git
	$ cd /srv/git/GitProjectTwo.git
	
Now initialize a “bare” Git Repository
	
	$ sudo git init --bare
	
Now we set permissions from /srv/
	
	$ sudo crown -R git.git git
	
Now note the IP address for this host
	
	$ ifconfig
	
Next move to the client computer where the local repository will live and create a directory for it
	
	$ mkdir -p ~/RemoteRepos/GitProjectTwo
	
Move into the directory and initialize the repo
	
	$ cd ~/RemoteRepos/GitProjectTwo
	$ git init
	
Create some files and stage them for commit, and commit them
	
	$ git add .
	$ git commit -m “Initial Commit”
	
Now add the remote server as the origin for this repo

	$ git remote add origin git@<is-address>:/srv/git/GitProjectTwo.git
	
Verify with
	
	$ git remote -v
	
There will be a remote host listed for pushing and fetching
Now we will push our files to the origin
	
	$ git push origin master
	
Accept the fingerprint, and enter the git user’s password
This repo can now be cloned from any other host with network connection to this host.

# Using SSH keys for Git repo authentication!
From the git client you will want to create the ssh keys and share them to the git server
	
	$ ssh-keygen -t rsa -b 4096 -C “aerck42@gmail.com”
	
You can use the defaults for file name and passwords here
Now share the key
	
	$ ssh-copy-id git@<ip-address>
	
We can also use this ssh key pair with GitHub
	
	$ cat ~/.ssh/id_rsa.pub
	
Copy the printed key to clipboard
Log in to GitHub, open settings, and go to “SSH and GPG keys”
Add a new key, name it, and past in the ssh key from the git client. Make sure it is saved.

Now from the git client, set the repo to use SSH remotes instead of HTTPS
	
	$ git remote -v
	
You will see that git is using https here. We need to change that with the following command

	$ git remote set-url origin <ssh-github-link>
	
Verify with
	
	$ git remote -v
	
Now we can push our files to the origin
	
	$ git push origin master

# Strategies for managing system files
System Administration Caveats
	◦ Can’t manage binary files
	◦ Doesn’t store file metadata
		‣ Ownership
		‣ Permissions
		‣ Extended attributes
	◦ Not a replacement for full backups
	◦ Be carful not to manage file with sensitive data, like .ssh or /home
Use a local repository for Onyx, but make sure it is included in the system wide backup





