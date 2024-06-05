# 1 `$()`

The `$(eval ...)` construct in a shell script is used to evaluate a command and then execute it. Here’s a breakdown of what each component does:

1. **`$( ... )`**: This is command substitution. It executes the command inside the parentheses and replaces it with the output of the command. For example, `$(echo "Hello")` will be replaced by `Hello`.

2. **`eval ...`**: The `eval` command takes a string as an argument and executes it as if it were a command. For example, `eval echo "Hello"` will execute `echo "Hello"`.

When combined as `$(eval ...)`, it first evaluates the inner command and then executes the resulting string as a command. 

For example, consider the following:

```bash
SSH_KEY_PATH="~/.ssh/id_rsa"
echo $(eval echo $SSH_KEY_PATH)
```

Here's what happens step-by-step:

1. `eval echo $SSH_KEY_PATH` is evaluated. Since `SSH_KEY_PATH` is set to `~/.ssh/id_rsa`, it becomes `eval echo ~/.ssh/id_rsa`.
2. `eval echo ~/.ssh/id_rsa` is executed, which outputs `/home/username/.ssh/id_rsa` (assuming `~` expands to `/home/username`).

This effectively resolves and expands any variables or tilde (`~`) notation in the path, ensuring that the command receives the correct, fully expanded file path.

In the context of your script, `$(eval echo $SSH_KEY_PATH)` ensures that the path provided by `SSH_KEY_PATH` is fully expanded before it's used in commands like `chmod` or `ssh-add`.

# 2 `work/abikesa_jbb_https.sh`

```bash
# Create a template Jupyter Book; to be modified later
jb create jb
jb build jb
git clone https://github.com/abikesa/create

# User-defined inputs for abi/abikesa_jbb.sh; substantive edits on 08/14/2023:
read -p "Enter your GitHub username: " GITHUB_USERNAME
read -p "Enter your GitHub repository name: " REPO_NAME
read -p "Enter your email address: " EMAIL_ADDRESS
read -p "Enter your root directory (e.g., ~/Dropbox/1f.ἡἔρις,κ/1.ontology): " ROOT_DIR
read -p "Enter the name of the subdirectory to be built within the root directory: " SUBDIR_NAME
read -p "Enter your commit statement: " COMMIT_THIS

# Build the book with Jupyter Book
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"

cd "$(eval echo $ROOT_DIR)"

# rm -rf $SUBDIR_NAME/_build; cuts runtimes by 90%+;
rm -rf $SUBDIR_NAME/_build
jb build $SUBDIR_NAME
rm -rf $REPO_NAME

if [ -d "$REPO_NAME" ]; then
  echo "Directory $REPO_NAME already exists. Choose another directory or delete the existing one."
  exit 1
fi

# Cloning
git clone "https://github.com/$GITHUB_USERNAME/$REPO_NAME.git"
if [ ! -d "$REPO_NAME" ]; then
  echo "Failed to clone the repository. Check your GitHub username, repository name, and permissions."
  exit 1
fi

# Copy files from subdirectory to the current repository directory; restored $REPO_NAME!!!
cp -r $SUBDIR_NAME/* $REPO_NAME
cd $REPO_NAME

git add ./*
git commit -m "$COMMIT_THIS"
git remote -v

git remote set-url origin "https://github.com/$GITHUB_USERNAME/$REPO_NAME.git"
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"

# Checkout the main branch
git checkout main
if [ $? -ne 0 ]; then
  echo "Failed to checkout the main branch. Make sure it exists in the repository."
  exit 1
fi

# Pushing changes
# git config pull.rebase true
# git pull
git push -u origin main
if [ $? -ne 0 ]; then
  echo "Failed to push to the repository. Check your credentials and GitHub permissions."
  exit 1
fi

ghp-import -n -p -f _build/html
cd ..
rm -rf $REPO_NAME
echo "Jupyter Book content updated and pushed to $GITHUB_USERNAME/$REPO_NAME repository!"

```

# 3 `work/abikesa_jbb_ssh.sh`

```bash
# User-defined inputs for abi/abikesa_jbb.sh; substantive edits on 08/14/2023:
read -p "Enter your GitHub username: " GITHUB_USERNAME
read -p "Enter your GitHub repository name: " REPO_NAME
read -p "Enter your email address: " EMAIL_ADDRESS
read -p "Enter your root directory (e.g., ~/Dropbox/1f.ἡἔρις,κ/1.ontology): " ROOT_DIR
read -p "Enter the name of the subdirectory to be built within the root directory: " SUBDIR_NAME
read -p "Enter your commit statement " COMMIT_THIS
read -p "Enter your SSH key path (e.g., ~/.ssh/id_nh_projectbetaprojectbeta): " SSH_KEY_PATH

# Ensure ssh-agent is running; https://github.com/jhurepos/projectbeta 
eval "$(ssh-agent -s)"

# Remove all identities from the SSH agent
ssh-add -D

# Add your SSH key to the agent
`chmod 600 "$(eval echo $SSH_KEY_PATH)"`
ssh-add "$(eval echo $SSH_KEY_PATH)"

# Build the book with Jupyter Book
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"

cd "$(eval echo $ROOT_DIR)"

# rm -rf $SUBDIR_NAME/_build; cuts runtimes by 90%+;
rm -rf $SUBDIR_NAME/_build
jb build $SUBDIR_NAME
rm -rf $REPO_NAME

if [ -d "$REPO_NAME" ]; then
  echo "Directory $REPO_NAME already exists. Choose another directory or delete the existing one."
  exit 1
fi

# Cloning
git clone "git@github.com:$GITHUB_USERNAME/$REPO_NAME"
if [ ! -d "$REPO_NAME" ]; then
  echo "Failed to clone the repository. Check your GitHub username, repository name, and permissions."
  exit 1
fi

# Copy files from subdirectory to the current repository directory; restored $REPO_NAME!!!
cp -r $SUBDIR_NAME/* $REPO_NAME
cd $REPO_NAME

git add ./*
git commit -m "$COMMIT_THIS"
git remote -v
ssh-add -D 
# Remove all identities from the SSH agent
chmod 600 "$(eval echo $SSH_KEY_PATH)"
# ls -l ~/.ssh/id_stata0elemental
#chmod 600 "$(eval echo ~/.ssh/id_stata0elemental"

git remote set-url origin "git@github.com:$GITHUB_USERNAME/$REPO_NAME"
ssh-add "$(eval echo $SSH_KEY_PATH)"
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"


# Checkout the main branch
git checkout main
if [ $? -ne 0 ]; then
  echo "Failed to checkout the main branch. Make sure it exists in the repository."
  exit 1
fi

# Pushing changes
# git config pull.rebase true
# git pull
git push -u origin main
if [ $? -ne 0 ]; then
  echo "Failed to push to the repository. Check your SSH key path and GitHub permissions."
  exit 1
fi

ghp-import -n -p -f _build/html
cd ..
rm -rf $REPO_NAME
echo "Jupyter Book content updated and pushed to $GITHUB_USERNAME/$REPO_NAME repository!"

```

