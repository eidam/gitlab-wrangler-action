# Wrangler Action for Gitlab CI/CD

✨ Zero-config [Cloudflare Workers](https://workers.cloudflare.com) deployment using [Wrangler](https://github.com/cloudflare/wrangler) and [Gitlab CI/CD](https://docs.gitlab.com/ee/ci/)

Built from Cloudflare's [Wrangler Action](https://github.com/cloudflare/wrangler-action) and should be fairly easy to migrate between them as it uses same deployment script in the background.

## Usage

Add `gitlab-wrangler-action` to the CI/CD configuration of your Gitlab repository. The below example will publish your application on pushes to the `master` branch:

```yaml
include: 
  - remote: "https://raw.githubusercontent.com/eidam/gitlab-wrangler-action/main/templates/1.3.0.yml"

stages:
  - deploy

wrangler deploy:
  extends: .wrangler_deploy
  stage: deploy
  rules:
    - if: '$CI_COMMIT_BRANCH == "master"'
```

## Authentication

You'll need to configure Wrangler using Gitlab CI/CD variables feature - go to "Settings -> CI/CD -> Variables" and add your Cloudflare API token (for help finding this, see the [Workers documentation](https://developers.cloudflare.com/workers/quickstart/#api-token)). Your API token is encrypted by Gitlab, and the action won't print it into logs _(make sure you select "Mask variable" option)_.

| Variable name | Description           |
| ------------- | -------------         |
| CF_API_TOKEN  | Cloudflare API token  | 
| CF_ACCOUNT_ID | Can be set either in CI/CD variable or in your `wrangler.toml` |
| CF_ZONE_ID    | Can be set either in CI/CD variable or in your `wrangler.toml` |


`gitlab-wrangler-action` also supports using your [global API key and email](https://developers.cloudflare.com/workers/quickstart/#global-api-key) as an authentication method, although API tokens are preferred. Pass in `CF_API_KEY` and `CF_API_EMAIL` to the CI/CD variables intead.

## Configuration

You can additionaly set following CI/CD variables, either in repository settings or within the `wrangler deploy` job.

| Variable name         | Description
| --------------------- | ------------
| WRANGLER_VERSION      | Use specific Wrangler version
| WORKING_DIRECTORY     | Folder to run Wrangler commands from 
| ENVIRONMENT           | Wrangler environment to use
| PUBLISH               | If false, skip Wranger put secrets / deployment, useful for cases to run build only
| SECRETS               | Secret to create, see example below
| PRECOMMANDS           | Commands to run before Wrangler deployment, see example below
| POSTCOMMANDS          | Comamnd to run after Wrangler deployment

The below example will publish your application on pushes to the `dev` branch to `dev` env, with creation of KV namespace before the deployment:

```yaml
include: 
  - remote: "https://raw.githubusercontent.com/eidam/gitlab-wrangler-action/main/templates/1.3.0.yml"

stages:
  - deploy

wrangler deploy dev:
  extends: .wrangler_deploy
  stage: deploy
  rules:
    - if: '$CI_COMMIT_BRANCH == "dev"'
  variables:
    ENVIRONMENT: dev
    PRECOMMANDS: |
      wrangler kv:namespace create MY_NAMESPACE || namespace already exists
```

[Worker secrets](https://developers.cloudflare.com/workers/tooling/wrangler/secrets/) can be optionally passed as a new line deliminated string of names in `secrets`.  Each secret name must match an environment variable name specified in the CI/CD variables. Creates or replaces the value for the Worker secret using the `wrangler secret put` command.

```yaml
include: 
  - remote: "https://raw.githubusercontent.com/eidam/gitlab-wrangler-action/main/templates/1.3.0.yml"

stages:
  - deploy

wrangler deploy:
  extends: .wrangler_deploy
  stage: deploy
  variables:
    SECRETS: |
      SECRET_ONE
      SECRET_TWO
      SECRET_THREE
  rules:
    - if: '$CI_COMMIT_BRANCH == "master"'
```

## Troubleshooting

This Gitlab CI template is in beta, and I'm looking for folks to use it! If something goes wrong, please file an issue! That being said, there's a couple things you should know:

### "I just started using Workers/Wrangler and I don't know what this is!"

No problem! Check out the [Quick Start guide](https://developers.cloudflare.com/workers/quickstart) in our docs to get started. Once you have a Workers application, you may want to set it up to automatically deploy from Gitlab whenever you change your project. That's where this action comes in - nice!

### "I'm trying to deploy my static site but it isn't working!"

To deploy static sites and frontend applications to Workers, check out the documentation for [Workers Sites](https://developers.cloudflare.com/workers/sites).

Note that this action makes no assumptions about _how_ your project is built! **If you need to run a pre-publish step, like building your application, you need to specify a build step in your Gitlab CI/CD configuration and pass it as for example CI artifacts.**

## Plans / TODO

- host README and templates on Cloudflare Pages
