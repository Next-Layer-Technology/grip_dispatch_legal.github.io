# Tracked-content malware policy

This is a static incident-indicator policy, not proof that a repository or host
is malware-free. Deployment/provisioning freezes and incident recovery remain
in effect. It scans current tracked Git blobs, not Git history, deployed files,
untracked files, dependency downloads or submodule contents. Submodules and
symlinks fail closed; symlinks are rejected before reading their blobs/targets.

The policy checks obfuscated JavaScript incident markers anywhere in tracked
content, nested editor/workspace folder-open tasks, automatic-task settings,
Node commands executing fonts/data, unexpected public/fonts files, font magic
bytes (including EOT and both OpenType variants), and executable asset content.
Font signatures are identification checks, not complete font-parser validation.
README.md and reviewed EOT/SVG/TTF/OTF/WOFF/WOFF2 formats are allowed in public/fonts.
Executable SVG handlers/scripts are rejected. No content directories are exempt.

Run the trusted policy without candidate imports:

```sh
python3 -I /trusted/path/test_malware_scan.py
python3 -I /trusted/path/malware_scan.py --repo /candidate --ref HEAD
# Omit --ref to inspect staged Git blobs, even if the worktree conceals them.
python3 -I /trusted/path/malware_scan.py --repo /candidate
```

The pull_request_target workflow is security-sensitive. GitHub uses the base
workflow; it checks out the exact base SHA as `trusted` and head SHA separately
as `candidate`. Only the base policy and base regression fixtures execute.
Candidate policy/tests, app commands, dependencies, lifecycle hooks, local
Actions and tests never execute in the scanner. Python isolated mode prevents
candidate imports. The token has read-only contents permission, checkout is
immutably pinned, credentials are not persisted, and scan jobs have a timeout.
No deployment credentials or application secrets are passed to the job.
Push, manual and reusable calls use the triggering commit as policy; those
are bootstrap/deployment checks, not independent approval of policy edits.

**Migration:** the trusted PR event does not become active until this workflow
and policy reach the base branch through review. A passing branch/manual scan
cannot verify the future PR event. Rollout commits use `[skip ci]` where existing application workflows could
execute during incident cleanup; repositories without other workflows can use
a scanner-only push. Manual dispatches are recorded separately. After review/merge (not performed
by this rollout), open/update a PR and verify its check and EWS runner assignment.
Do not infer required-check enforcement from the presence of this workflow.
Require the reviewed check for merging where GitHub's plan supports it; private
repositories on the current plan report protection/ruleset restrictions.

Keep existing deployment gates and other security/secret/test checks. A reusable
legacy malware gate can call the standardized workflow without duplicate PR
triggers. Application/deployment workflows remain subject to the incident hold.

## Public repository enforcement bootstrap

GitHub Pages is configured to publish only the default branch. This branch's
additional documentation commit intentionally permits the scanner-only push
workflow, without dispatching Pages or executing application code. That push
checks the exact proposed commit on the dedicated EWS scanner.

Manual-dispatch checks verify runner execution but do not satisfy GitHub's
required PR checks. A required-check rule must use an eligible event and the
GitHub Actions check identity. The first trusted PR event remains unverified
until the reviewed workflow reaches the base branch. This rollout does not
merge the policy or release the incident hold.