# 4 `work/abikesa_jbc.sh`

```bash
#!/bin/bash

# Check if required commands are available
for cmd in git ssh-keygen jb ghp-import; do
  if ! command -v $cmd &> /dev/null; then
    echo "Error: $cmd is not installed."
    exit 1
  fi
done

# Parameters required by the script
read -p "Enter your GitHub username (e.g., abikesa): " GITHUB_USERNAME
read -p "Enter your GitHub repository name (e.g., nabongo): " REPO_NAME
read -p "Enter your email address (e.g., abikesa.sh@gmail.com): " EMAIL_ADDRESS
read -p "Enter your root directory (e.g., ~/Dropbox/1f.ἡἔρις,κ/1.ontology): " ROOT_DIR
read -p "Enter the name of the subdirectory to be created within the root directory (e.g., charles): " SUBDIR_NAME
read -p "Enter the name of the populate_be.ipynb file in ROOT_DIR (e.g., populate_be.ipynb): " POPULATE_BE
read -p "Enter your git commit message (e.g., automated abikesa_fromfolks.sh script): " GIT_COMMIT_MESSAGE
read -p "Enter the number of acts (e.g., 4): " NUMBER_OF_ACTS
read -p "Enter the number of files per act (e.g., 1): " NUMBER_OF_FILES_PER_ACT
read -p "Enter the number of sub-files per file (e.g., 1): " NUMBER_OF_SUB_FILES_PER_FILE
read -p "Enter the number of notebooks (e.g., 7): " NUMBER_OF_NOTEBOOKS

# Set up directories and paths
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"
cd $(eval echo $ROOT_DIR)
rm -rf $REPO_NAME
mkdir -p $SUBDIR_NAME
cp $POPULATE_BE $SUBDIR_NAME/intro.ipynb
cd $SUBDIR_NAME

# Check if SSH keys already exist, and if not, generate a new one
SSH_KEY_PATH="$HOME/.ssh/id_${SUBDIR_NAME}${REPO_NAME}"
rm -rf $SSH_KEY_PATH*
if [ ! -f "$SSH_KEY_PATH" ]; then
  ssh-keygen -t ed25519 -C "$EMAIL_ADDRESS" -f $SSH_KEY_PATH
fi

cat ${SSH_KEY_PATH}.pub
echo "Please manually add the above SSH public key to your GitHub account's SSH keys."
read -p "Once you have added the SSH key to your GitHub account, press Enter to continue..."
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain $SSH_KEY_PATH

# Create _toc.yml file
toc_file="_toc.yml"
echo "format: jb-book" > $toc_file
echo "root: intro.ipynb" >> $toc_file # Make sure this file exists
echo "title: Play" >> $toc_file
echo "parts:" >> $toc_file

# Iterate through the acts, files per act, and sub-files per file
for ((i=0; i<$NUMBER_OF_ACTS; i++)); do
  mkdir -p "act_${i}"
  echo "  - caption: Part $(($i + 1))" >> $toc_file
  echo "    chapters:" >> $toc_file
  for ((j=0; j<$NUMBER_OF_FILES_PER_ACT; j++)); do
    mkdir -p "act_${i}/act_${i}_${j}"
    for ((k=0; k<$NUMBER_OF_SUB_FILES_PER_FILE; k++)); do
      mkdir -p "act_${i}/act_${i}_${j}/act_${i}_${j}_${k}"
      for ((n=1; n<=$NUMBER_OF_NOTEBOOKS; n++)); do
        new_file="act_${i}/act_${i}_${j}/act_${i}_${j}_${k}/act_${i}_${j}_${k}_${n}.ipynb"
        touch "$new_file"
        cp "intro.ipynb" "$new_file" # This line copies the content into the new file
        echo "      - file: $new_file" >> $toc_file
      done
    done
  done
done

# Create _config.yml file
config_file="_config.yml"
echo "title: Your Book Title" > $config_file
echo "copyright: Mwaka" > $config_file
echo "author: Your Name" >> $config_file
echo "logo: https://github.com/muzaale/muzaale.github.io/blob/main/png/hub_and_spoke.jpg?raw=true" >> $config_file

# Build the book with Jupyter Book
cd ..
jb build $SUBDIR_NAME
git clone "https://github.com/$GITHUB_USERNAME/$REPO_NAME"
cp -r $SUBDIR_NAME/* $REPO_NAME
cd $REPO_NAME
git add ./*
git commit -m "$GIT_COMMIT_MESSAGE"
chmod 600 $SSH_KEY_PATH

# Configure the remote URL with SSH
git remote set-url origin "git@github.com:$GITHUB_USERNAME/$REPO_NAME"

# Push changes
git push -u origin main
ghp-import -n -p -f _build/html
rm -rf $REPO_NAME
echo "Jupyter Book content updated and pushed to $GITHUB_USERNAME/$REPO_NAME repository!"

```


