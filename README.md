# Confidential Ubuntu configuration

Public deployment configuration for Tinfoil's private Ubuntu sandbox image.
Each release pins an immutable image digest and publishes its measured deployment.
Image source and bootstrap SSH authorization stay in the private image repository.

## Deploy

Configure access to the private GHCR image in your Tinfoil organization's registry
settings. The GitHub account supplying those credentials needs read access to the
private image repository, with **Inherit access from repository** enabled on its
GHCR package. Use a personal access token (classic) with `read:packages`, and
authorize it for SSO if your organization requires it. See
[GitHub's registry authentication guide](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-to-the-container-registry).

Use an allocated persistent disk on the selected host and an approved
configuration release. Set `TINFOIL_API_KEY` to your Tinfoil admin API key:

```sh
tinfoil container create my-sandbox \
  --repo tinfoilsh/confidential-ubuntu-config \
  --tag <release-tag> \
  --host <host> \
  --volume <disk-name>:workspace
```

The selected SSH login key must already be authorized by the image. After the
container is ready, verify the approved configuration release and install its
attested host-key profile:

```sh
tinfoil attest-ssh my-sandbox \
  --repo tinfoilsh/confidential-ubuntu-config@<release-tag> \
  --identity <bootstrap-private-key-file> \
  --install
ssh my-sandbox workspace-unlock < workspace.key
ssh my-sandbox
```

For a new blank disk, generate `workspace.key` once with
`openssl rand -out workspace.key 64` before unlocking. For an existing disk,
use its original key file. Reuse that key after relaunch and refresh the attested
SSH profile before reconnecting. Keep the key while retaining the persistent disk.
The customer enrollment tool can automate disk unlock and Teleport joining with
AWS Secrets Manager and the customer's Teleport settings.

See the [CLI documentation](https://docs.tinfoil.sh/containers/cli),
[attested SSH guide](https://docs.tinfoil.sh/containers/attested-ssh), and
[encrypted-volume guide](https://docs.tinfoil.sh/containers/encrypted-volumes).

## Release

Build and publish the image from the private source repository. Update the
`image` digest in `tinfoil-config.yml`, review that commit, then push a new `v*` tag.
The release workflow measures the configuration and publishes its signed
deployment artifacts. It does not build or pull the workload image and needs no
private-image credentials.

Keep GHCR package permissions linked to the private source repository. This
public repository contains no bootstrap authorized keys or registry credentials.
