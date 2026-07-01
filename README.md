# nais/docker-build-push

This action contains recommended steps for building and publishing a Docker image used with the Nais platform. See the [docs](https://doc.nais.io/build/how-to/build-and-deploy/) for a full example

It will authenticate with the registry and build the image, named after the repository, and use a generated tag containing the timestamp and short commit hash.

The `image` output is the canonical image reference. When a digest is available, it is returned as `<registry>/<repo>:<tag>@<digest>` so deployments and attestation can use an immutable image reference while still preserving the readable tag.

Before:

`europe-north1-docker.pkg.dev/nais-io/nais/images/app:2026.07.01-12.34-abcdef0`

After:

`europe-north1-docker.pkg.dev/nais-io/nais/images/app:2026.07.01-12.34-abcdef0@sha256:abc123...`

If no digest is available, the `image` output falls back to the tagged image reference only. This can happen for example when `push_image` is `false`, or when the underlying build step does not produce a digest. Consumers that require an immutable image reference should ensure a digest is present before using the output for deployment or attestation.

If you need to build multiple images from the same repository, you can use the `image_suffix` input to add a suffix to the image name.

## Usage

```yaml
jobs:
  build_and_push:
    permissions:
      contents: "read"
      id-token: "write"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - uses: nais/docker-build-push@v0
        id: docker-push
        with:
          team: myteam # required
          tenant: nav # optional
          tag: custom_tag # optional
          push_image: true # optional, default true
          pull: true # optional, default false
          dockerfile: Dockerfile # optional, default docker_context/Dockerfile
          docker_context: . # optional, default .
          image_suffix: # optional, default empty
          cache_from: type=gha # optional, default type=gha
          cache_to: type=gha,mode=max # optional, default type=gha,mode=max
          build_args: | # optional, default empty
            FOO=bar
            BAZ=qux
          build_secrets:
            | # optional, default empty. See https://docs.docker.com/build/ci/github-actions/secrets/
            key=string
          target: runtime # optional, default empty
          salsa: true # optional, default true, generates a attestation for the image
          byosbom: # defaults to use Trivy for SBOM generation, if salsa is true, but can be overwritten sending in a path to a  pre-generated SBOM
          additional_sboms: | # optional, newline-separated list of extra CycloneDX SBOM files to merge with the primary SBOM
            npm-sbom.json
            frontend-sbom.json
          platforms: # optional, pass trough to docker/build-push-action. See https://github.com/docker/build-push-action#usage. Requires setup-qemu-action, https://github.com/docker/setup-qemu-action#usage
      - name: Deploy
        uses: nais/deploy/actions/deploy@v2
        env:
          # ...
          IMAGE: ${{ steps.docker-push.outputs.image }}
```

## Dependency

This action depends on [nais/login](https://github.com/nais/login) to authenticate with the registry and [nais/attest-sign](https://github.com/nais/attest-sign) to add some [SLSA](https://slsa.dev/).

## SLSA

This action [signs](https://doc.nais.io/security/salsa/salsa/?h=slsa#what-is-slsa) the image so that its integrity can be verified by anyone wanting to use it. A "Software Bill Of Materials" is also automatically generated unless you bring your own. This happens "keylessly" behind the scenes, and the signatures are uploaded to the public [transparency log](https://search.sigstore.dev/).

If you need to combine image dependencies with application dependencies, use `byosbom` for the primary CycloneDX SBOM and `additional_sboms` for any extra CycloneDX SBOM files. These are merged by `nais/attest-sign` before attestation.

## Known issues

### Overwriting environment variables

When using this action in a job that also uses [navikt/frontend/actions/cdn-upload](https://github.com/navikt/frontend/tree/main/actions/cdn-upload/v1), you will get warnings that the latter of the two is overwriting multiple environment variables.

The two actions will both authenticate to Google Cloud, but with different service accounts and possibly project IDs.

The known solution is to split the job into two separate jobs.

### Long repo names or long branch names

`The size of mapped attribute exceeds the 127 bytes limit.`

When using [Google Workload-identity-federation](https://cloud.google.com/iam/docs/workload-identity-federation) we map attributes from authentication credentials issued by an external identity provider to Google Cloud attributes, such as `google.subject`.
`google.subject` is the principal IAM is authenticating and cannot exceed 127 bytes.
The error typically happens if you have a long branch name, or a long repo name.
Referring to this issue: https://github.com/google-github-actions/auth/blob/main/docs/TROUBLESHOOTING.md#subject-exceeds-the-127-byte-limit
