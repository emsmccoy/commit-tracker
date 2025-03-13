# Commit-Tracker
Tired of working in forked repositories or non-main branches without having your efforts recognized by GitHub? Commit-Tracker is here to help!

# Purpose
Commit-Tracker automates the process of logging your commits from forked repositories or non-main branches to a dedicated tracking repository. This ensures that all your contributions are counted towards your GitHub contribution graph.

# Setup Instructions
## Step 1: Create a Tracking Repository
1. Create a new GitHub repository named `commit-tracker`.
2. Clone this repository to your local machine:
```
bash
git clone https://github.com/your-username/commit-tracker.git
```

## Step 2: Create a Setup Script
1. Create a Bash script named `setup-githook.sh`.
2. Copy the following code into the script. **Before running the script, make sure to replace the following lines with the actual path to your local `commit-tracker` repository**:

- Line 4: `TRACKER_REPO_PATH="path/to/your/tracker/repo"`

- Line 21 (inside the hook): `cd path/to/your/tracker/repo`

Here is the script:

```
bash
#!/bin/bash

# Define the path to the commit tracker repository
TRACKER_REPO_PATH="path/to/your/tracker/repo"

# Check if the current branch is not main or if the repo is a fork
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
IS_FORK=$(git remote -v | grep -q "origin.*fork")

# Create the post-commit hook if it doesn't exist
HOOK_PATH=".git/hooks/post-commit"
if [ ! -f "$HOOK_PATH" ]; then
  echo "Creating post-commit hook..."

  cat > "$HOOK_PATH" <<EOF
  #!/bin/sh

  # Extract commit metadata
  COMMIT_HASH=\$(git rev-parse HEAD)
  COMMIT_DATE=\$(git log -1 --format=%cd --date=iso)
  REPO_NAME=\$(basename -s .git \$(git remote get-url origin))
  CURRENT_BRANCH=\$(git rev-parse --abbrev-ref HEAD)

  # Check if current branch is not main or if repo is a fork
  IS_FORK=\$(git remote -v | grep -q "origin.*fork")

  if [ "\$CURRENT_BRANCH" != "main" ] || [ "\$IS_FORK" = "true" ]; then
    # Navigate to the tracking repository
    cd path/to/your/tracker/repo

    # Append commit data to a log file
    echo "\$COMMIT_DATE | \$REPO_NAME | \$CURRENT_BRANCH | \$COMMIT_HASH" >> commits.log

    # Stage and commit the updated log file
    git add commits.log
    git commit -m "Track commit from \$REPO_NAME on branch \$CURRENT_BRANCH"

    # Push the commit to the tracking repository
    git push origin main

    # Return to the original repository
    cd -
  fi
EOF
 chmod +x "$HOOK_PATH"
 echo "Post-commit hook created successfully."
else
echo "Post-commit hook already exists."
fi
```

### Step 3: Simplify Running the Script (Optional)
To simplify running your script, you can define a Bash function:
1. Open your Bash configuration file:
```bash
nano ~/.bashrc
```
Add the following function:
```
bash
start_tracker() {
  /path/to/setup_hook.sh
}
```
Apply changes:

- Save and close the file.

- Reload your Bash configuration:
```
bash
source ~/.bashrc
```
### Step 4: Test Commit-Tracker
1. Clone a forked repository.

2. Navigate into the cloned repository.

3. Run the `start_tracker` command (if you defined the function) or execute the setup-githook.sh script directly.

4. Make a small change and commit it:
```
bash
git checkout -b test-branch
touch test.txt
git add test.txt
git commit -m "Test commit"
```
5. Check your `commit-tracker` repository to see if the commit was logged successfully.

By following these steps, you'll be able to track all your contributions from forked repositories or non-main branches in a dedicated tracking repository.
