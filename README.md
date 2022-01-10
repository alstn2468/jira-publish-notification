# Jira Publish Notification

## Inputs

- `GITHUB_TOKEN`: `${{ secrets.GITHUB_TOKEN }}`
- `JIRA_CLIENT_ID`: JIRA Application client id
- `JIRA_CLIENT_SECRET`: JIRA Application client secret
- `JIRA_INSTANCE`: JIRA instance name `(https://<name>.atlassian.net/) -> <name>`
- `DEPLOY_ENVIRONMENT`: dev, prod, qa, etc...
  - default: `prod`

## How to get input related to JIRA

- JIRA_CLIENT_ID, JIRA_CLIENT_SECRET

Check out [OAuth](https://developer.atlassian.com/server/jira/platform/oauth/) document.

- JIRA_INSTANCE

`https://<name>.atlassian.net/` -> `<name>` is your instance name.

## Example workflow

```yml
name: Example using jira publish notification

on:
  push:
    branches: [master]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Setup Node.js environment
        uses: actions/setup-node@v2-beta
        with:
          node-version: '14'

      - uses: actions/cache@v2
        id: yarn-cache
        with:
          path: |
            node_modules
            packages/**/node_modules
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-

      - name: Install dependencies
        if: steps.yarn-cache.outputs.cache-hit != 'true'
        run: yarn

      - name: Build and deploy website
        run: |
          yarn build
          yarn deploy --yes

      - name: Send notification to JIRA
        uses: alstn2468/jira-publish-notification@v1.0.0
        if: ${{ success() }}
        with:
          GITHUB_TOKEN: '${{ secrets.GITHUB_TOKEN }}'
          JIRA_CLIENT_ID: '${{ secrets.JIRA_CLIENT_ID }}'
          JIRA_CLIENT_SECRET: '${{ secrets.JIRA_CLIENT_SECRET }}'
          JIRA_INSTANCE: 'instance'
          DEPLOY_ENVIRONMENT: 'dev'
```