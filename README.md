# Usage

```yaml
jobs:
  build_and_push:
    permissions:
      contents: "read"
      id-token: "write"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - uses: nais/docker-build-push@v0
        id: docker-push
        with:
          team: myteam # required
          tag: custom_tag # optional
          push_image: true # optional, default true
          dockerfile: Dockerfile # optional, default Dockerfile
          docker_context: . # optional, default .
          image_suffix: # optional, default empty
          cache_from: type=gha # optional, default type=gha
          cache_to: type=gha,mode=max # optional, default type=gha,mode=max
          project_id: ${{ vars.NAIS_MANAGEMENT_PROJECT_ID }} # required, but is defined as an organization variable
          identity_provider: ${{ secrets.NAIS_WORKLOAD_IDENTITY_PROVIDER }} # required, but is defined as an organization secret
      - name: Deploy
        uses: nais/deploy/actions/deploy@v1
        env:
          # ...
          IMAGE: ${{ steps.docker-push.outputs.image }}
```

## Dependency

This action depends on [nais/login](https://github.com/nais/login) to authenticate with the registry.

## Known issues

### Overwriting environment variables

When using this action in a job that also uses [navikt/frontend/actions/cdn-upload](https://github.com/navikt/frontend/tree/main/actions/cdn-upload/v1), you will get warnings that the latter of the two is overwriting multiple environment variables.

The two actions will both authenticate to Google Cloud, but with different service accounts and possibly project IDs.

The known solution is to split the job into two separate jobs.
