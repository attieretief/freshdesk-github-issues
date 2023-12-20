# freshdesk-github-issues
> GitHub Action to convert and synchronise Freshdesk tickets to Github issues and related Github projects.

## Usage

Create a file `.github/workflows/sync-freshdesk.yml` (or any filename) in your repository.

```yml
on:
    workflow_dispatch:
    schedule:
      - cron: "*/30 * * * *"

jobs:
    freshdesk-github-sync:
        runs-on: ubuntu-latest
        name: freshdesk-github-sync
        steps:
            - name: Checkout
              uses: actions/checkout@v3
              with:
                token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
            - name: Run Sync
              id: sync
              uses: attieretief/freshdesk-github-issues@v1
              with:
                token: ${{ secrets.GH_API_KEY }}
                freshdesk_key: ${{ secrets.FRESHDESK_API_KEY }}
                freshdesk_domain: linc.co.za
                freshdesk_tag: DEV
                github_project_number: 10
                github_priority_field: Priority
                github_status_field: Status
                github_company_field: Company
                github_date_field: "Planned Date"
                type_label_map: "[['Issue','bug'],['Change Request','enhancement']]"
```

## How It Works

For those unfamiliar with GitHub Actions, here's a breakdown of the process:

1. In this step, the repository is cloned. A personal access token must be provided as token to allow the workflow to commit and push changes to the remote.

```yml
uses: actions/checkout@v3
with:
    token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

1. In this step, the sync betweeh Freshdesk and Github is run.

```yml
    uses: attieretief/freshdesk-github-issues@v1
    with:
        token: ${{ secrets.GH_API_KEY }}
        freshdesk_key: ${{ secrets.FRESHDESK_API_KEY }}
        freshdesk_domain: linc.co.za
        freshdesk_tag: DEV
        github_project_number: 10
        github_priority_field: Priority
        github_status_field: Status
        github_company_field: Company
        github_date_field: "Planned Date"
        type_label_map: "[['Issue','bug'],['Change Request','enhancement']]"
```

## Personal Access Token

Visit https://github.com/settings/tokens/new to create a new personal access token. Choose "Tokens (classic)" instead of "Fine-grained tokens".
