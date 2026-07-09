# Deployment

Team51 uses **GitHub-based deployment workflows** for both WPCOM Simple and Pressable sites. All code lives in GitHub and is deployed automatically or semi-automatically depending on the platform.

## Branch strategy

| Branch | Purpose | Deploys to |
|---|---|---|
| `trunk` (or `main`) | Production-ready code | Production site |
| `develop` | Active development and integration | Staging site |
| Feature branches | Individual features or fixes | Local/review only |

- `trunk` is the **source of truth** for production.
- `develop` is used for staging and integration testing.
- Feature branches are merged into `develop` (or directly to `trunk` for hotfixes) via pull requests.

## Deployment to Pressable

### Current methods

Team51 uses multiple deployment tools during the ongoing migration:

1. **GitHub Deployments** (preferred, actively migrating to) — WPCOM/Pressable connects directly to the GitHub repo and deploys on push.
2. **DeployHQ** (legacy, being phased out) — SSH-based deployment triggered by GitHub webhooks.
3. **Pressable GitHub integration** — direct repo connection via the Pressable control panel.

### How it works (GitHub Deployments)

1. Code is pushed to `trunk` (production) or `develop` (staging).
2. GitHub Deployments triggers an automated deploy to the connected Pressable site.
3. Deployment status is posted to the `#team51-bots` Slack channel.
4. Verify the deployment by checking the site and reviewing Slack notifications.

### How it works (DeployHQ — legacy)

1. Code is pushed to the target branch.
2. A GitHub webhook notifies DeployHQ.
3. DeployHQ connects to Pressable via SSH and syncs files.
4. Deployment status is posted to Slack.

## Deployment to WPCOM Simple

1. The GitHub repo is connected to the WPCOM site via **GitHub Deployments** (WPCOM feature).
2. Pushing to `trunk` triggers an automatic deployment of theme/plugin code.
3. Only the `wp-content` directory contents are deployed (themes, plugins, mu-plugins).

## Staging workflow

### Pressable
1. Create a staging site via the Pressable control panel or Team51 CLI.
2. Connect the `develop` branch to the staging site.
3. Push feature branches to `develop` to test on staging.
4. After QA approval, merge `develop` into `trunk` to deploy to production.

### WPCOM Simple
1. WPCOM allows one staging site per production site.
2. The staging site can be connected to a different branch for testing.
3. Test thoroughly on staging before merging to `trunk`.

## Production deployment checklist

Before deploying to production:

- [ ] Code passes all CI checks (PHPCS, PHPStan, PHPMD, syntax check)
- [ ] Changes have been tested on a staging environment
- [ ] PR has been reviewed (or self-reviewed for low-risk changes)
- [ ] No `WP_DEBUG`-level notices or errors
- [ ] Database migrations (if any) are backwards-compatible
- [ ] Cache-busting version strings updated on changed CSS/JS assets — see [Asset versioning](../conventions/coding-standards.md#asset-versioning)
- [ ] Partner has been notified if the change is visible to end users

## Rollback procedures

### Pressable
1. **Quick rollback**: revert the merge commit on `trunk` and push.
2. **DeployHQ rollback**: re-deploy a previous commit via the DeployHQ dashboard.
3. **Manual rollback**: SSH into the site and restore files from a Jetpack backup.

### WPCOM Simple
1. Revert the merge commit on `trunk` and push — GitHub Deployments will redeploy.
2. If code deployment is broken, use the WPCOM dashboard to restore from a Jetpack backup.

## Deployment notifications

- Deployment events are posted to **`#team51-bots`** in Slack.
- If a deployment fails, ping **`#team51-deployers`** for help.
- For urgent production issues, escalate in **`#team51`**.

## Production safety

All write operations must target staging, never production. See the **Production Safety (Non-Negotiable)** section in `AGENTS.md` for the full set of rules — they override everything else, including deployment workflows.

## Important notes

- **Never force-push to `trunk` or `develop`** — these are protected branches.
- **Always deploy via the standard workflow** — do not manually upload files via SFTP unless it's an emergency hotfix, and document it immediately.
- **Coordinate deployments** — check with the team before deploying large changes, especially on high-traffic sites.
- **`.deployignore`** — some repos include a `.deployignore` file to exclude dev-only files (tests, build configs) from deployment. Check if one exists and respect its contents.
