# Comprehensive Guide to Managing Multiple GitHub Accounts

This guide will help you set up your development environment to work seamlessly with multiple GitHub accounts (e.g., personal and work), avoiding authentication headaches and ensuring your commits are properly attributed.

## The Problem

When working with multiple GitHub accounts, you face several challenges:

1. **Authentication**: GitHub needs to know which account you're using when you clone, pull, or push code.
2. **Author Attribution**: Your commits need to be attributed to the correct user name and email for each account.
3. **CLI Operations**: When using the GitHub CLI, you need an easy way to switch between accounts.

These challenges can lead to frustrating workflows:

- Constantly entering credentials
- Having to remember which account to use for which repository
- Commits being attributed to the wrong identity
- Needing to manually configure each repository

## Part 1: Git Credential Manager (GCM) Setup

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

## Part 2: Automatic Author Attribution Setup

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
username1.token=ghp_your_first_account_token
username2.name=Second Account Name
username2.email=second@example.com
username2.token=ghp_your_second_account_token
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

## Part 3: GitHub CLI Multi-Account Setup

The GitHub CLI (`gh`) is a powerful tool for interacting with GitHub from the command line. Here's how to set it up for multiple accounts using the same users.txt file we've already created.

### Prerequisites

- GitHub CLI installed (`brew install gh` on macOS)
- The users.txt file already set up with tokens (as shown in Part 2)

### How to Get Personal Access Tokens

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Create a new token with the necessary permissions (typically repo, read:org, and workflow)
3. Add the token to your users.txt file as shown in Part 2

### Create a Custom `gh` Function

Add the following function to your shell configuration file (`~/.bashrc` or `~/.zshrc`):

```bash
# Override the gh command to support multiple accounts
gh() {
  # Path to the users configuration file
  USERS_FILE="$HOME/.git-config/users.txt"

  # Check if first argument is provided and exists in users.txt
  if [ -n "$1" ] && grep -q "^$1\.token=" "$USERS_FILE"; then
    # Extract the username and remove it from the arguments
    GITHUB_USERNAME="$1"
    shift

    # Get the authentication token for the specified username
    TOKEN=$(grep "^$GITHUB_USERNAME.token=" "$USERS_FILE" | cut -d'=' -f2)

    if [ -z "$TOKEN" ]; then
      echo "Error: No token found for username '$GITHUB_USERNAME' in $USERS_FILE"
      return 1
    fi

    echo "Using token for $GITHUB_USERNAME"
    # Use the token for this gh command
    GH_TOKEN="$TOKEN" command gh "$@"
  else
    # If first arg isn't a username in our file, use gh normally
    command gh "$@"
  fi
}
```

After adding the function to your shell configuration file, reload it:

```bash
source ~/.bashrc  # or source ~/.zshrc
```

### Usage with GitHub CLI

The function allows you to use the `gh` command in two ways:

1. **With a specific GitHub account**

```bash
gh username1 repo list
```

This will use the token for "username1" from your users.txt file.

2. **Normal `gh` usage**

```bash
gh repo list
```

This will use the default behavior (whoever is currently logged in with `gh auth login`).

### Examples

```bash
# Create a new repository as a specific user
gh username1 repo create --public my-new-repo

# List issues for a specific repository as another user
gh username2 issue list --repo username2/some-repo

# Create a pull request as a specific user
gh username1 pr create --title "New feature" --body "Description"
```

## Part 4: Transferring Repositories Between Accounts

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

### GitHub CLI Issues

If you see an error like:

```
Error: No token found for username 'username1' in /path/to/users.txt
```

Check that:

1. The username is spelled correctly in both your command and users.txt
2. The token entry exists in the format `username1.token=ghp_token`
3. There are no spaces before or after the values
4. The users.txt file is in the correct location

If the command seems to ignore your username and uses the default account:

1. Make sure you reloaded your shell configuration
2. Check that the grep command is finding your username in the file:
   ```bash
   grep "^username1\.token=" "$HOME/.git-config/users.txt"
   ```
3. Try adding `set -x` before the function in your shell config to debug

## Security Notes

- Personal access tokens should be treated like passwords
- Consider using fine-grained tokens with the minimum necessary permissions
- Review and rotate your tokens regularly

---

With this setup, you should be able to seamlessly work with multiple GitHub accounts without constantly dealing with authentication or attribution issues, using both Git and the GitHub CLI.
