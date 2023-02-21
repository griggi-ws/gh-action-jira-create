# gh-action-jira-create

Use [GitHub actions](https://docs.github.com/en/actions) to create Jira tickets whenever an issue or pull request is created.
Supports arbitrary additional fields in the API call to Jira.

## Authentication

To provide a URL and credentials you can use the [`gajira-login`](https://github.com/atlassian/gajira-login) action, which will write a config file this action can read.
Alternatively, you can set some environment variables:

- `JIRA_BASE_URL` - e.g. `https://my-org.atlassian.net`. The URL for your Jira instance.
- `JIRA_API_TOKEN` - e.g. `iaJGSyaXqn95kqYvq3rcEGu884TCbMkU`. An access token.
- `JIRA_USER_EMAIL` - e.g. `user@example.com`. The email address for the access token.

## Inputs

- `project` (required) - The project key to create the issue in, e.g. `'TEST'`
- `issuetype` (required) - The issue type for the ticket, e.g. `'Bug'`
- `summary` (required) - The title of the issue, e.g. `'A summary'`
- `description` (required) - The body of the issue, e.g. `'A description of the issue'`
- `extraFields` - A JSON map as a string, specifying any additional fields to set in the create issue payload. See the [Jira REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-post) for more details of the available fields, e.g. `'{"parent": {"key": "FOO-23"}, "labels": ["github", "bug"], "customfield_10071": "from-github-action"}'`

## Outputs

- `issue` - The key of the issue created, e.g. TEST-23

## Examples

The following examples are valid `steps` for a particular job in a workflow; to see how to integrate them into a fully worked example, refer to the [documentation](https://docs.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow).

Using `atlassian/gajira-login` and [GitHub secrets](https://docs.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) for authentication:

```yaml
- name: Login
  uses: atlassian/gajira-login@v2.0.0
  env:
    JIRA_BASE_URL: ${{ secrets.JIRA_BASE_URL }}
    JIRA_USER_EMAIL: ${{ secrets.JIRA_USER_EMAIL }}
    JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}

- name: Create
  id: create
  uses: griggi-ws/gh-action-jira-create@v0.1.3
  with:
    project: FOO
    issuetype: "Bug"
    summary: "The summary"
    description: "The description"

- name: Log
  run: echo "Created issue ${{ steps.create.outputs.issue }}"
```

Using environment variables for authentication, and the `github` context to populate some additional fields:

```yaml
- name: Create
  id: create
  uses: griggi-ws/gh-action-jira-create@v0.1.3
  with:
    project: FOO
    issuetype: "Bug"
    summary: "${{ github.event.issue.title }} #${{ github.event.issue.number }}"
    description: "${{ github.event.issue.body }}\n\nCreated from GitHub Action"
    extraFields: '{"fixVersions": [{"name": "TBD"}], "customfield_20000": "product", "customfield_40000": "${{ github.event.issue.html_url }}"}'
  env:
    JIRA_BASE_URL: ${{ secrets.JIRA_BASE_URL }}
    JIRA_USER_EMAIL: ${{ secrets.JIRA_USER_EMAIL }}
    JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
```
