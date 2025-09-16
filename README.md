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

🎯 Objetivo

O objetivo deste repositório é servir de guia prático para qualquer desenvolvedor que queira melhorar a comunicação do time, recebendo notificações em tempo real no Discord sempre que algo importante acontecer no repositório.
