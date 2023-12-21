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
                freshdesk_domain: company.freshdesk.com
                freshdesk_tag: DEV
                github_project_number: 10
                github_priority_field: Priority
                github_status_field: Status
                github_company_field: Company
                github_date_field: "Planned Date"
                type_label_map: "[['Issue','bug'],['Change Request','enhancement']]"
                github_repo_language_filter: <LANGUAGE> [Optional]
```

## How It Works

For those unfamiliar with GitHub Actions, here's a breakdown of the process:

1. In this step, the repository is cloned. A personal access token must be provided as token to allow the workflow to commit and push changes to the remote.

```yml
uses: actions/checkout@v3
with:
    token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

1. In this step, the sync between Freshdesk and Github is run.

```yml
    uses: attieretief/freshdesk-github-issues@v1
    with:
        token: ${{ secrets.GH_API_KEY }}
        freshdesk_key: ${{ secrets.FRESHDESK_API_KEY }}
        freshdesk_domain: company.freshdesk.com
        freshdesk_tag: DEV
        github_project_number: 10
        github_priority_field: Priority
        github_status_field: Status
        github_company_field: Company
        github_date_field: "Planned Date"
        type_label_map: "[['Issue','bug'],['Change Request','enhancement']]"
        github_repo_language_filter: <LANGUAGE> [Optional]
```

### Freshdesk fields

The following list of fields will be created, if they don't exist in Freshdesk, as ticket fields
- Development Task Title (cf_development_task_title)
- Repository (cf_repository)
  - All repositories included in the sync will be added to the dropdown values for this field
  - Repositories are filtered for where ARCHIVED is False, and optionally where LANGUAGE = `github_repo_language_filter`
- Github Issue (cf_github_issue)
- Assigned Developer (cf_assigned_developer)
  - All organisation members will be added to the dropdown values for this field
- Development Status (cf_development_status)
  - All statuses from the `github_status_field` of the `github_project_number` specified in options, will be added to the dropdown values for this field
- Planned Date (cf_planned_date)

### Synced fields

#### Freshdesk to Github Issue

For each repository in github organisation within which this action is run, a list of Freshdesk tickets is retrieved using the following filters:
- connect to the Freshdesk API for the specified `freshdesk_domain` using the `freshdesk_key`
- filter tickets for where TAG = `freshdesk_tag`
- filter tickets for where status <=3 or >=6 (in other words all unresolved tickets by excluding statuses RESOLVED and CLOSED)

For each ticket retrieved, a Github issue is either created or retrieved if `cf_github_issue` is not NULL
- The Github issue title is created from the `cf_development_task_title` field, with the Freshdesk ticket number suffixed to the title as (FD#{ticket_id})
- A hyperlink to the Freshdesk ticket is added to the end of the Github issue body
- A label is added to the Github issue using the `type_label_map` option, which maps the Freshdesk ticket types to Github labels
- The Github issue is assigned to the `cf_assigned_developer` if one is specified
- If the Github issue exists in the specified `github_project_number`, the project item's `github_company_field` is updated with the Company of the Freshdesk ticket and the `github_priority_field` is updated with the Priority of the Freshdesk ticket

#### Github Issue to Freshdesk

If the Github issue exists in the specified `github_project_number`, the following project item fields are used to update the Freshdesk ticket:
- ticket `cf_development_status` is updated from the project item's `github_status_field`
- ticket `cf_planned_date` is updated from the project item's `github_date_field`
- ticket `cf_assigned_developer` is updated from the project item's Assignee

## Personal Access Token

Visit https://github.com/settings/tokens/new to create a new personal access token. Choose "Tokens (classic)" instead of "Fine-grained tokens".
