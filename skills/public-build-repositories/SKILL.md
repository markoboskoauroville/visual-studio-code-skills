---
name: public-build-repositories
description: Prefer public visibility for repositories that need GitHub-hosted Actions builds so their standard public-repository minutes are not charged to the private-runner allowance. Use when creating, moving, or maintaining a build repository, while enforcing an exposure audit and explicit approval.
---

# Public repositories for GitHub Actions builds

Marko's 2026-11-26 instruction: repositories that need builds should use public GitHub visibility where possible so
GitHub Actions does not consume his paid/private repository minute allowance.

## Trigger

Use this when:

- a new application/release repository gets a GitHub Actions build workflow;
- a build repository is currently private and Marko asks to avoid private Actions charges;
- CI needs to publish a release, APK, DMG, app package, or website.

## Required safety gate

Public visibility exposes the repository, commit history, workflow files, names, and release assets. It is not a
routine rename and must not be performed silently.

Before changing visibility:

1. Stop and inspect secrets:
   - search the full repository and tracked history for tokens, passwords, signing passwords, private keys,
     personal addresses, private API endpoints, customer data, and licensed third-party SDKs;
   - scan GitHub Actions workflow references to `secrets.*` and confirm values remain only in GitHub Secrets;
   - never print secret values in output.
2. Inspect licences and third-party sources:
   - remove or gitignore vendor code that may not be redistributed;
   - preserve required copyright, attribution, and licence files.
3. Confirm that the app can be signed without committing a signing key. Keep local keys and repository secret values
   out of git.
4. Ask for explicit confirmation naming the exact repository to expose. The user can blanket-approve one named repo or a
   clearly bounded group; do not infer approval from a cost-saving request alone.
5. Record a fresh public snapshot check after the change:
   - clone the public URL into a temporary directory;
   - verify expected files only;
   - verify no secret patterns in repository history.

## Procedure

After the gate passes:

```bash
gh repo edit OWNER/REPOSITORY --visibility public
gh repo view OWNER/REPOSITORY --json visibility,nameWithOwner,url
git remote set-url origin https://github.com/OWNER/REPOSITORY.git
```

Prefer `gh` for the visibility change. Read the returned `visibility` value back as proof.

For repositories that cannot be public:

- keep the repo private;
- minimize workflow minutes;
- use a self-hosted/local runner or build on the Mac where appropriate;
- disable redundant workflows and do not duplicate the public-default rule around the safety gate.

## Verification

Done means all of these are true:

- the repository's GitHub `visibility` is `PUBLIC`;
- the Actions workflow still resolves from the default branch;
- the latest build is green;
- release assets are still published;
- a fresh clone contains no secrets or prohibited third-party material.

## History

- 2026-11-26: Marko requested that this be standard practice to reduce GitHub build charges. The rule is written with
  the necessary audit and confirmation gate because changing private source to public is irreversible to visibility
  history and must never happen silently.
