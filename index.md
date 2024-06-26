# Bash

![](https://cdn-wordpress-info.futurelearn.com/info/wp-content/uploads/b520f2d9-52b6-4eef-b207-ab75e6e04c46.png)

## 1 `$()`

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

## 2 `work/abikesa_jbb_https.sh`

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

## 3 `work/abikesa_jbb_ssh.sh`

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

## 4 `work/abikesa_jbc.sh`

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


## 5 `work/abikesa_clone.sh`

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

## 6 `work/abikesa_csv.sh`

```bash
# testing a new workflow
export PATH=$PATH:/applications/stata/statamp.app/contents/macos/
stata-se -b work/abikesa_csv.do
```

## 7 `work/abikesa_editrepo.sh`

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
read SSHw

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

## 8 `work/abikesa_forked.sh`

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

## 9 `work/abikesa_key_g.sh`

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

## 10 `work/abikesa_key.sh`

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
```


## 11 `work/abikesa_ngoma.sh`

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

## 12 `work/abikesa_remove_duplicages.sh`

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

## 13 `work/abikesa_rmdir.sh`

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

# token for stata/intermediate ghp_thisisafaketoken
```

## 14 `work/abikesa_rmfiles.sh`

```bash
# Tokens: Profile photo, Settings, Developer Settings
# ghp_thisisafaketoken
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

## 15 `work/abikesa_rmstuff.sh`

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

## 16 `work/abikesa_statacurl.sh`

```bash
# remote work
# Download the remote script using curl
curl -O https://github.com/abikesa/do/raw/main/hello.do

# Run Stata in batch mode with the downloaded script
export PATH=$PATH:/applications/stata/statamp.app/contents/macos/
stata-mp -b hello.do
```

## 17 `work/bash.sh`

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

## 18 `jb build bloc/denotas`

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

## 19 `git config --global credential`

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

## 20 `git credential-cache exit`

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

## 21 `conda` vs. `venv`

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

## 22 `pip install`

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

## 23 `work/abikesa_jbb_ssh.sh` again!

The error message indicates that there are several issues encountered during the build process and when pushing to the repository. Here’s a step-by-step guide to address these issues:

### 1. Handling Jupyter Book Build Errors
The primary error is due to the kernel dying before replying to `kernel_info`. This could be due to a misconfigured kernel or missing dependencies. 

#### Steps to Resolve:

1. **Install Missing Dependencies:**
   Ensure that all required dependencies are installed in your virtual environment.

   ```bash
   pip install jupyter jupyterlab nbclient sphinx myst-nb
   ```

2. **Check the Kernel Configuration:**
   Ensure that the `stata_kernel` and other required kernels are installed correctly. You can install the `stata_kernel` by following its [installation instructions](https://kylebarron.github.io/stata_kernel/getting_started/).

3. **Rebuild the Jupyter Book:**
   After ensuring all dependencies are installed, try rebuilding the Jupyter Book.

   ```bash
   jb build /Users/d/Documents/GitHub/work
   ```

### 2. Addressing GitHub Push Issues
The error message indicates a failure when pushing to GitHub due to an unexpected disconnect.

#### Steps to Resolve:

1. **Check SSH Key Configuration:**
   Ensure that your SSH key is correctly configured with GitHub.

   ```bash
   ssh-add ~/.ssh/id_workflow
   ssh -T git@github.com
   ```

   The second command should result in a successful authentication message if your SSH key is set up correctly.

2. **Push Changes Again:**
   Retry pushing your changes to GitHub.

   ```bash
   git push origin main
   ```

3. **Check Network Connection:**
   Ensure that your network connection is stable and retry the push operation. If the problem persists, try using HTTPS instead of SSH for pushing.

   ```bash
   git remote set-url origin https://github.com/abikesa/flow.git
   git push origin main
   ```

### Summary of Steps

1. **Install Required Dependencies:**
   Ensure all necessary packages are installed in your virtual environment.

   ```bash
   source /Users/d/Documents/GitHub/myenv/bin/activate
   pip install jupyter jupyterlab nbclient sphinx myst-nb stata_kernel
   ```

2. **Rebuild Jupyter Book:**
   Retry building the Jupyter Book after installing dependencies.

   ```bash
   jb build /Users/d/Documents/GitHub/work
   ```

3. **Check SSH Key Configuration and Push:**
   Ensure SSH keys are configured correctly and push changes to GitHub.

   ```bash
   ssh-add ~/.ssh/id_workflow
   ssh -T git@github.com
   git push origin main
   ```

4. **Use HTTPS for Git Push (if needed):**
   If SSH continues to fail, switch to HTTPS for Git operations.

   ```bash
   git remote set-url origin https://github.com/abikesa/flow.git
   git push origin main
   ```

Following these steps should help resolve the issues with the Jupyter Book build and pushing to GitHub. If you encounter further issues, please provide the specific error messages for additional assistance.

## 24 `~/.ssh/id_workflow`

Based on the error messages, it seems like there are a few issues you need to resolve to successfully build your Jupyter Book and push to your GitHub repository.

### 1. Jupyter Book Build Errors

#### Issue 1: Kernel died before replying to `kernel_info`

This issue indicates that the Jupyter kernel is not properly configured or that there are missing dependencies.

#### Solution:
- Ensure all necessary packages are installed in your virtual environment:
  ```bash
  source ~/documents/github/myenv/bin/activate
  pip install jupyter jupyterlab nbclient sphinx myst-nb stata_kernel
  ```

- Ensure the `stata_kernel` is installed and properly configured. Follow the [stata_kernel installation instructions](https://kylebarron.github.io/stata_kernel/getting_started/).

- Rebuild the Jupyter Book:
  ```bash
  jb build ~/documents/github/work
  ```

### 2. GitHub Push Issues

#### Issue 2: Connection to GitHub closed by remote host

This issue might be related to network instability or SSH key configuration.

#### Solution:
1. **Check SSH Key Configuration:**
   Ensure your SSH key is correctly configured with GitHub.
   ```bash
   ssh-add ~/.ssh/id_workflow
   ssh -T git@github.com
   ```

2. **Use HTTPS for Git Push (if SSH fails):**
   If SSH continues to fail, switch to HTTPS for Git operations:
   ```bash
   git remote set-url origin https://github.com/abikesa/flow.git
   git push origin main
   ```

### Steps to Resolve:

1. **Install Required Dependencies:**
   ```bash
   source ~/documents/github/myenv/bin/activate
   pip install jupyter jupyterlab nbclient sphinx myst-nb stata_kernel
   ```

2. **Rebuild Jupyter Book:**
   ```bash
   jb build ~/documents/github/work
   ```

3. **Check SSH Key Configuration and Push:**
   ```bash
   ssh-add ~/.ssh/id_workflow
   ssh -T git@github.com
   git push origin main
   ```

4. **Switch to HTTPS for Git Push (if needed):**
   ```bash
   git remote set-url origin https://github.com/abikesa/flow.git
   git push origin main
   ```

Following these steps should help resolve the issues with the Jupyter Book build and pushing to GitHub. If you encounter further issues, please provide the specific error messages for additional assistance.

## 25 `iframe` & `source`

The `src` attribute in an `<iframe>` element stands for "source." It specifies the URL of the document or content that you want to embed within the iframe. Essentially, it tells the browser where to find the content that will be displayed inside the iframe.

Here is the `src` attribute in the context of an `<iframe>` element:

```html
<iframe src="https://www.example.com" width="600" height="400"></iframe>
```

In this example, the iframe will load and display the content from "https://www.example.com".

## 26 `pip install`

Great, you've successfully authenticated with GitHub using your SSH key. Now let's address the remaining issues:

### 1. Jupyter Book Build Errors

First, make sure all required packages are installed in your virtual environment:

```bash
source ~/documents/github/myenv/bin/activate
pip install jupyter jupyterlab nbclient sphinx myst-nb stata_kernel
```

Next, ensure the `stata_kernel` is properly installed and configured. Follow the [stata_kernel installation instructions](https://kylebarron.github.io/stata_kernel/getting_started/).

Then, rebuild your Jupyter Book:

```bash
jb build ~/documents/github/work
```

### 2. GitHub Push Issues

Now that SSH is correctly configured, you should be able to push your changes to GitHub. Use the following commands:

```bash
git add .
git commit -m "vscode credentials first, then ssh"
git push origin main
```

### Rebuild Jupyter Book and Push Changes to GitHub

1. **Install Required Dependencies:**
   ```bash
   source ~/documents/github/myenv/bin/activate
   pip install jupyter jupyterlab nbclient sphinx myst-nb stata_kernel
   ```

2. **Rebuild Jupyter Book:**
   ```bash
   jb build ~/documents/github/work
   ```

3. **Push Changes to GitHub:**
   ```bash
   git add .
   git commit -m "Rebuild Jupyter Book and resolve issues"
   git push origin main
   ```

Following these steps should help you rebuild your Jupyter Book and successfully push your changes to GitHub. If you encounter any further issues, please provide the specific error messages for additional assistance.

## 27 `git config --global http.postBuffer 524288000`

This issue often arises due to network problems, server-side restrictions, or SSH configuration issues. Here are some steps to troubleshoot and potentially resolve the problem:

### 1. Verify SSH Configuration

Ensure that your SSH key is correctly configured and added to the SSH agent:

```bash
ssh-add -l
```

If your key is not listed, add it again:

```bash
ssh-add ~/.ssh/id_workflow
```

### 2. Increase Git Buffer Size

Sometimes, increasing the Git buffer size can resolve issues related to large pushes:

```bash
git config --global http.postBuffer 524288000
```

### 3. Check Your Network Connection

Ensure you have a stable internet connection. If you're on a VPN or behind a proxy, try disabling it and pushing again.

### 4. Use SSH Verbose Mode

Enable verbose mode to get more information about the SSH connection:

```bash
GIT_SSH_COMMAND="ssh -v" git push origin main
```

### 5. Split Your Push

If the repository is large, consider pushing smaller parts:

```bash
git add .
git commit -m "Partial commit"
git push origin main
```

### 6. Retry the Push

Sometimes, simply retrying the push can work:

```bash
git push origin main
```

### 7. Check GitHub Status

Ensure GitHub is not experiencing any issues by checking the [GitHub status page](https://www.githubstatus.com/).

### Summary

1. Ensure SSH key is added to the agent.
2. Increase Git buffer size.
3. Check network connection.
4. Use SSH verbose mode for more details.
5. Split large pushes into smaller commits.
6. Retry the push.
7. Check GitHub status.

### Example Commands

```bash
ssh-add ~/.ssh/id_workflow
git config --global http.postBuffer 524288000
GIT_SSH_COMMAND="ssh -v" git push origin main
```

If you still encounter issues after trying these steps, please share any additional output or error messages for further assistance.

## 28 `bash`

The "unexpected disconnect while reading sideband packet" error indicates that there might be an issue on GitHub's end or with the network connection between your local machine and GitHub's servers. Here are a few additional steps you can try to resolve the issue:

### 1. Reduce Pack Size

Sometimes large pack files can cause issues. Try reducing the pack size:

```bash
git config --global pack.windowMemory "100m"
git config --global pack.packSizeLimit "100m"
git config --global pack.threads "1"
```

### 2. Use HTTP/HTTPS Instead of SSH

If SSH continues to cause issues, you might want to try using HTTPS to push the changes:

```bash
git remote set-url origin https://github.com/abikesa/flow.git
git push -u origin main
```

### 3. Ensure Network Stability

Make sure your network connection is stable and not experiencing any interruptions. If possible, try switching to a different network or disabling VPN/proxy.

### 4. Use Verbose Mode for More Information

Run the push command with increased verbosity to gather more information:

```bash
GIT_TRACE_PACKET=1 GIT_TRACE=1 GIT_CURL_VERBOSE=1 git push origin main
```

### 5. Split Your Commit

If you have a very large commit, try splitting it into smaller commits and pushing them incrementally:

```bash
git reset --soft HEAD~1
git add -p  # or git add -i
git commit -m "Part 1 of the changes"
git push origin main
```

### Example Commands

```bash
# Reduce pack size
git config --global pack.windowMemory "100m"
git config --global pack.packSizeLimit "100m"
git config --global pack.threads "1"

# Switch to HTTPS
git remote set-url origin https://github.com/abikesa/flow.git
git push -u origin main

# Push with verbose mode
GIT_TRACE_PACKET=1 GIT_TRACE=1 GIT_CURL_VERBOSE=1 git push origin main

# Split commit and push
git reset --soft HEAD~1
git add -p  # or git add -i
git commit -m "Part 1 of the changes"
git push origin main
```

If you continue to encounter issues, consider reaching out to GitHub support for further assistance, as the problem may be related to server-side restrictions or issues.

## 29 `rm: quickdeploy: Permission denied`

It looks like you're encountering a permission issue while trying to remove the `quickdeploy` directory. You might need to use `sudo` to remove the directory with elevated privileges.

Try running the following command:

```sh
sudo rm -rf quickdeploy
```

You'll be prompted to enter your password. This command should remove the directory regardless of the permissions set on it.

## 30 `ghp-import -n -p -f _build/html`

Certainly! Let's go through the process of setting up your workflow on your new Studio Pro and MacBookPro from scratch, step by step. **(Later create a hybrid of `#30` & `#31` for optimal setup process)**

### Step 1: Install Homebrew
Homebrew is a package manager for macOS that makes installing third-party software easy.

1. Open Terminal.
2. Install Homebrew by pasting the following command and pressing Enter:
   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

- Run these two commands in your terminal to add Homebrew to your PATH:

```bash
    (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/apollo/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
```

- Run brew help to get started
- Further documentation:
    https://docs.brew.sh

### Step 2: Install Python
Using Homebrew, install Python.

1. In Terminal, run:
   ```sh
   brew install python
   ```
Python has been installed as
  `/opt/homebrew/bin/python3`

Unversioned symlinks `python`, `python-config`, `pip` etc. pointing to
`python3`, `python3-config`, `pip3` etc., respectively, have been installed into
  `/opt/homebrew/opt/python@3.12/libexec/bin`

See: https://docs.brew.sh/Homebrew-and-Python

2. Verify the installation:
   ```sh
   python3 --version
   ```

### Step 3: Install VSCode
Download and install Visual Studio Code from the [official website](https://code.visualstudio.com/).

### Step 4: Install VSCode Extensions
Install the following extensions in VSCode:
1. Python
2. Jupyter
3. GitHub Repositories

You can find and install these extensions from the Extensions view in VSCode (`Cmd+Shift+X`).

### Step 5: Set Up a Virtual Environment
1. Create a directory for your project:
   ```sh
   mkdir my_project
   cd my_project
   ```
2. Set up a virtual environment:
   ```sh
   python3 -m venv venv
   ```
3. Activate the virtual environment:
   ```sh
   source venv/bin/activate
   ```

### Step 6: Install Jupyter Book and ghp-import
1. Install `pip` if not already installed:
   ```sh
   python3 -m ensurepip --upgrade
   ```
2. Install Jupyter Book:
   ```sh
   pip install jupyter-book
   ```

WARNING: You are using pip version 21.2.4; however, version 24.0 is available.
You should consider upgrading via the '/Users/apollo/Documents/Athena/venv/bin/python3 -m pip install --upgrade pip' command.

3. Install `ghp-import`:
   ```sh
   pip install ghp-import
   ```

### Step 7: Create a Jupyter Book
1. Create a new Jupyter Book:
   ```sh
   jb create mybook
   cd mybook
   ```
2. Build the book:
   ```sh
   jb build .
   ```

### Step 8: Use ghp-import to Publish to GitHub Pages
1. Ensure your book is built:
   ```sh
   jb build .
   ```
2. Use `ghp-import` to publish to GitHub Pages:
   ```sh
   ghp-import -n -p -f _build/html
   ```

### Step 9: Automate Workflow with VSCode
1. Open your project in VSCode.
2. Use the integrated terminal (`Ctrl+``) to activate your virtual environment:
   ```sh
   source venv/bin/activate
   ```
3. Create tasks in VSCode to automate your workflow (optional).

### Additional Tips
- Ensure your GitHub repository is correctly set up for GitHub Pages.
- Regularly commit and push changes to your repository.
- Customize your Jupyter Book as needed by editing the configuration files.

By following these steps, you should be able to reproduce your previous workflow on your new machines. If you encounter any issues, feel free to ask for further assistance!

## 31 `fatal: the remote end hung up unexpectedly`

You're right. For a public repository using HTTPS, GitHub credentials should not be necessary unless you're pushing changes. If you still encounter issues, the problem might be related to the network or other configurations. Let's adjust the approach to focus on the push error and the SSL issue:

### Step-by-Step Guide to Resolve Push Issues and SSL Errors

### Step 1: Increase Git Buffer Size
1. Open your terminal.
2. Increase the Git buffer size by running:
   ```sh
   git config --global http.postBuffer 524288000
   ```

### Step 2: Retry the Push
1. Ensure you are in your project directory.
2. Retry pushing your changes:
   ```sh
   git add .
   git commit -m "workflow 3.0 (ghp-import from any computer)"
   git push origin main
   ```

### Step 3: SSL Configuration

If you still face SSL issues, you can try reconfiguring the SSL settings for Git:

1. Disable SSL verification (not recommended for long-term use):
   ```sh
   git config --global http.sslVerify false
   ```

2. Retry the push:
   ```sh
   git push origin main
   ```

3. If the push is successful, re-enable SSL verification for security:
   ```sh
   git config --global http.sslVerify true
   ```

### Step 4: Reconfigure Remote URL (if needed)
If the issue persists, ensure your remote URL is correctly set:

1. Check the current remote URL:
   ```sh
   git remote -v
   ```

2. Reconfigure the remote URL (if necessary):
   ```sh
   git remote set-url origin https://github.com/abikesa/dionysus.git
   ```

### Step 5: Verify Your Changes on GitHub
1. Go to your GitHub repository in your web browser.
2. Verify that your changes have been pushed successfully and that the GitHub Pages site is properly updated.

### Step-by-Step Recap (Updated)

1. **Install Homebrew**:
   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install Python**:
   ```sh
   brew install python
   ```

3. **Install VSCode** from [VSCode website](https://code.visualstudio.com/).

4. **Install VSCode Extensions**:
   - Python
   - Jupyter
   - GitHub Repositories

5. **Set Up a Virtual Environment**:
   ```sh
   mkdir my_project
   cd my_project
   python3 -m venv venv
   source venv/bin/activate
   ```

6. **Install Jupyter Book and ghp-import**:
   ```sh
   python3 -m ensurepip --upgrade
   pip install jupyter-book ghp-import
   ```

7. **Create and Build a Jupyter Book**:
   ```sh
   jb create mybook
   cd mybook
   jb build .
   ```

8. **Publish to GitHub Pages**:
   ```sh
   ghp-import -n -p -f _build/html
   ```

9. **Increase Git Buffer Size and Retry Push**:
   ```sh
   git config --global http.postBuffer 524288000
   git add .
   git commit -m "workflow 3.0 (ghp-import from any computer)"
   git push origin main
   ```

10. **SSL Configuration (if needed)**:
    ```sh
    git config --global http.sslVerify false
    git push origin main
    git config --global http.sslVerify true
    ```

By following these updated steps, you should be able to resolve the issue and successfully push your changes to GitHub. If you encounter any further issues, let me know!

By following these steps, you should be able to resolve the issue and successfully push your changes to GitHub. If you encounter any further issues, let me know!

## 32 `Software Update Tool`

Only received this query on Hades/MacBookPro. Didn't showup on Apollo/Mac Studio Pro

Software Update Tool

Finding available software

Downloading Command Line Tools for Xcode

Downloaded Command Line Tools for Xcode
Installing Command Line Tools for Xcode

## 33 `xcode-select --install`

Yes, you may need to install Xcode Command Line Tools to ensure that you have all the necessary compilers and libraries for building Python packages and other software. Here’s how you can install it:

### Step 1: Install Xcode Command Line Tools
1. Open Terminal.
2. Run the following command to install Xcode Command Line Tools:
   ```sh
   xcode-select --install
   ```

This will prompt a pop-up window to install the tools. Follow the instructions to complete the installation.

### Step 2: Confirm Installation
1. To confirm that the installation was successful, run:
   ```sh
   xcode-select -p
   ```

This should return `/Library/Developer/CommandLineTools`.

### Step-by-Step Guide Recap (Including Xcode Command Line Tools)

1. **Install Homebrew**:
   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install Xcode Command Line Tools**:
   ```sh
   xcode-select --install
   ```

3. **Install Python**:
   ```sh
   brew install python
   ```

4. **Install VSCode** from [VSCode website](https://code.visualstudio.com/).

5. **Install VSCode Extensions**:
   - Python
   - Jupyter
   - GitHub Repositories

6. **Set Up a Virtual Environment**:
   ```sh
   mkdir my_project
   cd my_project
   python3 -m venv venv
   source venv/bin/activate
   ```

7. **Install Jupyter Book and ghp-import**:
   ```sh
   python3 -m ensurepip --upgrade
   pip install jupyter-book ghp-import
   ```

8. **Create and Build a Jupyter Book**:
   ```sh
   jb create mybook
   cd mybook
   jb build .
   ```

9. **Publish to GitHub Pages**:
   ```sh
   ghp-import -n -p -f _build/html
   ```

10. **Increase Git Buffer Size and Retry Push**:
    ```sh
    git config --global http.postBuffer 524288000
    git add .
    git commit -m "workflow 3.0 (ghp-import from any computer)"
    git push origin main
    ```

11. **SSL Configuration (if needed)**:
    ```sh
    git config --global http.sslVerify false
    git push origin main
    git config --global http.sslVerify true
    ```

By following these updated steps, you should be able to resolve any issues related to dependencies and successfully push your changes to GitHub. If you encounter any further issues, let me know!

## 34 `pip install pandas ... `

Certainly! You can install the most frequently used Python packages for data analysis in a single command using `pip`. Here is the command to install packages like `numpy`, `pandas`, `matplotlib`, `seaborn`, `scipy`, `scikit-learn`, and `jupyter`:

```sh
pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyter
```

If you want to ensure these packages are installed in your virtual environment, make sure the environment is activated before running the command:

1. **Activate your virtual environment**:
   ```sh
   source venv/bin/activate
   ```

2. **Install the packages**:
   ```sh
   pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyter
   ```

This will install the most commonly used packages for data analysis in one go. If you need any additional packages, you can always add them to the list.

```bash
# this takes about an hour

# and this took half hour
brew install python
```

## 35 `install.packages('IRkernel')`

To run R commands, you need to install & open the R console from your terminal. Here are the steps:

1. **Open the R Console**:
   In your terminal, type `R` and press Enter to start the R console.

2. **Install and Configure `IRkernel`**:
   Once in the R console, run the following commands:
   ```R
   install.packages('IRkernel')
   IRkernel::installspec(user = FALSE)
   ```

Here is the complete workflow:

1. **Open R Console**:
   ```bash
   R
   ```

2. **Run R Commands**:
   ```R
   install.packages('IRkernel')
   IRkernel::installspec(user = FALSE)
   ```

After completing these steps, you can exit the R console by typing `q()` and pressing Enter.

3. **Verify Kernel Installation**:
   After configuring the kernel, verify its installation in your terminal:
   ```bash
   jupyter kernelspec list
   ```

This should list the available kernels, including `ir`.

### Example:

**Opening the R Console:**
```bash
(venv) apollo@Athena Athena % R
```

**Installing and Configuring `IRkernel`:**
```R
> install.packages('IRkernel')
> IRkernel::installspec(user = FALSE)
> q()
```

After running these commands, your R kernel should be installed and configured correctly. Then, you can retry building your Jupyter Book.

For VSCode:
1. Extensions
2. Search for `@id:reditorsupport.r`
3. Install `R Extension for visual Studio Code`

## 36 `IRkernel` (take 2)

Here's a summary of the steps to configure the R kernel in VSCode on your MacBook Pro for easy reproducibility:

### Summary of Steps

1. **Install Prerequisites**
    - Install Python 3 (if not already installed).
    - Install R from [CRAN](https://cran.r-project.org/mirrors.html).

2. **Set Up a Virtual Environment**
    ```bash
    python3 -m venv ~/documents/athena/myenv
    source ~/documents/athena/myenv/bin/activate
    ```

3. **Install Jupyter and IRKernel**
    ```bash
    pip install jupyter jupyter-client
    R -e "install.packages('IRkernel'); IRkernel::installspec(user = TRUE)"
    ```

4. **Ensure R is in the PATH**
    - Add R to your PATH in your `~/.zshrc`:
    ```bash
    echo 'export PATH=/usr/local/bin:$PATH' >> ~/.zshrc
    source ~/.zshrc
    ```

5. **Verify Kernel Installation**
    ```bash
    jupyter kernelspec list
    ```

6. **Install VSCode Extensions**
    - Install the [Jupyter Extension](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter).
    - Optionally, install the [R Extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=REditorSupport.r).

7. **Configure VSCode to Use the Jupyter Server**
    - Open VSCode.
    - Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`).
    - Run `Jupyter: Select Interpreter to start Jupyter server` and select the Python interpreter from your virtual environment.

8. **Open or Create a Jupyter Notebook**
    - Open an existing `.ipynb` file or create a new one.

9. **Select the R Kernel**
    - Click on the kernel name at the top right corner of the notebook interface.
    - Select the `R` kernel from the dropdown menu.
    - If none apparently available, select `Jupyter Kernel` and `R` will be an option

### Step-by-Step Commands

1. **Install Python 3 and R (if not already installed)**:
    - Follow the installation instructions for Python 3 and R on your MacBook Pro.

2. **Set Up Virtual Environment**:
    ```bash
    python3 -m venv ~/documents/athena/myenv
    source ~/documents/athena/myenv/bin/activate
    ```

3. **Install Jupyter and IRKernel**:
    ```bash
    pip install jupyter jupyter-client
    R -e "install.packages('IRkernel'); IRkernel::installspec(user = TRUE)"
    ```

4. **Add R to PATH**:
    ```bash
    echo 'export PATH=/usr/local/bin:$PATH' >> ~/.zshrc
    source ~/.zshrc
    ```

5. **Verify Kernel Installation**:
    ```bash
    jupyter kernelspec list
    ```

6. **Install VSCode Extensions**:
    - Open the Extensions view in VSCode (`Ctrl+Shift+X` or `Cmd+Shift+X`).
    - Install the Jupyter Extension.
    - Optionally, install the R Extension.

7. **Configure Jupyter Server in VSCode**:
    - Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`).
    - Run `Jupyter: Select Interpreter to start Jupyter server` and select the Python interpreter from your virtual environment.

8. **Open or Create a Jupyter Notebook**:
    - Open an existing `.ipynb` file or create a new one in VSCode.

9. **Select the R Kernel**:
    - Click on the kernel name at the top right corner of the notebook interface.
    - Select the `R` kernel from the dropdown menu.

By following these summarized steps, you should be able to replicate the setup on your MacBook Pro successfully. If you encounter any issues or need further assistance, please let me know.

## 37 `jb create`

```bash
source ~/documents/athena/myenv/bin/activate
pip install jupyter-book
jb --version
jb create ir
```
## 38 `ghp-import`

To install `ghp-import`, you can use `pip`, the Python package installer. Here's how you can do it:

1. Open your terminal.

2. Ensure you have `pip` installed by running:

   ```sh
   pip --version
   ```

   If `pip` is not installed, you can install it by following the instructions [here](https://pip.pypa.io/en/stable/installation/).

3. Install `ghp-import` using `pip`:

   ```sh
   pip install ghp-import
   ```

If you want to install `ghp-import` in a specific virtual environment, make sure to activate your virtual environment first:

1. Navigate to your project directory:

   ```sh
   cd /path/to/your/project
   ```

2. Activate your virtual environment. For example, if you are using `venv`, you can activate it like this:

   - On macOS and Linux:

     ```sh
     source venv/bin/activate
     ```

   - On Windows:

     ```sh
     .\venv\Scripts\activate
     ```

3. Once the virtual environment is activated, install `ghp-import`:

   ```sh
   pip install ghp-import
   ```

To verify the installation, you can check the version of `ghp-import`:

```sh
ghp-import --version
```

This should confirm that `ghp-import` is installed and ready to use. If you encounter any issues, feel free to ask for further assistance.

## 39 setup_stata_kernel.sh

Certainly! Below is a comprehensive script that you can run from start to finish to set up your virtual environment, install the necessary packages, configure the Stata kernel, and start Jupyter Notebook. This script ensures that the entire process is reproducible and hassle-free.

### Complete Script for Setting Up Stata Kernel in Jupyter Notebook

```sh
#!/bin/bash

# Set up variables
VENV_PATH="/Users/hades/Documents/hades/myenv"
STATA_PATH="/Applications/Stata/StataMP.app/Contents/MacOS/stata-mp"
ZSHRC_PATH="$HOME/.zshrc"

# Step 1: Remove existing virtual environment if it exists
echo "Removing existing virtual environment..."
rm -rf $VENV_PATH

# Step 2: Create a new virtual environment
echo "Creating a new virtual environment..."
python3 -m venv $VENV_PATH

# Step 3: Activate the virtual environment
echo "Activating the virtual environment..."
source $VENV_PATH/bin/activate

# Step 4: Install necessary packages
echo "Installing necessary packages..."
pip install stata_kernel ipykernel setuptools notebook

# Step 5: Configure environment variable for Stata path
echo "Configuring environment variable for Stata path..."
if [ ! -f $ZSHRC_PATH ]; then
    touch $ZSHRC_PATH
fi
grep -qxF "export STATA_KERNEL_STATA_PATH=$STATA_PATH" $ZSHRC_PATH || echo "export STATA_KERNEL_STATA_PATH=$STATA_PATH" >> $ZSHRC_PATH
source $ZSHRC_PATH

# Step 6: Install the Stata kernel
echo "Installing the Stata kernel..."
python -m stata_kernel.install

# Step 7: Start Jupyter Notebook
echo "Starting Jupyter Notebook..."
jupyter notebook

echo "Setup complete!"
```

### Instructions to Use the Script

1. **Save the Script**:
   - Save the script to a file, for example, `setup_stata_kernel.sh`.

2. **Make the Script Executable**:
   - Make the script executable by running the following command:

   ```sh
   chmod +x setup_stata_kernel.sh
   ```

3. **Run the Script**:
   - Run the script by executing:

   ```sh
   ./setup_stata_kernel.sh
   ```

This script will handle the entire process of setting up the virtual environment, installing the necessary packages, configuring the environment variable, installing the Stata kernel, and starting Jupyter Notebook. This ensures a reproducible and smooth setup experience.

## 40 `XQuartz`

Needed to install it for either [R](https://abikesa.github.io/kernel/intro.html) or [Stata](https://abikesa.github.io/pystata/intro.html), I forget which

## 41 `setup_myenv.sh`

```sh
#!/bin/zsh

# Define the environment directory
ENV_DIR="myenv"

# Remove the existing environment if it exists
if [ -d "$ENV_DIR" ]; then
    rm -rf "$ENV_DIR"
fi

# Create a new virtual environment
python3 -m venv $ENV_DIR

# Check if virtual environment was created successfully
if [ ! -d "$ENV_DIR" ]; then
    echo "Failed to create virtual environment."
    exit 1
fi

# Activate the virtual environment
source $ENV_DIR/bin/activate

# Check if activation was successful
if [ "$VIRTUAL_ENV" != "" ]; then
    echo "Virtual environment activated."
else
    echo "Failed to activate virtual environment."
    exit 1
fi

# Ensure pip is installed
if ! command -v pip &> /dev/null; then
    echo "pip could not be found, installing pip..."
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    python get-pip.py
    if [ $? -ne 0 ]; then
        echo "Failed to install pip."
        deactivate
        exit 1
    fi
fi

# Upgrade pip
pip install --upgrade pip

# Install necessary Python packages
pip install jupyter jupyterlab ipykernel jupyter-book ghp-import matplotlib numpy wordcloud pandas scipy seaborn scikit-learn tensorflow keras plotly dash flask django pytest pylint black isort mypy pydantic fastapi uvicorn requests beautifulsoup4 sqlalchemy ipython tqdm joblib sympy h5py pyqt5 nbconvert nbformat ipywidgets

# Check if Jupyter Book is installed
if ! command -v jb &> /dev/null; then
    echo "Jupyter Book could not be found. Something went wrong with the installation."
    deactivate
    exit 1
fi

# Install IRkernel for R
Rscript -e "install.packages('IRkernel', repos='https://cloud.r-project.org/')"
if [ $? -ne 0 ]; then
    echo "Failed to install IRkernel."
    deactivate
    exit 1
fi
Rscript -e "IRkernel::installspec(name = 'ir', displayname = 'R')"
if [ $? -ne 0 ]; then
    echo "Failed to install IRkernel spec."
    deactivate
    exit 1
fi

# Install Stata kernel (assuming you have Stata installed)
pip install stata_kernel
python -m stata_kernel.install
if [ $? -ne 0 ]; then
    echo "Failed to install Stata kernel."
    deactivate
    exit 1
fi

echo "Environment setup complete."

# Deactivate the virtual environment
deactivate

# Print instructions for reactivating the virtual environment
echo "To reactivate the virtual environment, use the following command:"
echo "source $ENV_DIR/bin/activate"

```

## 42 `git config --global http.sslVerify false`

Alright, let's switch back to using HTTPS for your remote repository. Here’s how to resolve the issue using HTTPS:

1. **Set Remote URL Back to HTTPS**:
   ```sh
   git remote set-url origin https://github.com/username/repo.git
   ```

2. **Retry the Push**:
   ```sh
   git push -u origin main
   ```

3. **If You Encounter the SSL Error Again**:
   Since we increased the HTTP post buffer and switched to HTTP/1.1, we can try a few more steps to resolve the SSL issue.

   - **Disable SSL Verification**:
     This is not recommended for regular use due to security risks, but it can help diagnose the problem:
     ```sh
     git config --global http.sslVerify false
     ```
     Then try the push again.

   - **Check GitHub Status**:
     Make sure GitHub is not experiencing any issues by checking [GitHub Status](https://www.githubstatus.com/).

   - **Reauthenticate**:
     Sometimes, clearing saved credentials can help:
     ```sh
     git credential-cache exit
     ```
     Then try the push again. You might be prompted to enter your username and password.

   - **Update Git**:
     Ensure your Git version is up-to-date. You can download the latest version from [Git's official website](https://git-scm.com/).

If the push is successful, you can turn SSL verification back on for security reasons:
```sh
git config --global http.sslVerify true
```

Try these steps, and let me know if you encounter any specific errors during the process.

## 43 `pip install "sphinx>=5.0,<6.0"`

We are caught in a [dependency conflict](https://abikesa.github.io/wikitemplate/) due to the required versions of sphinx and other extensions. Let’s take a different approach:

1. Uninstall all currently installed versions.
2. Install compatible versions of the packages together, ensuring there are no version conflicts.

Let’s uninstall the current packages again:

```sh
pip uninstall sphinx sphinx-panels jupyter-book sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design sphinx-external-toc pydata-sphinx-theme sphinx-jupyterbook-latex sphinx-book-theme myst-nb myst-parser
```

Next, install the correct versions together:

```
pip install "sphinx>=5.0,<6.0" sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design "sphinx-external-toc<2" "pydata-sphinx-theme>=0.15.2" "sphinx-jupyterbook-latex<2" "sphinx-book-theme>=1.1.0,<2" "myst-nb<3" "myst-parser<3" "jupyter-book>=0.12.0" "sphinx-thebe>=0.3,<1" sphinx-panels
```

This command should install compatible versions of the required packages. If there are any issues during installation, we will need to adjust the versions accordingly. Once installed, try building your Jupyter Book again:

```sh
jb build nia/book/website
```

If further adjustments are needed, please let me know.

## 44

### `pip install -r requirements.txt && jupyter-book build .`

#### Deploying

[First](https://abikesa.github.io/worthy/intro.html), create a work directory & environment

```sh
git clone https://github.com/abikesa/workflow && mv workflow new && new/setup_myenv.sh && source myenv/bin/activate 
```

#### On Netlify

Next, navigate to your project template:

```sh
git clone https://github.com/the-turing-way/the-turing-way
mv the-turing-way project
```

- Publish directory: `project/book/website/_build/html`
- Base directory: `book/website`
- Build command: `pip install -r requirements.txt && jupyter-book build .`
- This will give an error, but after installing key files. So just ignore & follow these instructions:

```sh
pip uninstall sphinx sphinx-panels jupyter-book sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design sphinx-external-toc pydata-sphinx-theme sphinx-jupyterbook-latex sphinx-book-theme myst-nb myst-parser
pip install "sphinx>=5.0,<6.0" sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design "sphinx-external-toc<2" "pydata-sphinx-theme>=0.15.2" "sphinx-jupyterbook-latex<2" "sphinx-book-theme>=1.1.0,<2" "myst-nb<3" "myst-parser<3" "jupyter-book>=0.12.0" "sphinx-thebe>=0.3,<1" sphinx-panels
jupyter-book build .
```

###  `git credential-cache exit`


```sh
git credential-cache exit
cd home/book/website
git add ./*
```


## 45 `wiki-style jb`

```sh
# myenv
git clone https://github.com/abikesa/workflow && mv workflow new && new/setup_myenv.sh && source myenv/bin/activate 

# template
git clone https://github.com/the-turing-way/the-turing-way
mv the-turing-way kenny
cd kenny/book/website
pip install -r requirements.txt && jupyter-book build .

# error message
pip uninstall sphinx sphinx-panels jupyter-book sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design sphinx-external-toc pydata-sphinx-theme sphinx-jupyterbook-latex sphinx-book-theme myst-nb myst-parser
pip install "sphinx>=5.0,<6.0" sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design "sphinx-external-toc<2" "pydata-sphinx-theme>=0.15.2" "sphinx-jupyterbook-latex<2" "sphinx-book-theme>=1.1.0,<2" "myst-nb<3" "myst-parser<3" "jupyter-book>=0.12.0" "sphinx-thebe>=0.3,<1" sphinx-panels
jupyter-book build .

# big files
rm -rf figures
rm -rf _build/html/_images

# parent directory
cd ~/documents/hades

# block ghp-import (use netlify or import manually)
new/jbb_https.sh
```

## 46 `everyday`

Create & destroy, everyday! That should be our motto!!

#### 1 `myenv`

```sh
git clone https://github.com/abikesa/workflow && mv workflow new && new/setup_myenv.sh && source myenv/bin/activate 

```

#### 2 `clone` & `rm`

```sh
git clone https://github.com/the-turing-way/the-turing-way
mv the-turing-way cnd
```

#### 3 `_toc.yml`

Keep only 3 sections as template for structure

```sh
cat cnd/book/website/_toc.yml
```

- Welcome
   - index.md
- Forward
   - forward.md
   - background.md
- Guide for Reproducible Research
   - reproducible-reseaarch.md
   - overview-definitions.md
- Guide for Project Design
   - project-design.md
   - pd-overview.md
   - pd-overview
      - pd-overview-methods


#### 4 `requirements.txt`

This gives an error on `Athena` & `Hades`, but not on `Poseidon`

```sh
cd cnd/book/website
pip install -r requirements.txt && jupyter-book build .
```

And here's the fix for `Athena` & `Poseidon`

```sh
pip uninstall sphinx sphinx-panels jupyter-book sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design sphinx-external-toc pydata-sphinx-theme sphinx-jupyterbook-latex sphinx-book-theme myst-nb myst-parser
pip install "sphinx>=5.0,<6.0" sphinxcontrib-applehelp sphinxcontrib-devhelp sphinxcontrib-jsmath sphinxcontrib-htmlhelp sphinxcontrib-serializinghtml sphinxcontrib-qthelp sphinx-design "sphinx-external-toc<2" "pydata-sphinx-theme>=0.15.2" "sphinx-jupyterbook-latex<2" "sphinx-book-theme>=1.1.0,<2" "myst-nb<3" "myst-parser<3" "jupyter-book>=0.12.0" "sphinx-thebe>=0.3,<1" sphinx-panels
```

```sh
jupyter-book build .
```

#### 5 `rm`

Can skip this since step `#3` sanized the repo

```sh
rm -rf figures
rm -rf _build/html/_images
```

#### 6 `new/jbb_https.sh`

Prepare to `push` the `.git`  

```sh
cd ~/documents/athena
new/jbb_https.sh
```

#### 7 `ghp-import`

```sh
git clone https://github.com/abikesa/everyday/
mv cnd/book/website/_build everyday/book/website/_build
```
