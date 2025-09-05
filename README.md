# Auth0 Account Link Extension

This extension provides a rule and interface for giving users the option of linking a new account
with an existing registered with the same email address from a different provider.

> **NOTE:** Please make sure you are using your own social connections (Google, Facebook, etc...) API keys. Using Auth0's keys will result on an 'Unauthorized' error on account linking skip.

## Example:
- You signed up with FooApp with your email, `greatname@example.com`.
- You come back some time later and forget whether you signed in with your email or Google account with the same email address.
- You try to use your Google account
- You're then greeted with the UI presented from this extension, asking you if
  you'd like to link this account created with your Google account with a
  pre-existing account (the original you created with a username and password).

## Running in Development

> **NOTE:** You will need your own NGROK_TOKEN. You can sign up for a free account at: https://dashboard.ngrok.com/signup

Update the configuration file under `./server/config.json`:

```json
{
  "EXTENSION_SECRET": "mysecret",
  "AUTH0_DOMAIN": "me.auth0.com",
  "AUTH0_CLIENT_ID": "myclientid",
  "AUTH0_CLIENT_SECRET": "myclientsecret",
  "WT_URL": "http://localhost:3000",
  "AUTH0_CALLBACK_URL": "http://localhost:3000/callback",
  "NGROK_TOKEN": "myngroktoken"
}
```

Then you can run the extension:

```bash
nvm use 22
yarn install
yarn run build
yarn run serve:dev
```

## Running puppeteer tests

In order to run the tests you'll have to [start the extension server locally](https://github.com/auth0-extensions/auth0-account-link-extension#running-in-development), fill the `config.test.json` file (normally with the same data as the `config.json` file) and run the Sample Test application located in `sample-app/` (create a dedicated client for this app).

Then, you can run the tests running:
```bash
yarn test
```

## Release Process

Deployment is now handled by the GitHub Actions workflow (`.github/workflows/build.yml`). It runs automatically on:

1. Push/merge to `master` (stable releases)
2. Manual trigger ("Run workflow" in the GitHub UI) against any branch (useful for beta / pre‑release testing)

### 1. Bump the Version
Update the version in BOTH `package.json` and `webtask.json`.

Use semantic versions. Append `-beta.N` for beta builds, e.g. `3.5.0-beta.1`.

### 2. Open a PR
Commit the version bump + any changes. When the PR merges to `master`, the workflow will build and publish automatically.

For a beta: work on any branch and ensure the version string itself includes `beta`.

### 3. (Optional) Manual Run
If you need to publish without merging yet, open the Actions tab, select the Build workflow, click "Run workflow", choose the branch, optionally include a note, and run it.

### 4. Build Locally (if you want to verify before pushing)
```bash
nvm use 22
yarn install
yarn run build
```

Artifacts produced:
- Bundle file (`auth0-account-link.extension.VERSION.js`) is found in `/dist`
- Asset CSS files are found in `/dist/assets`

### 5. Publication Targets
The workflow/script (`tools/cdn.sh`) uploads to S3:
- Stable (no `beta` in version): `s3://assets.us.auth0.com/extensions/auth0-account-link/`
- Beta (version contains `beta`): `s3://assets.us.auth0.com/extensions/develop/auth0-account-link/`

We publish both an full version (eg: `1.2.3`), and a major.minor version (eg: `1.2`). The major.minor version will be overwritten on each publish, while the full version will NOT be overwritten. 

### 6. Caching & Re-Publishing
Increment the version to force a new publish.

### 7. Testing a Beta or Candidate
Because beta assets are isolated under `extensions/develop`, production consumers will not pick them up automatically.

### 8. Promoting to Stable
Once validated, remove the `-beta.*` suffix, bump to the final version (e.g. `3.5.0`), merge to `master` (or manually trigger on the release commit), and a stable publish will occur.