# 5 `work/abikesa_clone.sh`

```bash
# Ensure your SSH agent is running:
eval "$(ssh-agent -s)"

# And that your key is added:
ssh-add ~/.ssh/id_rsa

# Test SSH Connection: Test your SSH connection to GitHub:
ssh -T git@github.com

# Clone your private repo
git clone git@github.com:jhurepos/projectalpha.git

ghp_3YBLxNDurzhkp11AvUazjyNdgem05Hw1txxx

git clone https://ghp_3YBLxNDurzhkp11AvUazjyNdgSm05Hw1txxx@github.com/jhurepos/rdc

# 

# was automatically renaming stataone "stataone 2"
mkdir stataone
mv "stataone 2"/* stataone/
rm -r "stataone 2"

# phrase is "work", not "workflow"
```

# 6 `work/abikesa_csv.sh`

```bash
# testing a new workflow
export PATH=$PATH:/applications/stata/statamp.app/contents/macos/
stata-se -b work/abikesa_csv.do
```

# 7 `work/abikesa_editrepo.sh`

```bash
#!/bin/bash

# Usage: ./script.sh
# The script will ask for necessary inputs.

echo "Enter GitHub username:"
read USERNAME

echo "Enter repository name:"
read REPO

echo "Is the repository private? (yes/no)"
read IS_PRIVATE

if [[ "$IS_PRIVATE" == "yes" ]]; then
    echo "Enter your GitHub personal access token:"
    read TOKEN
    URL="https://${TOKEN}@github.com/${USERNAME}/${REPO}.git"
else
    URL="https://github.com/${USERNAME}/${REPO}.git"
fi

echo "Enter your email for git config:"
read EMAIL

echo "Enter the filename of your SSH key (e.g., id_rsa):"
read SSH

echo "Enter the exact name of the file or directory to delete (e.g., obsolete_code.py or 'old folder/'):"
read FILENAME_TO_DELETE

# Clone the repository
git clone "$URL"
cd "$REPO"

# Perform actions on the repository
git checkout main
rm -rf "$FILENAME_TO_DELETE"  # Deletes the exact file or directory specified by the user, handling spaces properly
# work/abikesa_remove_duplicates.sh # prompt based deletion of all duplicates in repo, with confirmation for each files

# Stage the deletions
git add -A 

# The -u option with git add stages all modifications and deletions, but not new files (if there are any new files you want to commit, use git add <file> or git add . to add everything).
# git add -u

# Commit the deletions
git commit -m "Removed '$FILENAME_TO_DELETE' from the repository" 

# Setup SSH (ensure you have configured SSH keys on your GitHub account)
ssh-add -D
chmod 600 "$(eval echo ~/.ssh/$SSH)"
git remote set-url origin "git@github.com:${USERNAME}/${REPO}"
ssh-add "$(eval echo ~/.ssh/$SSH)"

# Set local git configurations
git config --local user.name "$USERNAME"
git config --local user.email "$EMAIL"

# Push the changes
git push -u origin main

# git status
# git log

```

# 8 `work/abikesa_forked.sh`

```bash
#!/bin/bash

# Parameters required by the script
read -p "Enter your GitHub username (e.g., abikesa): " GITHUB_USER
read -p "Enter your GitHub repository name (e.g., nabongo): " GITHUB_REPO
read -p "Enter your email address (e.g., abikesa.sh@gmail.com): " EMAIL_ADDRESS
read -p "Enter your root directory (e.g., ~/Dropbox/1f.ἡἔρις,κ/1.ontology): " ROOT_DIR
read -p "Enter the name of the local directory to be created within the root directory (e.g., charles): " LOCAL_DIR
read -p "Enter the SSH key name (e.g., id_charlesnabongo): " SSH_KEYNAME
read -p "Enter your git commit message (e.g., automated abikesa_fromfolks.sh script): " GIT_COMMIT_MESSAGE

# Folked from some_repo to GITHUB_USER/GITHUB_REPO
cd $(eval echo $ROOT_DIR)
git clone "https://github.com/$GITHUB_USER/$GITHUB_REPO"
cp -r "$GITHUB_REPO/*" "$LOCAL_DIR"
jb build "$LOCAL_DIR"
cp -r "$LOCAL_DIR/*" "$GITHUB_REPO"
cd "$GITHUB_REPO"
git add .
git commit -m "$GIT_COMMIT_MESSAGE"

# Overcoming the error: no sufficient permissions
git remote -v
git remote set-url origin "git@github.com:$GITHUB_USER/$GITHUB_REPO"
git config user.name "$GITHUB_USER"
git config user.email "$EMAIL_ADDRESS"
git checkout main

# Check if SSH keys already exist, and if not, generate a new one
read -p "Enter your SSH key name (e.g., id_charlesnabongo, not ~/.ssh/id_charlesnabongo): " SSH_KEYNAME
SSH_KEYPATH="$HOME/.ssh/$SSH_KEYNAME"

if [ ! -f "$SSH_KEYPATH" ]; then
  ssh-keygen -t ed25519 -C "$EMAIL_ADDRESS" -f $SSH_KEYPATH
fi

cat "$SSH_KEYPATH.pub"
echo "Please manually add the above SSH public key to your GitHub account's SSH keys."
read -p "Once you have added the SSH key to your GitHub account, press Enter to continue..."
eval "$(ssh-agent -s)"
ssh-add -D
ssh-add $SSH_KEYPATH
chmod 600 $SSH_KEYPATH

# Configure the remote URL with SSH
git remote set-url origin "git@github.com:$GITHUB_USER/$GITHUB_REPO"

# Push changes
git push -u origin main
ghp-import -n -p -f _build/html
rm -rf "$GITHUB_REPO"
echo "jb content updated & pushed to $GITHUB_USER/$GITHUB_REPO repository!"


```

