# Helping external-dns find the parent Cloudflare zone

## Overview

I opened a pull request to fix a Cloudflare zone-matching issue in external-dns. When configured to manage `dev.example.com`, the tool could miss the parent zone, `example.com`, and make no DNS changes.

The fix adds an optional flag that allows parent-zone matching while keeping record changes within the requested subdomain. Claude Code helped me investigate the issue and write the change. I reproduced the bug and reviewed the diff. The [pull request](https://github.com/kubernetes-sigs/external-dns/pull/6967) is open and awaiting review.

## What happened

I used `example.com` above to illustrate the bug. To reproduce it, I ran three filters against the mock Cloudflare client’s `bar.com` test zone. The original code found `bar.com` when the filter matched its name, but returned no zones for either subdomain:

```text
domain-filter="bar.com"         -> zones found: [bar.com]
domain-filter="dev.bar.com"     -> zones found: []
domain-filter="www.dev.bar.com" -> zones found: []
```

The problem was in `Zones()`, which checked the domain filter against the zone name. That meant a filter for `dev.bar.com` didn’t match the zone `bar.com`.

On November 16, I asked on the issue if I could take it on. The previous claimant, omkar619-dev, told me to go ahead, confirmed their earlier suggestion to make `--cloudflare-zone-match-parent` opt-in, and offered to review the pull request.

The AWS provider already supported parent-zone matching, so I followed that approach and reused the existing `MatchParent` helper in the Cloudflare provider. The new flag, `--cloudflare-zone-match-parent`, is off by default to preserve existing behavior.

Three of the eight zone-matching test cases failed before I added the matching logic. After the change, all eight passed.

Once the parent zone matched, I wanted to check whether the fix could affect other records in that zone. I tested three records together: `dev.bar.com`, `other.bar.com`, and `bar.com`.

My first test assumed that updating a record wouldn’t involve deleting it. Running the test showed that the Cloudflare provider updates a record by deleting the old one and creating a replacement. I adjusted the test to allow that for `dev.bar.com` while checking that the sibling and apex records stayed untouched.

The [test in the PR](https://github.com/kubernetes-sigs/external-dns/pull/6967/files) includes these checks:

```go
assert.NotContains(t, []string{"sibling", "apex"}, action.RecordId, "records outside the domain filter must not be touched")
if action.Name == "Delete" {
    // the provider applies an update as a delete of the old record plus a create
    assert.Equal(t, "inside", action.RecordId, "only the record under the domain filter may be deleted")
}
```

To check that the test could catch a regression, I temporarily removed the domain filter from the planner setup inside the test. The test failed when records outside the requested subdomain were deleted.

After restoring the filter, I pointed the built binary at a small HTTP server I wrote to mimic the Cloudflare API. I set `CLOUDFLARE_BASE_URL` to the server’s address. With the new flag off, the binary made no record changes. With the flag on, it created six records under `dev.example.com`, with no updates or deletes.

I submitted the fix with regression tests and documentation. The contributor agreement check passed, and the PR is waiting for review and permission to run upstream CI.

## Limitations and what's next

So far, I’ve tested the change using the project’s mock client and a local fake API. I still need to validate it against a live Cloudflare zone. For that, I would use a disposable zone containing an existing record under the target subdomain and a sibling record outside it, then confirm that syncing updates the target record and leaves the sibling untouched.
