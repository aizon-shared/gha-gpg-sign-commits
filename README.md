# gha-sign-commit

Install and configure GPG and Git to be able to sign commit using Github Actions runners

## Inputs

```yaml
inputs:
  gpgPrivateKey:
    description: "The private key in ASCII format to import"
    required: true
  gpgPublicKey:
    description: "The public key in ASCII format to import"
    required: false
  privateKeyPassphrase:
    description: "The Passphrase of the private key"
    required: false
  gpgDefaultCacheTtl:
    description: "The default-cache-ttl value to set in gpg-agent.conf"
    default: 7200
    required: false
  gpgMaxCacheTtl:
    description: "The max-cache-ttl value to set in gpg-agent.conf"
    default: 7200
    required: false
  gitSignCommits:
    description: |-
      Enable git configurations to sign commits with the imported key.

      This will override any existing git configurations for:
      - user.email
      - user.name
      - user.signingkey
      - commit.gpgsign
      - gpg.program

      Name and email will be set to the values of the imported private key.
    default: true
    required: false
  debug:
    description: "Enable debug mode. This shows the content of the important files."
    default: false
    required: false
```

## Outputs

```yaml
outputs:
  gpg-version:
    description: "GPG used version"
    value: ${{ steps.install.outputs.gpg-version }}
  gpg-fingerprint:
    description: "GPG key fingerprint short format"
    value: ${{ steps.export-gpg-details.outputs.gpg-fingerprint }}
  gpg-fingerprint-long:
    description: "GPG key fingerprint long format"
    value: ${{ steps.export-gpg-details.outputs.gpg-fingerprint-long }}
  gpg-email:
    description: "GPG key email"
    value: ${{ steps.export-gpg-details.outputs.gpg-email }}
  gpg-name:
    description: "GPG key name"
    value: ${{ steps.export-gpg-details.outputs.gpg-name }}
  git-user-email:
    description: "Git user email coming from GPG key"
    value: ${{ steps.setup-git.outputs.git-user-email }}
  git-user-name:
    description: "Git user name coming from GPG key"
    value: ${{ steps.setup-git.outputs.git-user-name }}
  git-user-signingkey:
    description: "Git user signing key coming from GPG key"
    value: ${{ steps.setup-git.outputs.git-user-signingkey }}
  git-commit-gpgsign:
    description: "Git commit gpgsign value"
    value: ${{ steps.setup-git.outputs.git-commit-gpgsign }}
  git-gpg-program:
    description: "Git gpg program value"
    value: ${{ steps.setup-git.outputs.git-gpg-program }}
```

## Usage

```yaml
...

jobs:
  sign-commit:
    runs-on: ubuntu-latest
    steps:
      - name: Setup GPG and Git sign commits
        id: setup
        uses: aizon-shared/gha-gpg-sign-commits@V1
        with:
          gpgPrivateKey: ${{ secrets.GPG_PRIVATE_KEY }}
          privateKeyPassphrase: ${{ secrets.GPG_PRIVATE_KEY_PASSPHRASE }}

      - name: Show outputs
        id: show-outputs
        run: |
          echo "gpg version: ${{ steps.setup.outputs.gpg-version }}"
          echo "gpg fingerprint: ${{ steps.setup.outputs.gpg-fingerprint }}"
          echo "gpg email: ${{ steps.setup.outputs.gpg-email }}"
          echo "gpg name: ${{ steps.setup.outputs.gpg-name }}"
          echo "git user email value: ${{ steps.setup.outputs.git-user-email }}"
          echo "git user name value: ${{ steps.setup.outputs.git-user-name }}"
          echo "git user singning key value: ${{ steps.setup.outputs.git-user-signingkey }}"
          echo "git commit gpgsign value: ${{ steps.setup.outputs.git-commit-gpgsign }}"
          echo "git commit gpg program value: ${{ steps.setup.outputs.git-gpg-program }}"

      - name: Commit and push changes
        run: |
          NOW=$(date)
          echo "${NOW}: test signed commit from ${{ github.repository }} in ${{ github.repository_owner }} with job id ${{ github.run_id }}" >> this-file-is-signed-commit.txt
          git add .
          git commit -m "test: this commit is signed with GPG"
          git push
...

```

## License

This module is released under the Apache License Version 2.0:

* [http://www.apache.org/licenses/LICENSE-2.0.html](http://www.apache.org/licenses/LICENSE-2.0.html)