# 9 `work/abikesa_key_g.sh`

```bash
#!/bin/bash

# Check if required commands are available
for cmd in git ssh-keygen jb ghp-import; do
  if ! command -v $cmd &> /dev/null; then
    echo "Error: $cmd is not installed."
    exit 1
  fi
done

# User Inputs
read -p "Enter your GitHub username: " GITHUB_USERNAME
read -p "Enter your GitHub repository name: " REPO_NAME
read -p "Enter your email address: " EMAIL_ADDRESS
read -p "Enter your root directory: e.g. ~/dropbox/1f.ἡἔρις,κ/1.ontology " ROOT_DIR
read -p "Enter the name of the subdirectory: " SUBDIR_NAME
read -p "Enter the .ipynb file name in ROOT_DIR:  " POPULATE_BE
read -p "Enter your git commit message: " GIT_COMMIT_MESSAGE
read -p "Enter the number of acts: " NUMBER_OF_ACTS
read -p "Enter the number of files per act: " NUMBER_OF_FILES_PER_ACT
read -p "Enter the number of sub-files per file: " NUMBER_OF_SUB_FILES_PER_FILE
read -p "Enter the number of notebooks: " NUMBER_OF_NOTEBOOKS

# Initialize directory
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"
cd $(eval echo $ROOT_DIR)
rm -rf $REPO_NAME
git clone "https://github.com/$GITHUB_USERNAME/$REPO_NAME.git"
cp -r $REPO_NAME $SUBDIR_NAME

# SSH Setup
# SSH Setup
SSH_KEY_PATH="$HOME/.ssh/id_${SUBDIR_NAME}${REPO_NAME}"

# Check if the SSH key already exists
if [ -f "$SSH_KEY_PATH" ]; then
  # Prompt the user for confirmation before deletion
  read -p "The SSH key already exists. Do you want to delete it? [y/N] " confirm
  if [ "$confirm" == "y" ] || [ "$confirm" == "Y" ]; then
    rm -rf $SSH_KEY_PATH*
    echo "SSH key deleted."
  else
    echo "SSH key not deleted."
  fi
fi

# Generate a new SSH key if it does not exist
if [ ! -f "$SSH_KEY_PATH" ]; then
  ssh-keygen -t ed25519 -C "$EMAIL_ADDRESS" -f $SSH_KEY_PATH
fi

# Copy & paste key to GitHub 
echo "Please manually add the SSH public key to GitHub."
read -p "Press Enter to continue..."
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain $SSH_KEY_PATH

# Jupyter Book Setup
jb build $SUBDIR_NAME

cp -r $SUBDIR_NAME/* $REPO_NAME/
cd $REPO_NAME
git add ./*
git commit -m "$GIT_COMMIT_MESSAGE"
chmod 600 $SSH_KEY_PATH

# Git Operations
git remote set-url origin "git@github.com:$GITHUB_USERNAME/$REPO_NAME.git"
git push -u origin main
ghp-import -n -p -f _build/html
rm -rf $REPO_NAME
echo "Jupyter Book content updated and pushed to $GITHUB_USERNAME/$REPO_NAME repository!"


```

# 10 `work/abikesa_key.sh`

