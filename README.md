# Rebel Media GitHub Action `gitconfig`

Configure GIT to use a set user account and sign with GPG.

GPG key but be in base64 format.
`gpg --export-secret-keys <GPPKEYID> | base64 > private.key`


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
