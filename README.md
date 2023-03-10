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
