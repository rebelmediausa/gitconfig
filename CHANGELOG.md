# Changelog

## 1.0.1

- Pass every input to the shell through quoted environment variables, so a name with a space works and no input can inject a command
- Import the key with `gpg --batch`, so it never waits for a prompt
- Pin the release workflow's actions by commit SHA
- Releases now also publish a GitHub release with the CHANGELOG notes
- Repository: gitflow (develop is the default branch), rulesets, CodeQL,
  Scorecard, dependency review, and a Lint check that rejects unpinned actions
  and interpolated inputs

## 1.0.0

Initial Release
