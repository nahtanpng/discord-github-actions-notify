
# GitHub Actions + Discord Webhook

This repository shows how to integrate **GitHub Actions** with **Discord**, sending automatic notifications to a channel whenever important events happen in the repository, such as:

- Push to main branches (`main`, `homolog`, etc)
- Opening Pull Requests
- New reviewers assigned

All of this using **Discord Webhooks** and **simple YAML workflows**.

---

## Prerequisites

1. Have a **Discord** server
2. Create a **Webhook** in the channel where you want to receive notifications  
   *(Channel Settings → Integrations → Webhooks → New Webhook)*
3. Copy the webhook URL
4. Save this URL as a secret in your GitHub repository  
   *(Repository → Settings → Secrets → Actions → New repository secret)*

---

## Example: Push Notification
```yaml
name: Discord Notify (Push)

on:
  push:
    branches:
      - main
      - homolog

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send Discord Notification
        run: |
          COMMIT_URL="https://github.com/${{ github.repository }}/commit/${{ github.sha }}"
          AUTHOR="${{ github.actor }}"
          BRANCH="${{ github.ref_name }}"
          REPO="${{ github.repository }}"
          MESSAGE="🚀 Push detected in **$REPO**!\n📌 Branch: \`$BRANCH\`\n👤 Author: $AUTHOR\n🔗 [View commit]($COMMIT_URL)"

          curl -H "Content-Type: application/json" \
               -X POST \
               -d "{\"content\": \"$MESSAGE\"}" \
               ${{ secrets.DISCORD_WEBHOOK }}
```

### Demo:
<img width="539" height="143" alt="image" src="https://github.com/user-attachments/assets/b7181372-b5d9-4ae1-824f-6f28e1a0a2cb" />

## Example: Pull Request Notification

```yaml
name: Discord Notify (PR)

on:
  pull_request:
    types: [opened]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send Discord Notification
        run: |
          PR_URL="${{ github.event.pull_request.html_url }}"
          PR_TITLE="${{ github.event.pull_request.title }}"
          PR_AUTHOR="${{ github.event.pull_request.user.login }}"
          BRANCH="${{ github.event.pull_request.base.ref }}"
          REPO="${{ github.repository }}"

          MESSAGE="📥 New PR opened in **$REPO**!\n📌 Branch: \`$BRANCH\`\n📝 Title: $PR_TITLE\n👤 Author: $PR_AUTHOR\n🔗 [Open PR]($PR_URL)"

          curl -H "Content-Type: application/json" \
               -X POST \
               -d "{\"content\": \"$MESSAGE\"}" \
               ${{ secrets.DISCORD_WEBHOOK }}
```

### Demo:
<img width="452" height="143" alt="image" src="https://github.com/user-attachments/assets/a8654870-f845-4888-9866-5fe2e95ec3bb" />

## Styling

A way to style these notifications is to use Discord `embeds`.

## Example: Push notification
```yml
name: Notify Discord on branch push

on:
  push:
    branches:
      - main
      - homolog

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
    - name: Send Discord notification (with embed + mention)
      run: |
        COMMIT_URL="https://github.com/${{ github.repository }}/commit/${{ github.sha }}"
    
        jq -n \
          --arg branch "${{ github.ref_name }}" \
          --arg author "${{ github.actor }}" \
          --arg url "$COMMIT_URL" \
          --arg repo "${{ github.repository }}" \
          '{
            content: "@here",
            embeds: [
              {
                title: (":rocket: Push detected in `" + $repo + "`! Update your branches."),
                description: (":pushpin: **Branch:** `" + $branch + "`\n:pencil: **Author:** " + $author + "\n:link: [View Commit](" + $url + ")"),
                color: 5814783,
              }
            ]
          }' \
          | curl -H "Content-Type: application/json" \
                 -X POST \
                 -d @- \
                 ${{ secrets.DISCORD_WEBHOOK }}
```

### Demo:
<img width="740" height="219" alt="image" src="https://github.com/user-attachments/assets/0deb8ac6-9500-4c9c-8698-1dae2ed946dc" />

## Example: Pull Request Notification

```yml
name: Notify Discord on PR open

on:
  pull_request:
    branches:
      - main
      - homolog

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send Discord notification (PR opened with reviewers)
        run: |
          PR_URL="${{ github.event.pull_request.html_url }}"
          PR_TITLE="${{ github.event.pull_request.title }}"
          PR_AUTHOR="${{ github.event.pull_request.user.login }}"
          BASE_BRANCH="${{ github.event.pull_request.base.ref }}"
          REPO="${{ github.repository }}"
          REVIEWERS=$(jq -r '.pull_request.requested_reviewers | map(.login) | join(", ")' "$GITHUB_EVENT_PATH")

          if [ -z "$REVIEWERS" ]; then
            REVIEWERS="None"
          fi
      
          jq -n \
            --arg branch "$BASE_BRANCH" \
            --arg author "$PR_AUTHOR" \
            --arg url "$PR_URL" \
            --arg title "$PR_TITLE" \
            --arg repo "$REPO" \
            --arg reviewers "$REVIEWERS" \
            '{
              content: "@here",
              embeds: [
                {
                  title: ("📥 New PR opened in `" + $repo + "` for branch `" + $branch + "`"),
                  description: ("📝 **Title:** " + $title + "\n👤 **Author:** " + $author + "\n👀 **Reviewers:** " + $reviewers + "\n🔗 [Open PR](" + $url + ")"),
                  color: (if $branch == "main" then 3066993 else 15105570 end),
                  footer: { text: $repo }
                }
              ]
            }' \
            | curl -H "Content-Type: application/json" \
                   -X POST \
                   -d @- \
                   ${{ secrets.DISCORD_WEBHOOK }}
```

### Demo:
<img width="656" height="269" alt="image" src="https://github.com/user-attachments/assets/bce21a1c-4860-46db-8ee4-a227cff5a888" />
