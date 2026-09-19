# Changelog

## 1.0.1

- Pass every input to the shell through quoted environment variables, so a name with a space works and no input can inject a command
- Import the key with `gpg --batch`, so it never waits for a prompt
- Pin the release workflow's actions by commit SHA

## 1.0.0

Initial Release
