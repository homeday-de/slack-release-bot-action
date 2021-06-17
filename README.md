# Slack release BOT

Announce project releases into your Slack channels, like the one below:

![Slack release BOT example](https://user-images.githubusercontent.com/1734873/122429377-066ed100-cf93-11eb-9665-efc21214b880.png)

## Usage

### Pre-requisites

Add a Slack webhook URL into your secrets (We, Homeday, have an application in Slack called Release BOT which has all the webhooks).

### Inputs

- `webhook_url` (required): the Slack webhook URL that will be in your secrets.
- `title` (required): the bold title to be used in the top of the message. From the example above: `v17.0.3 (2021-06-01)`.
- `body` (required): the message that describes the release. It will use Slack markdown so you might need to Slackify it first, see the examples below.
- `context` (required): the application that is being released. From the example above: `Homeday Blocks`.

### Example

- Simple

```yml
jobs:
  my-slack-job:
    runs-on: ubuntu-latest
    steps:
      - uses: homeday-de/slack-release-bot-action@main
        with:
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
          title: 🚀 v1.0.0
          body: This major release means that we are live!
          context: Slack release BOT
```

- Latest commit info

```yml
name: Announce the release on Slack

on:
  push:
    branches: [main]
    
jobs:
  slack:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2.3.4
      - name: Get latest commit info
        run: |
          echo "::set-output name=TITLE::$(git show -1 --format='%s' -s)"
          echo "::set-output name=BODY::$(git show -1 --format='%b' -s)"
        id: latest_commit
      - uses: homeday-de/slack-release-bot-action@main
        with:
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
          title: ${{ steps.latest_commit.outputs.TITLE }}
          body: ${{ steps.latest_commit.outputs.BODY }}
          context: Website
```

- GitHub release Slackified (used by homeday-de/homeday-blocks)

```yml
name: Announce the release on Slack

on:
  release:
    types: [created]

jobs:
  slack:
    runs-on: ubuntu-latest
    steps:
      - name: Slackify the release body
        id: release_body
        uses: LoveToKnow/slackify-markdown-action@v1.0.0
        with:
          text: ${{ github.event.release.body }}
      - uses: homeday-de/slack-release-bot-action@main
        with:
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
          title: ${{ github.event.release.name }}
          body: ${{ steps.release_body.outputs.text }}
          context: Homeday Blocks
```