```bash
# Fork/clone then create SSH key

jb build quick
ls -al ~/.ssh
ssh-keygen -t ed25519 -C "abikesa.sh@gmail.com.com" -f ~/.ssh/id_gpt4omni
echo "Please manually add the above SSH public key to your GitHub account's SSH keys."
read -p "Once you have added the SSH key to your GitHub account, press Enter to continue..."
cat ~/.ssh/id_gpt4omni.pub
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_gpt4omni

# Build the book with Jupyter Book
jb build gpt4
git clone "https://github.com/abikesa/omni"
cp -r gpt4/* omni
cd omni
git add ./*
git commit -m "gpt4-o, thank you!"
chmod 600 ~/.ssh/id_gpt4omni

# Configure the remote URL with SSH
git remote set-url origin "git@github.com:abikesa/omni"

# Push changes
git push -u origin main
ghp-import -n -p -f _build/html
cd ..
rm -rf deploy
echo "Jupyter Book content updated and pushed to abikesa/got repository!"



# commitment issues!!

```bash
(myenv) d@Poseidon project % git add ./*
(myenv) d@Poseidon project % git commit -m "zero jb"
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    _config.yml
        deleted:    _toc.yml
        deleted:    intro.md
        deleted:    logo.png
        deleted:    markdown-notebooks.md
        deleted:    markdown.md
        deleted:    notebooks.ipynb
        deleted:    references.bib
        deleted:    requirements.txt
        deleted:    stmarkdown.css.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .DS_Store

no changes added to commit (use "git add" and/or "git commit -a")
(myenv) d@Poseidon project % git rm  _config.yml
rm '_config.yml'
(myenv) d@Poseidon project % git rm   _toc.yml intro.md
rm '_toc.yml'
rm 'intro.md'
(myenv) d@Poseidon project % git rm logo.png markdown-notebooks.md markdown.md notebooks.ipynb references.bib requirements.txt stmarkdown.css.txt
rm 'logo.png'
rm 'markdown-notebooks.md'
rm 'markdown.md'
rm 'notebooks.ipynb'
rm 'references.bib'
rm 'requirements.txt'
rm 'stmarkdown.css.txt'
(myenv) d@Poseidon project % git add ./*
(myenv) d@Poseidon project % git commit -m "no jb"
[main 217e24c] no jb
 10 files changed, 485 deletions(-)
 delete mode 100644 _config.yml
 delete mode 100644 _toc.yml
 delete mode 100644 intro.md
 delete mode 100644 logo.png
 delete mode 100644 markdown-notebooks.md
 delete mode 100644 markdown.md
 delete mode 100644 notebooks.ipynb
 delete mode 100644 references.bib
 delete mode 100644 requirements.txt
 delete mode 100644 stmarkdown.css.txt
(myenv) d@Poseidon project % chmod 600 ~/.ssh/id_intermediateproject
(myenv) d@Poseidon project % git remote set-url origin "git@github.com:jhustata/project"
(myenv) d@Poseidon project % git push -u origin main
To github.com:jhustata/project
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:jhustata/project'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
(myenv) d@Poseidon project % git pull
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 3 (delta 1), reused 1 (delta 1), pack-reused 2
Unpacking objects: 100% (3/3), 1.67 KiB | 568.00 KiB/s, done.
From github.com:jhustata/project
   db503bf..3d0675b  main       -> origin/main
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint: 
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint: 
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.
(myenv) d@Poseidon project % git config pull.rebase true 
(myenv) d@Poseidon project % git pull                    
Successfully rebased and updated refs/heads/main.
(myenv) d@Poseidon project % git push -u origin main     
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 20 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 230 bytes | 230.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:jhustata/project
   3d0675b..20ff4e7  main -> main
branch 'main' set up to track 'origin/main'.
(myenv) d@Poseidon project % 
```

# 11 `work/abikesa_ngoma.sh`

```bash
#!/bin/bash

# Check if required commands are available
for cmd in git ssh-keygen jb ghp-import; do
  if ! command -v $cmd &> /dev/null; then
    echo "Error: $cmd is not installed."
    exit 1
  fi
done

# Input information
read -p "Enter your GitHub username: " GITHUB_USERNAME
read -p "Enter your GitHub repository name: " REPO_NAME
read -p "Enter your email address: " EMAIL_ADDRESS
read -p "Enter your root directory (e.g., ~/Dropbox/1f.ἡἔρις,κ/1.ontology): " ROOT_DIR
read -p "Enter the name of the subdirectory to be created within the root directory: " SUBDIR_NAME
read -p "Enter the name of the populate_be.ipynb file in ROOT_DIR: " POPULATE_BE
read -p "Enter your git commit message: " GIT_COMMIT_MESSAGE
read -p "Enter the number of acts: " NUMBER_OF_ACTS
read -p "Enter the number of files per act: " NUMBER_OF_FILES_PER_ACT
read -p "Enter the number of sub-files per file: " NUMBER_OF_SUB_FILES_PER_FILE
read -p "Enter the number of notebooks: " NUMBER_OF_NOTEBOOKS

# Set up directories and paths
git config --local user.name "$GITHUB_USERNAME"
git config --local user.email "$EMAIL_ADDRESS"
cd $(eval echo $ROOT_DIR)

# Only remove repo if it already exists
if [ -d "$REPO_NAME" ]; then
    echo "Repository directory $REPO_NAME already exists. Removing it..."
    rm -rf $REPO_NAME
fi

# Only create subdirectory if it does not already exist
if [ ! -d "$SUBDIR_NAME" ]; then
    echo "Creating subdirectory $SUBDIR_NAME..."
    mkdir -p $SUBDIR_NAME
fi

# Only copy populate_be if the intro notebook does not already exist
if [ ! -f "$SUBDIR_NAME/intro.ipynb" ]; then
    echo "Copying $POPULATE_BE into $SUBDIR_NAME/intro.ipynb..."
    cp $POPULATE_BE $SUBDIR_NAME/intro.ipynb
fi

cd $SUBDIR_NAME

# Check if SSH keys already exist, and if not, generate a new one
SSH_KEY_PATH="$HOME/.ssh/id_${SUBDIR_NAME}${REPO_NAME}"
if [ ! -f "$SSH_KEY_PATH" ]; then
    echo "Generating SSH keys..."
    ssh-keygen -t ed25519 -C "$EMAIL_ADDRESS" -f $SSH_KEY_PATH
fi

# ... rest of your script remains unchanged
cat ${SSH_KEY_PATH}.pub
echo "Please manually add the above SSH public key to your GitHub account's SSH keys."
read -p "Once you have added the SSH key to your GitHub account, press Enter to continue..."
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain $SSH_KEY_PATH

# Create _toc.yml file
toc_file="_toc.yml"
echo "format: jb-book" > $toc_file
echo "root: intro.ipynb" >> $toc_file # Make sure this file exists
echo "title: Play" >> $toc_file
echo "parts:" >> $toc_file

# Iterate through the acts, files per act, and sub-files per file
for ((i=0; i<$NUMBER_OF_ACTS; i++)); do
  mkdir -p "act_${i}"
  echo "  - caption: Part $(($i + 1))" >> $toc_file
  echo "    chapters:" >> $toc_file
  for ((j=0; j<$NUMBER_OF_FILES_PER_ACT; j++)); do
    mkdir -p "act_${i}/act_${i}_${j}"
    for ((k=0; k<$NUMBER_OF_SUB_FILES_PER_FILE; k++)); do
      mkdir -p "act_${i}/act_${i}_${j}/act_${i}_${j}_${k}"
      for ((n=1; n<=$NUMBER_OF_NOTEBOOKS; n++)); do
        new_file="act_${i}/act_${i}_${j}/act_${i}_${j}_${k}/act_${i}_${j}_${k}_${n}.ipynb"
        touch "$new_file"
        cp "intro.ipynb" "$new_file" # This line copies the content into the new file
        echo "      - file: $new_file" >> $toc_file
      done
    done
  done
done

# Create _config.yml file
config_file="_config.yml"
echo "title: Your Book Title" > $config_file
echo "copyright: Mwaka" > $config_file
echo "author: Your Name" >> $config_file
echo "logo: https://github.com/muzaale/muzaale.github.io/blob/main/png/hub_and_spoke.jpg?raw=true" >> $config_file

# Build the book with Jupyter Book
cd ..
jb build $SUBDIR_NAME
git clone "https://github.com/$GITHUB_USERNAME/$REPO_NAME"
cp -r $SUBDIR_NAME/* $REPO_NAME
cd $REPO_NAME
git add ./*
git commit -m "$GIT_COMMIT_MESSAGE"
chmod 600 $SSH_KEY_PATH

# Configure the remote URL with SSH
git remote set-url origin "git@github.com:$GITHUB_USERNAME/$REPO_NAME"

# Push changes
git push -u origin main
ghp-import -n -p -f _build/html
rm -rf $REPO_NAME
echo "Jupyter Book content updated and pushed to $GITHUB_USERNAME/$REPO_NAME repository!"


```

# 12 `work/abikesa_remove_duplicages.sh`

```bash
#!/bin/bash

# Check if a directory path is provided
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <directory_path>"
    exit 1
fi

directory_path=$1

# Check if the provided path is a directory
if [ ! -d "$directory_path" ]; then
    echo "Error: '$directory_path' is not a valid directory."
    exit 1
fi

echo "Scanning for duplicates in $directory_path..."

# Change to the specified directory
cd "$directory_path"

# Loop through all files in the specified directory
for file in *; do
    if [[ -f "$file" ]]; then
        # Check if a duplicate with a ' 2' suffix exists
        duplicate="${file%.*} 2.${file##*.}"
        if [[ -f "$duplicate" ]]; then
            echo "Duplicate found: $duplicate"
            # Ask user if they want to delete the duplicate
            read -p "Do you want to delete this file? (y/n) " answer
            case $answer in
                [Yy]* ) rm "$duplicate"; echo "$duplicate deleted.";;
                [Nn]* ) echo "Kept $duplicate.";;
                * ) echo "Invalid response. No action taken.";;
            esac
        fi
    fi
done

echo "Duplicate scan complete."

```

# 13 `work/abikesa_rmdir.sh`

```bash
#!/bin/bash

# Variables
TOKEN="$1"
REPO_URL="$2"
DELETE_PATH="$3"
SSH_KEY_PATH="$4"
USERNAME="$5"
EMAIL="$6"

# Clone the repo
git clone "https://$TOKEN@github.com/$REPO_URL.git"
cd "$(basename "$REPO_URL")"

# Switch to the main branch
git checkout main

# Remove the specified path (can be directory, file, or file type)
rm -r $DELETE_PATH

# Stage, commit, and push deletions
git add -A 
git commit -m "Removed $DELETE_PATH in main directory"
git remote -v

# Unblock if repo linked to ~/.ssh/id_etc

# ssh-add -D 
# chmod 600 "$SSH_KEY_PATH"

git remote set-url origin "git@github.com:$REPO_URL.git"

# ssh-add "$SSH_KEY_PATH"

git config --local user.name "$USERNAME"
git config --local user.email "$EMAIL"
git push -u origin main

# token for stata/intermediate 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
```

# 14 `work/abikesa_rmfiles.sh`

```bash
# Tokens: Profile photo, Settings, Developer Settings
# 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
# jhustata/basic

# Delete repo folder
git clone https://token@github.com/abikesa/ssh.git
cd ssh
git checkout main
rm -r ssh*

# Stage the Deletions
git add -A 

# Commit the Deletions
git commit -m "Removed ssh in main directory" 
    
# Push the Deletions
git remote -v
ssh-add -D 
chmod 600 "$(eval echo ~/.ssh/id_sharessh)"
git remote set-url origin "git@github.com:abikesa/ssh"
ssh-add "$(eval echo ~/.ssh/id_sharessh)"
git config --local user.name "abikesa"
git config --local user.email "abikesa.sh@gmail.com"
git push -u origin main

```

# 15 `work/abikesa_rmstuff.sh`

```bash
# $TOKEN if private repo
# git clone "https://$TOKEN@github.com/$REPO_URL.git"
# https://github.com/jhustata/basic

read -p "Enter your GitHub username: " USERNAME
read -p "Enter your GitHub repo: " REPO
read -p "Enter your GitHub filename: " FILENAME
read -p "Enter your GitHub email: " EMAIL
read -p "Enter your GitHub ssh: " SSH

# Switch to the main branch
git clone "https://github.com/$USERNAME/$REPO"
cd "$(basename "https://github.com/$USERNAME/$REPO")"

# Remove the specified path (can be directory, file, or file type)
rm -rf $FILENAME

# Stage, commit, and push deletions
git add -A 
git commit -m "Removed $FILENAME in main directory"
git remote -v

# Permissions for access
chmod 600 "$(eval echo $SSH)"
git remote set-url origin "git@github.com:$USERNAME/$REPO_NAME"
ssh-add "$(eval echo $SSH)"
git config --local user.name "$USERNAME"
git config --local user.email "$EMAIL"

# Checkout the main branch
git checkout main
git push -u origin main
```

# 16 `work/abikesa_statacurl.sh`

```bash
# remote work
# Download the remote script using curl
curl -O https://github.com/abikesa/do/raw/main/hello.do

# Run Stata in batch mode with the downloaded script
export PATH=$PATH:/applications/stata/statamp.app/contents/macos/
stata-mp -b hello.do
```

# 17 `work/bash.sh`

```bash
#!/bin/bash

# User input or some other dynamic data
dynamic_part=""

# Some conditional logic to decide the final directory
if [ "$dynamic_part" == "cst" ]; then
    dir_to_go="./../$dynamic_part"
else
    dir_to_go="~/dropbox/1f.ἡἔρις,κ/1.ontology/"
fi

# Now, we use eval and echo to dynamically generate the directory path
cd $(eval echo $dir_to_go)
jb build git2

```

# 18 `jb build bloc/denotas`

Since `jupyter-client` cannot be found in the specified channels, let's try to resolve the issue by installing it via pip, which might help us bypass the package resolution problems in conda.

First, let's ensure that `pip` is available in your conda environment and then use it to install the necessary packages:

1. **Install `jupyter-client` using pip**:
   ```bash
   pip install jupyter-client
   ```

2. **Install `nbclient` using pip**:
   ```bash
   pip install nbclient
   ```

3. **Install `jupyter` using pip**:
   ```bash
   pip install jupyter
   ```

After installing the required packages, try to register your environment as a Jupyter kernel and then build your Jupyter Book again:

4. **Register the kernel**:
   ```bash
   python -m ipykernel install --user --name=myenv --display-name "Python (myenv)"
   ```

5. **Build the Jupyter Book**:
   ```bash
   jb build bloc
   ```

Here are the steps consolidated:

```bash
# Activate your environment
conda activate /Users/d/Documents/github/myenv

# Install jupyter-client using pip
pip install jupyter-client

# Install nbclient using pip
pip install nbclient

# Install jupyter using pip
pip install jupyter

# Register the kernel
python -m ipykernel install --user --name=myenv --display-name "Python (myenv)"

# Build the Jupyter Book
jb build bloc
```

These steps should help you resolve the package availability issues and proceed with building your Jupyter Book.

# 19 `git config --global credential`

It looks like you're encountering an issue with pushing your changes to the GitHub repository due to an SSL error. Here are some steps to troubleshoot and resolve the issue:

### Check Your GitHub Credentials

1. **Re-authenticate with GitHub**:
   - Ensure you're logged in with the correct GitHub account.
   - Open the terminal and run:
     ```bash
     git config --global credential.helper cache
     git credential-cache exit
     ```

2. **Clear Cached Credentials**:
   - Clear any cached credentials by running:
     ```bash
     git credential-cache exit
     ```

### SSL/TLS Configuration

3. **Check SSL/TLS Configuration**:
   - Modify the Git SSL verification settings. Sometimes, the SSL certificate verification might cause issues. Temporarily disable SSL verification to test:
     ```bash
     git config --global http.sslVerify false
     ```

4. **Update Git Configuration**:
   - Update your Git configuration to use a more robust SSL backend:
     ```bash
     git config --global http.sslBackend openssl
     ```

5. **Re-enable SSL Verification**:
   - If disabling SSL verification works, re-enable it to maintain security:
     ```bash
     git config --global http.sslVerify true
     ```

### Network Configuration

6. **Network Issues**:
   - Check your network connection. A weak or unstable connection might cause this error. Try pushing your changes from a different network.

### Large File Handling

7. **Reduce Push Size** (`skipped this`): 
   - If your push contains large files, consider reducing the size of the push by using Git LFS (Large File Storage):
     ```bash
     git lfs install
     git lfs track "*.your-large-file-type"
     git add .gitattributes
     git add your-large-file
     git commit -m "Add large file with Git LFS"
     git push origin main
     ```

### Retry Push

8. **Retry the Push**:
   - After trying the above steps, retry pushing your changes:
     ```bash
     git push origin main
     ```

9. Repeat step `# 2` above: `work/abikesa_jbb_https.sh`
   - I can confirm that this worked in the outlined 18 -> 19 -> 2 sequence :)

By following these steps, you should be able to resolve the SSL error and successfully push your changes to the GitHub repository. If the issue persists, you might need to check for updates or reinstall Git and related tools.

# 20 VSCode Credential Madness

To change the GitHub account you're using in VSCode, you can follow these steps:

1. **Sign Out from GitHub in VSCode**:
   - Open VSCode.
   - Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on macOS) to open the Command Palette.
   - Type `Sign Out` and select `GitHub: Sign Out`.
   - This will sign you out of your current GitHub account.

2. **Remove Old Credentials**:
   - Open a terminal or command prompt.
   - Run the following command to clear any cached credentials:
     ```bash
     git credential-cache exit
     ```

3. **Clear Git Credentials from Credential Manager**:
   - On **Windows**:
     - Open the Credential Manager (Control Panel > User Accounts > Credential Manager).
     - Look for entries related to GitHub and remove them.
   - On **macOS**:
     - Open the Keychain Access application.
     - Search for `github.com` and delete any related entries.
   - On **Linux**:
     - Credentials are often stored in a file called `~/.git-credentials`. You can edit this file to remove old credentials.

4. **Sign In to New GitHub Account in VSCode**:
   - Open VSCode.
   - Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on macOS) to open the Command Palette.
   - Type `GitHub: Sign In` and select it.
   - Follow the prompts to sign in with your new GitHub account.

5. **Update Remote URL to Use New Account**:
   - If you are switching to a new account, you might need to update the remote URL of your repository:
     ```bash
     git remote set-url origin https://github.com/newusername/repository.git
     ```

6. **Push Changes Using New Account**:
   - Once signed in with the new account, try pushing your changes again:
     ```bash
     git push origin main
     ```

By following these steps, you should be able to switch your GitHub account in VSCode and push your changes using the new account credentials. Let me know if you encounter any issues or need further assistance!

# 21 `conda` vs. `venv`

It looks like your virtual environment is structured differently, likely because it was created using Anaconda or a similar distribution. Instead of the standard `venv` structure, it seems you have a conda environment. Here’s how you can work with it:

Sure, here’s a coherent step-by-step guide to setting up a Python virtual environment using `venv` in VSCode on your new Mac laptop:

### Setting Up Python Virtual Environment in VSCode

1. **Install Python:**
   Ensure you have Python installed on your Mac. You can download it from the official Python website: [Python Downloads](https://www.python.org/downloads/).

2. **Open Terminal:**
   Open the Terminal application on your Mac.

3. **Navigate to Your Project Directory:**
   Change the directory to where you want to set up your virtual environment. For example:

   ```bash
   cd /path/to/your/project
   ```

4. **Create a Virtual Environment:**
   Create a virtual environment named `myenv` using `venv`:

   ```bash
   python3 -m venv myenv
   ```

5. **Activate the Virtual Environment:**
   Activate the virtual environment:

   ```bash
   source myenv/bin/activate
   ```

6. **Install Required Packages:**
   Install the necessary Python packages within the virtual environment:

   ```bash
   pip install numpy matplotlib scipy pandas ipykernel
   ```

7. **Add the Virtual Environment to Jupyter:**
   Add your virtual environment to Jupyter kernels:

   ```bash
   python -m ipykernel install --name=myenv --display-name "Python (myenv)"
   ```

### Configuring VSCode

1. **Open VSCode:**
   Open Visual Studio Code on your Mac.

2. **Open Command Palette:**
   Open the command palette by pressing `Cmd+Shift+P`.

3. **Select Python Interpreter:**
   Type `Python: Select Interpreter` and select it from the dropdown list.

4. **Choose the Correct Environment:**
   Choose the interpreter located at `/path/to/your/project/myenv/bin/python`.

5. **Reload VSCode:**
   Reload VSCode to ensure all settings are applied. You can do this by typing `Reload Window` in the command palette and selecting it.

### Verifying the Setup

1. **Open a New Terminal in VSCode:**
   Open a new terminal within VSCode. It should show `(myenv)` indicating the virtual environment is activated.

2. **Run a Python Script:**
   Create and run a Python script to verify everything is set up correctly. Here’s an example script:

   ```python
   import numpy as np
   import matplotlib.pyplot as plt
   from scipy import stats
   import pandas as pd

   # Sample DataFrame
   data = {'col1': [1, 2, 3, 4], 'col2': [5, 6, 7, 8]}
   df = pd.DataFrame(data)

   # Basic Statistics
   print(df.describe())

   # Plot
   df.plot(kind='bar')
   plt.show()
   ```

By following these steps, you should have a fully functioning Python development environment in VSCode on your new Mac laptop. If you encounter any issues during the setup, feel free to ask for further assistance!

# 22 `pip install`

To simplify the installation of multiple packages, you can create a script that installs all the required packages at once. Here’s how you can do it:

1. **Create a Requirements File:**
   Create a file named `requirements.txt` and list all the packages you want to install. For example:

   ```txt
   numpy
   matplotlib
   scipy
   pandas
   ipykernel
   requests
   statsmodels
   geopandas
   networkx
   ```

2. **Install Packages Using the Requirements File:**
   You can then install all the packages listed in the `requirements.txt` file with a single command. Here’s how you can do it:

   ```bash
   pip install -r requirements.txt
   ```

### Step-by-Step Instructions:

1. **Create and Activate a Virtual Environment:**

   ```bash
   cd /Users/d/Documents/GitHub
   python3 -m venv myenv
   source myenv/bin/activate
   ```

2. **Create a Requirements File:**

   Create a file named `requirements.txt` in your project directory and add the following content:

   ```txt
   numpy
   matplotlib
   scipy
   pandas
   ipykernel
   requests
   statsmodels
   geopandas
   networkx
   ```

3. **Install Packages Using the Requirements File:**

   Run the following command to install all the packages:

   ```bash
   pip install -r requirements.txt
   ```

4. **Add the Virtual Environment to Jupyter:**

   After installing the packages, add your virtual environment to Jupyter:

   ```bash
   python -m ipykernel install --name=myenv --display-name "Python (myenv)"
   ```

### Example Script

Here’s an example script (`setup_env.sh`) that you can run to automate these steps:

```bash
#!/bin/bash

# Navigate to the project directory
cd /Users/d/Documents/GitHub

# Remove the existing virtual environment if it exists
rm -rf myenv

# Create a new virtual environment
python3 -m venv myenv

# Activate the virtual environment
source myenv/bin/activate

# Create a requirements file
cat <<EOT > requirements.txt
numpy
matplotlib
scipy
pandas
ipykernel
requests
statsmodels
geopandas
networkx
EOT

# Install the required packages
pip install -r requirements.txt

# Add the virtual environment to Jupyter
python -m ipykernel install --name=myenv --display-name "Python (myenv)"

# Clean up
rm requirements.txt

echo "Setup completed successfully!"
```

To run the script:

1. Save the script as `setup_env.sh`.
2. Make it executable:

   ```bash
   chmod +x setup_env.sh
   ```

3. Run the script:

   ```bash
   ./setup_env.sh
   ```

This script will set up your virtual environment, install the required packages, and add the environment to Jupyter.