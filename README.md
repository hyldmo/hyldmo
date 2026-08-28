![Metrics](https://raw.githubusercontent.com/hyldmo/hyldmo/main/github-metrics.svg)

## Notification janitor

The notification janitor marks selected bot-comment notifications as Done while
keeping their GitHub threads subscribed. It checks every repository available to
the configured token.

The safety checks preserve a notification when:

- Its issue or pull request was created by someone outside the configured rule.
- A human comment or review was added after the thread was last read.
- The notification timestamp does not match the latest bot comment.
- The thread changes while the janitor is checking it.
- GitHub returns incomplete activity or an API error.

### Token

Create a personal access token (classic) with the `repo` scope. Authorize it for
each organization that uses SAML SSO, then save it as the repository Actions
secret `NOTIFICATIONS_TOKEN`.

GitHub's classic `repo` scope grants read and write access. The janitor only calls
repository read endpoints and the notification endpoint that marks a thread Done.
Workflow logs contain hashed thread references and omit repository names, issue
titles, and pull request titles.

### Configuration

Edit [`.github/notification-janitor.json`](.github/notification-janitor.json):

```json
{
  "version": 1,
  "dryRun": true,
  "rules": [
    {
      "name": "GitHub Actions comments on my threads",
      "commentAuthors": ["github-actions[bot]"],
      "threadAuthors": ["@me"],
      "subjectTypes": ["PullRequest", "Issue"],
      "action": "done"
    }
  ]
}
```

`commentAuthors` contains exact GitHub logins and ignores letter case. `@me`
resolves to the token owner. Remove `threadAuthors` to apply a rule to threads
created by anyone. Rules apply across all repositories because the configuration
has no repository selector. `notificationLookbackHours` defaults to 72. It lets
the janitor process a new notification after you open it, while avoiding old Done
notifications returned by GitHub's API. The default `concurrency` value runs five
notification checks at a time and accepts values from 1 through 10.

The scheduled workflow runs daily at 04:17 UTC and follows the configuration's
`dryRun` value. Manual runs provide a `dry-run` or `apply` choice that overrides
the file.

Start with a manual dry run. Check the workflow summary, run it manually in apply
mode, then set `dryRun` to `false` for scheduled cleanup.
