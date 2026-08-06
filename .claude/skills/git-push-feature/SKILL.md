# git-push-feature

When the user wants to push feature changes with a custom commit message, this skill stages all changes, pulls latest from remote, creates a commit with a meaningful message combining user-provided short description and analysis of changes, then pushes to the current branch. Use this skill when the user says "/git-push-feature <short description>" to streamline the git workflow.

## Steps

1. **Check for staged changes**: Run `git diff --staged` to see what's already staged.
2. **Stage all changes**: Run `git add .` to stage all modifications, new files, and deletions.
3. **Pull latest changes**: Run `git pull` to ensure local branch is up-to-date with remote.
4. **Generate commit message**: 
   - Take the user-provided short description (the argument after `/git-push-feature`)
   - Analyze the staged changes to understand what was modified (e.g., list files changed, types of changes)
   - Combine into a conventional commit message format: `<type>(<scope>): <short description>` where type is inferred from changes (e.g., feat, fix, chore, etc.) and scope is optional.
   - If unsure, use `chore: <short description>` as fallback.
5. **Commit changes**: Run `git commit -m "<generated message>"`.
6. **Push changes**: Run `git push` to push the commit to the remote repository.
7. **Report summary**: Output a brief summary to the user showing what was done, including the commit message used and push result.

## Details

- The skill assumes the current branch is the feature branch to push.
- It will stage all changes (including new and deleted files). If you want to stage only specific changes, use git commands directly before invoking this skill.
- The commit message generation attempts to infer the type of change:
  - If changes include new feature files or additions: `feat`
  - If changes include bug fixes: `fix`
  - If changes include documentation: `docs`
  - If changes include refactoring: `refactor`
  - If changes include test additions/modifications: `test`
  - If changes include configuration, build scripts, or chores: `chore`
  - Otherwise defaults to `chore`.
- The scope is omitted for simplicity; you can modify the skill to include a scope if desired.
- After pushing, the skill will output the commit hash, branch name, and a summary of files changed.

## Example

User input: `/git-push-feature add user login form`

Skill actions:
- Stages all changes
- Pulls latest from origin
- Generates commit message: `feat: add user login form` (if changes indicate new feature)
- Commits with that message
- Pushes to remote
- Outputs: `Successfully pushed commit abc123 to branch feature-login. Commit message: feat: add user login form`
