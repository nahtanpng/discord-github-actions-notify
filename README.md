# GitHub Actions + Discord Webhook

Este repositório mostra como integrar **GitHub Actions** com o **Discord**, enviando notificações automáticas para um canal sempre que eventos importantes acontecem no repositório, como:

- Push nas branches principais (`main`, `homolog`, etc)
- Abertura de Pull Requests
- Novos reviewers atribuídos

Tudo isso usando **Webhooks do Discord** e **workflows simples em YAML**.

---

## Pré-requisitos

1. Ter um servidor no **Discord**
2. Criar um **Webhook** no canal que deseja receber as notificações  
   *(Configurações do Canal → Integrações → Webhooks → Novo Webhook)*
3. Copiar a URL do webhook
4. Salvar essa URL em um secret do seu repositório do Github  
   *(Repositório → Settings → Secrets → Actions → New repository secret)*

---

## Exemplo: Notificação de Push
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
          MESSAGE="🚀 Push detectado em **$REPO**!\n📌 Branch: \`$BRANCH\`\n👤 Autor: $AUTHOR\n🔗 [Ver commit]($COMMIT_URL)"

          curl -H "Content-Type: application/json" \
               -X POST \
               -d "{\"content\": \"$MESSAGE\"}" \
               ${{ secrets.DISCORD_WEBHOOK }}
```

### Demonstração:
<img width="539" height="143" alt="image" src="https://github.com/user-attachments/assets/b7181372-b5d9-4ae1-824f-6f28e1a0a2cb" />

## Exemplo: Notificação de Pull Request

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

          MESSAGE="📥 Novo PR aberto em **$REPO**!\n📌 Branch: \`$BRANCH\`\n📝 Título: $PR_TITLE\n👤 Autor: $PR_AUTHOR\n🔗 [Abrir PR]($PR_URL)"

          curl -H "Content-Type: application/json" \
               -X POST \
               -d "{\"content\": \"$MESSAGE\"}" \
               ${{ secrets.DISCORD_WEBHOOK }}
```

### Demonstração:
<img width="452" height="143" alt="image" src="https://github.com/user-attachments/assets/a8654870-f845-4888-9866-5fe2e95ec3bb" />

## Estilizações

Uma forma para estilizar esse aviso é utilizar `embeds` do Discord.

## Exemplo: Notificação de push
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
                title: (":rocket: Push detectado em `" + $repo + "`! Atualizem suas branchs."),
                description: (":pushpin: **Branch:** `" + $branch + "`\n:pencil: **Autor:** " + $author + "\n:link: [Ver Commit](" + $url + ")"),
                color: 5814783,
              }
            ]
          }' \
          | curl -H "Content-Type: application/json" \
                 -X POST \
                 -d @- \
                 ${{ secrets.DISCORD_WEBHOOK }}
```

### Demonstração:
<img width="740" height="219" alt="image" src="https://github.com/user-attachments/assets/0deb8ac6-9500-4c9c-8698-1dae2ed946dc" />

## Exemplo: Notificação de Pull Request

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
      
          # Pega todos os reviewers pedidos
          REVIEWERS=$(jq -r '.pull_request.requested_reviewers | map(.login) | join(", ")' "$GITHUB_EVENT_PATH")
          # Se não tiver nenhum, coloca "Nenhum"
          if [ -z "$REVIEWERS" ]; then
            REVIEWERS="Nenhum"
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
                  title: ("📥 Novo PR aberto em `" + $repo + "` para a branch `" + $branch + "`"),
                  description: ("📝 **Título:** " + $title + "\n👤 **Autor:** " + $author + "\n👀 **Reviewers:** " + $reviewers + "\n🔗 [Abrir PR](" + $url + ")"),
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

### Demonstração:
<img width="656" height="269" alt="image" src="https://github.com/user-attachments/assets/bce21a1c-4860-46db-8ee4-a227cff5a888" />

