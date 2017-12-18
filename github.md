GitHub Operations
=================

# Creating A New Github Repository

### Github Remote Repository

  1. Goto Github account.
  2. Click + at top and select **New Repository**
  3. Name it, select public or private, and press **Create Repository**
  
### Local Repository

  1. Create or move to a new directory under local git folder, e.g. c:\data\git\xyz
  2. Initialize with ***git init***
  3. Edit files as necessary.
  4. Add files to staging ***git add .***
  5. Commit files ***git commit -m "Commit message"***
  6. Add a shortcut for remote repository created above  
     ***git remote add origin  https://github.com/CanyonCasa/xyz.git***
  7. Push files to remote  
     ***git push origin master***
  8. Go to Github and verify changes.

# Updating a Local Repository

  All these actions assume you are located in the local repo directory of choice.
  
  ### To sync with remote repo (i.e. copy latest files to local repo)
  
  git pull
  
  ### To see status of local repository (i.e. pending changes)
  
  git status
  
  ### To stage files for upload
  
  git add .
  or
  git add <filename>
  
  ### To commit

  git commit -m "commit comment"

  ### To upload to remote repo

  git push
  
# Removing A Github Repository Fork

  1. Go to Github account.
  2. Copy name of repository to be removed to the clipboard.
  3. Click on the repository to be removed.
  4. The click on Settings at the top.
  5. Scroll to the bottom under Danger Zone.
  6. Click on **Delete this Repository**.
  7. Paste the name into the dialog.
  8. Click OK.

# Other useful Git commands

    git config --global core.autocrlf false|true|input
    use "git rm --cached <file>..." to unstage
    git remote -v
    git reset HEAD <file>
    
    
    