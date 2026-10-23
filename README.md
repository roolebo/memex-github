# memex-github

A [memex] adapter for GitHub: `discussion` in pull requests, `tracker` in
issues and `ci` in Actions. The roles are specified in memex's
[discussion.md], [tracker.md] and [ci.md].

## Reaching GitHub

ADDR is the repository URL, `https://HOST/OWNER/REPO`. The adapter uses
the GraphQL API, and REST for `locate`, with the token in `GITHUB_TOKEN`,
or else from `gh auth token` for HOST, or else in `~/.netrc` for the API
host: `api.github.com`, or HOST on GitHub Enterprise.


## Commands

    locate COMMIT   GET /repos/OWNER/REPO/commits/COMMIT/pulls
    change CHANGE   pullRequest with force-push events and
                    latestOpinionatedReviews
    threads CHANGE  pullRequest with reviewThreads, comments and reviews
    drafts CHANGE   the viewer's PENDING review
    draft CHANGE    addPullRequestReviewThread or
                    addPullRequestReviewThreadReply on the pending review
    discard CHANGE ID
                    deletePullRequestReviewComment
    publish CHANGE  submitPullRequestReview

Capabilities: `series` is false, the others true.

## Change

- CHANGE is the pull request number.
- REVISION is a head commit: each force-push's `beforeCommit` and
  `afterCommit`, and `headRefOid`.
- `status`: OPEN is open, MERGED merged, CLOSED abandoned.
- `message`: the pull request's `body`.
- `votes`: APPROVED is `approved` 1, CHANGES_REQUESTED `approved` -1.
- `links`: `change`.

## Thread

- A thread is each of `reviewThreads`; a conversation comment or a
  review body is a thread of one comment on `change`.
- `resolved`: `isResolved`.
- ANCHOR: `subjectType` FILE is `file`; LINE is `lines` from
  `originalStartLine` to `originalLine`. `diffSide` LEFT is `base`.
- No `message` ANCHOR, no columns.

## Draft

- A draft on `change` is the pending review's `body`.
- `publish`: `approved` 1 is APPROVE, -1 REQUEST_CHANGES, else COMMENT.
- `resolved` in a draft is not supported.

## Tracker

    issue ISSUE     GET /repos/OWNER/REPO/issues/ISSUE and its comments
    search          GET /search/issues, `repo:OWNER/REPO is:issue` and
                    the QUERY
    comment ISSUE   POST /repos/OWNER/REPO/issues/ISSUE/comments
    transition ISSUE STATUS
                    PATCH /repos/OWNER/REPO/issues/ISSUE, STATUS open or
                    closed

- ISSUE is the issue number; a pull request's number is refused.
- `type`: the issue type, or null; `priority`: null.
- `closed`: `state` is closed.
- `related`: empty.
- `basis`: `updated_at`.

## CI

    runs COMMIT     GET /repos/OWNER/REPO/actions/runs?head_sha=COMMIT
    run RUN         GET /repos/OWNER/REPO/actions/runs/RUN
    suites RUN      GET /repos/OWNER/REPO/actions/runs/RUN/jobs
    logs ID         a job's log, or a run's logs and artifacts
    retest RUN      POST /repos/OWNER/REPO/actions/runs/RUN/rerun

Capabilities: `tests` and `bugs` are false, the others true.

- RUN is a workflow run, SUITE one of its jobs.
- `result`: success is pass, failure fail, skipped skip, timed_out
  timeout, neutral pass, anything else error.
- `config`: the workflow, event and branch.
- `links`: `commit`, and `change` while a pull request of the repository
  points at it.

## Open questions

- Fast-forward pushes leave no event: how to count REVISIONs?
- Lines outside the diff's hunks: draft on `file`, or fail?
- Stacked pull requests as a SERIES?
- `resolved` takes effect at once: hold it until `publish`?
- A fork: bind it, or its `parent`?

## Tests

`test/run [TEST...]` runs the tests against a fake GitHub; they need jq,
and no network.

[memex]: https://github.com/roolebo/memex
[discussion.md]: https://github.com/roolebo/memex/blob/main/skills/memex/discussion.md
[tracker.md]: https://github.com/roolebo/memex/blob/main/skills/memex/tracker.md
[ci.md]: https://github.com/roolebo/memex/blob/main/skills/memex/ci.md
