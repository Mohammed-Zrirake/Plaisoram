# Git Configuration & Contribution Problems

This guide explains how to fix issues where your commits are not appearing as "contributions" (green squares) on your GitHub profile.

## The Problem
GitHub only counts contributions if the email address used in the Git commit matches a **verified email** on your GitHub account. 

If you have the **"Keep my email addresses private"** setting enabled in your GitHub Email Settings, GitHub may ignore commits made with your personal email and require you to use your official **GitHub No-Reply Email**.

## The Fix: Setting your Identity Correctly

To ensure all future commits are attributed to you, run these commands globally:

```bash
# Set your display name
git config --global user.name "Mohammed-Zrirake"

# Set your official GitHub private email
git config --global user.email "136124921+Mohammed-Zrirake@users.noreply.github.com"
```

## How to Repair "Missing" Squares (Retroactive Fix)

If you made commits today that aren't showing up, you can "repair" the authorship of the most recent commit like this:

```bash
# 1. Update the author of the last commit
git commit --amend --author="Mohammed-Zrirake <136124921+Mohammed-Zrirake@users.noreply.github.com>" --no-edit

# 2. Force push the change to GitHub
git push --force
```

### Why do I have multiple branches?
If you see branches like `release-please--...`, these are created by automated release bots. If you prefer a single-branch workflow, you should remove the `release-please` configuration files (`release-please-config.json`) and stick to a simpler semantic release workflow that pushes directly to your main branch.

---
*Generated as a reference for the Plaisoram Project.*
