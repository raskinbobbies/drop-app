# Publishing an approved Drop installer

This is a manual process for the distribution repository
**`raskinbobbies/drop-app`**. It does not build the application, access a source
repository, or use GitHub Actions. Keep source, private history, and internal
release evidence outside this repository.

## Before publishing

1. Obtain explicit approval for the exact installer version and SHA-256 from the
   maintainer after physical acceptance and the required release gates.
   A generated installer or a partially accepted candidate is not sufficient.
2. Verify that it is the production installer, not an experimental or validation
   package. Confirm version, architecture, signing status, installer behavior,
   upgrade requirements, and known limitations.
3. Audit the exact installer and its payload for accidental data, source, debug
   symbols, credentials, private paths, test fixtures, logs, and models. Keep audit
   evidence private. Do not upload build folders or internal artifact manifests.
4. Stage only the approved EXE in a temporary directory outside this Git checkout.
   Recalculate its hash and compare it with the approval record. Do not rebuild,
   repackage, or otherwise change the approved bytes.
5. Prepare public release notes describing changes, requirements, signing status,
   upgrade guidance, and known limitations. Remove private issue links, file paths,
   source commit IDs, logs, and device/user identities. Get approval for publication.

## One-time repository creation

Only run these commands after approval to create the public repository and push
the independently prepared documentation history. Install GitHub CLI beforehand
and authenticate as the intended owner; never include tokens in a command or document.

Run from the distribution checkout, not from any development repository:

```powershell
gh auth status
gh api user --jq .login
gh api repos/raskinbobbies/drop-app --jq '{name: .full_name, visibility: .visibility}'
```

If the repository exists, **stop**. A public or private naming conflict is not
permission to reuse or change it. Proceed only if an authenticated lookup returns
HTTP 404 and the account has permission to create repositories under this owner;
an authentication, permission, or network error is not proof that the name is free.

Review `git ls-files`, `git log --all --stat`, and `git remote -v`. The initial
history must contain only public Markdown and the reviewed screenshots, with no
private ancestors and no existing remote. Then, after explicit approval:

```powershell
gh repo create raskinbobbies/drop-app --public --description "Fast, local-first notes and reminders for Windows. Official downloads and documentation." --source . --remote origin
git push -u origin main
```

This creates a new public repository. It must never be run with a private source
checkout as `--source`. Do not fork, mirror, copy Git objects, or change source
repository visibility. The proposed public repository has no software LICENSE file.

## Prepare release assets outside Git

The example below intentionally requires a real approved version and paths. Do
not substitute an unapproved candidate. The installer basename should be
`Drop-<version>-Setup.exe`; the hash must match the accepted bytes.

```powershell
$releaseVersion = '<approved-version>'
$installerPath = '<absolute-path-to-approved-installer>'
$releaseNotesPath = '<absolute-path-to-reviewed-public-release-notes.md>'
$checksumPath = Join-Path (Split-Path -Parent $installerPath) 'SHA256SUMS.txt'
$installerHash = (Get-FileHash -Algorithm SHA256 -LiteralPath $installerPath).Hash.ToLowerInvariant()
$installerName = Split-Path -Leaf $installerPath
"$installerHash  $installerName" | Set-Content -LiteralPath $checksumPath -Encoding ascii
```

Review the EXE, public notes, and checksum file as an explicit three-file
allowlist. Upload the installer and checksum only; use the notes as the release
description. Do not use wildcard uploads. Check that the version tag is unused:

```powershell
gh release view "v$releaseVersion" --repo raskinbobbies/drop-app
```

Stop if the version already exists. Do not replace an earlier approved release.
Do not interpret an authentication/network failure as a missing release.

## Create a draft, then publish

Creating or uploading a draft sends assets to GitHub and also requires approval.
Run from the distribution checkout only. Its current HEAD must already be pushed.
The target below is the documentation commit in this public repository, not a
private source commit. GitHub creates the version tag in the public repository.

```powershell
$distributionCommit = git rev-parse HEAD
gh release create "v$releaseVersion" "$installerPath" "$checksumPath" --repo raskinbobbies/drop-app --target "$distributionCommit" --title "Drop $releaseVersion" --notes-file "$releaseNotesPath" --draft
gh release view "v$releaseVersion" --repo raskinbobbies/drop-app
```

Inspect the draft assets and rendered notes. Verify uploaded bytes by downloading
to a new temporary folder outside the repository and comparing SHA-256 with the
approval record. After explicit approval to make this release public:

```powershell
gh release edit "v$releaseVersion" --repo raskinbobbies/drop-app --draft=false --latest
```

Use `--latest` only for an approved stable release, not a preview. Confirm the
[latest download link](https://github.com/raskinbobbies/drop-app/releases/latest),
asset names, checksums, and notes after publication. For the first release, update
the README's "downloads are not available yet" notice in a separate reviewed
documentation commit. Keep preview screenshots labeled until they match an
accepted release.

## Future updates

Repeat the approval, audit, hash, draft, verification, and publication steps with
each uniquely versioned installer. Commit only reviewed public documentation or
images to this repository. Keep installers, checksums, staging files, private
release records, and application code out of its Git history. No automated
connection to a private source repository is needed.

Command reference: [repository creation](https://cli.github.com/manual/gh_repo_create),
[release creation](https://cli.github.com/manual/gh_release_create), and
[release editing](https://cli.github.com/manual/gh_release_edit).
