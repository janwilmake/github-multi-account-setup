# Managing Multiple GitHub Accounts

This guide will help you set up your development environment to work seamlessly with multiple GitHub accounts, avoiding authentication headaches and ensuring your commits are properly attributed.

## The Problem

When working with multiple GitHub accounts (e.g., personal and work), you face two main challenges:

1. **Authentication**: GitHub needs to know which account you're using when you clone, pull, or push code.
2. **Author Attribution**: Your commits need to be attributed to the correct user name and email for each account.

These challenges can lead to frustrating workflows:

- Constantly entering credentials
- Having to remember which account to use for which repository
- Commits being attributed to the wrong identity
- Needing to manually configure each repository

## Installing and Setting Up Git Credential Manager (GCM)

Git Credential Manager (GCM) helps solve the authentication problem by securely storing your credentials.

### Installation

#### On macOS:

```bash
brew install --cask git-credential-manager
```

#### On Windows:

GCM comes bundled with Git for Windows.

#### On Linux:

Follow the [official installation instructions](https://github.com/GitCredentialManager/git-credential-manager#linux).

### Configuration

1. Check your current credential helpers:

```bash
git config --global --get-all credential.helper
```

2. If you see multiple entries or want to start fresh:

```bash
git config --global --unset-all credential.helper
```

3. Set GCM as your credential helper:

```bash
git config --global credential.helper manager
```

4. Verify the configuration:

```bash
git config --global --get-all credential.helper
```

### Using GCM with Multiple Accounts

1. When you clone a repository:

```bash
git clone https://github.com/owner/repo.git
```

2. GCM will prompt for authentication if needed.

3. For repositories from different accounts, GCM will detect this and prompt for the appropriate credentials.

4. To force reauthentication (e.g., to switch accounts):

```bash
git credential reject host=github.com protocol=https
```

## Ensuring Automatic Setting of Author Using users.txt

To ensure your commits are properly attributed to the correct GitHub identity, we'll set up a system that automatically configures the right user name and email based on the repository owner.

### Setup Files

1. Create a configuration directory and users.txt file:

```bash
mkdir -p ~/.git-config
touch ~/.git-config/users.txt
```

2. Edit the `~/.git-config/users.txt` file with your GitHub identities:

```
default.name=Your Default Name
default.email=default@example.com
username1.name=First Account Name
username1.email=first@example.com
username2.name=Second Account Name
username2.email=second@example.com
# Add more accounts as needed
```

3. Create Git template hooks directory:

```bash
mkdir -p ~/.git-templates/hooks
```

4. Create the post-checkout hook script:

```bash
touch ~/.git-templates/hooks/post-checkout
chmod +x ~/.git-templates/hooks/post-checkout
```

5. Add the following content to the `post-checkout` hook:

```bash
#!/bin/bash

# Path to the users configuration file
USERS_FILE="$HOME/.git-config/users.txt"

# Function to read value from users.txt
get_value() {
    local key=$1
    grep "^$key=" "$USERS_FILE" | cut -d'=' -f2
}

# Get the repository remote URL
REMOTE_URL=$(git config --get remote.origin.url)

# Extract owner from the remote URL
if [[ $REMOTE_URL =~ github\.com[/:]([^/]+) ]]; then
    OWNER="${BASH_REMATCH[1]}"

    # Remove potential .git suffix from username
    OWNER=${OWNER%.git}

    # Check if we have a matching user in our config
    NAME=$(get_value "$OWNER.name")
    EMAIL=$(get_value "$OWNER.email")

    # If no matching user found, use defaults
    if [ -z "$NAME" ]; then
        NAME=$(get_value "default.name")
    fi

    if [ -z "$EMAIL" ]; then
        EMAIL=$(get_value "default.email")
    fi

    # Set the git config for this repository
    if [ -n "$NAME" ]; then
        git config user.name "$NAME"
        echo "Set user.name to $NAME"
    fi

    if [ -n "$EMAIL" ]; then
        git config user.email "$EMAIL"
        echo "Set user.email to $EMAIL"
    fi
else
    echo "Could not extract owner from remote URL: $REMOTE_URL"
    echo "Using default values"

    # Use defaults
    NAME=$(get_value "default.name")
    EMAIL=$(get_value "default.email")

    if [ -n "$NAME" ]; then
        git config user.name "$NAME"
        echo "Set user.name to $NAME"
    fi

    if [ -n "$EMAIL" ]; then
        git config user.email "$EMAIL"
        echo "Set user.email to $EMAIL"
    fi
fi
```

6. Create a script for setting up existing repositories:

```bash
touch ~/.git-config/setup-existing-repos.sh
chmod +x ~/.git-config/setup-existing-repos.sh
```

7. Add the following content to the setup script:

```bash
#!/bin/bash

# Path to the users configuration file
USERS_FILE="$HOME/.git-config/users.txt"

# Function to read value from users.txt
get_value() {
    local key=$1
    grep "^$key=" "$USERS_FILE" | cut -d'=' -f2
}

# Get the repository remote URL
REMOTE_URL=$(git config --get remote.origin.url)

# Extract owner from the remote URL
if [[ $REMOTE_URL =~ github\.com[/:]([^/]+) ]]; then
    OWNER="${BASH_REMATCH[1]}"

    # Remove potential .git suffix from username
    OWNER=${OWNER%.git}

    # Check if we have a matching user in our config
    NAME=$(get_value "$OWNER.name")
    EMAIL=$(get_value "$OWNER.email")

    # If no matching user found, use defaults
    if [ -z "$NAME" ]; then
        NAME=$(get_value "default.name")
    fi

    if [ -z "$EMAIL" ]; then
        EMAIL=$(get_value "default.email")
    fi

    # Set the git config for this repository
    if [ -n "$NAME" ]; then
        git config user.name "$NAME"
        echo "Set user.name to $NAME"
    fi

    if [ -n "$EMAIL" ]; then
        git config user.email "$EMAIL"
        echo "Set user.email to $EMAIL"
    fi
else
    echo "Could not extract owner from remote URL: $REMOTE_URL"
    echo "Using default values"

    # Use defaults
    NAME=$(get_value "default.name")
    EMAIL=$(get_value "default.email")

    if [ -n "$NAME" ]; then
        git config user.name "$NAME"
        echo "Set user.name to $NAME"
    fi

    if [ -n "$EMAIL" ]; then
        git config user.email "$EMAIL"
        echo "Set user.email to $EMAIL"
    fi
fi
```

8. Configure Git to use your template directory:

```bash
git config --global init.templateDir ~/.git-templates
```

### Using the Setup

1. **For new repositories**:

   - When you clone a new repository, the hook will automatically set the correct user.name and user.email based on the repository owner.

2. **For existing repositories**, you have two options:
   - Run the setup script:
     ```bash
     cd /path/to/existing/repo
     ~/.git-config/setup-existing-repos.sh
     ```
   - Or reinitialize the Git hooks:
     ```bash
     cd /path/to/existing/repo
     git init
     ```

## Transferring Repositories Between Accounts

### Option 1: GitHub's Transfer Feature

For repositories you own:

1. Go to the repository settings on GitHub
2. Scroll down to the "Danger Zone"
3. Click "Transfer ownership"
4. Enter the repository name and the new owner's username
5. Follow the prompts to complete the transfer

### Option 2: Manual Transfer via Local Clone

When you don't have direct transfer access:

1. Clone the repository with all branches and tags:

   ```bash
   git clone --mirror git@github.com-account1:username/repo.git
   cd repo.git
   ```

2. Create a new empty repository on the target GitHub account

3. Change the remote URL and push:
   ```bash
   git remote set-url origin git@github.com-account2:newusername/repo.git
   git push --mirror
   ```

This approach preserves all commits, branches, and history while transferring to the new account.

## Troubleshooting

### Authentication Issues

If you're having issues with authentication:

1. Clear your stored credentials:

   ```bash
   git credential reject host=github.com protocol=https
   ```

2. Try cloning with an explicit username:
   ```bash
   git clone https://YOUR_USERNAME@github.com/owner/repo.git
   ```

### Author Attribution Issues

If your commits are being attributed to the wrong user:

1. Check the current configuration for the repository:

   ```bash
   git config user.name
   git config user.email
   ```

2. Ensure your `users.txt` file contains the correct mappings

3. Run the setup script in the repository:

   ```bash
   ~/.git-config/setup-existing-repos.sh
   ```

4. Make sure the repository URL has the correct owner:
   ```bash
   git remote -v
   ```

---

With this setup, you should be able to seamlessly work with multiple GitHub accounts without constantly dealing with authentication or attribution issues.
