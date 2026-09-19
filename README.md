# Rebel Media GitHub Action `gitconfig`

Configure GIT to use a set user account and sign with GPG.

The GPG key must be base64 encoded:
`gpg --export-secret-keys <GPG_KEY_ID> | base64 > private.key`

Every input is passed to the shell quoted, so values with spaces (`git_user_name: Rebel Bot`) work as written.


## Usage
```yaml
    steps:
      - name: Setup GIT
        uses: rebelmediausa/gitconfig@v1
        with:
          git_user_name: BOT
          git_user_email: bot@example.com
          git_gpg_key: ${{ secrets.GPG_KEY }}
          git_gpg_key_id: ${{ secrets.GPG_KEY_ID }}
```
