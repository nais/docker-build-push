# Usage

```yaml
- uses: nais/docker-build-push@v0
  id: docker-push
  with:
    project_id: ${{ vars.NAIS_MANAGEMENT_PROJECT_ID }} # required
    identity_provider: ${{ secrets.NAIS_WORKLOAD_IDENTITY_PROVIDER }} # required
    team: myteam # required
    tag: custom_tag # optional
    push_image: true # optional, default true
    dockerfile: Dockerfile # optional, default Dockerfile
    docker_context: . # optional, default .

# ...
- name: Deploy
  uses: nais/deploy/actions/deploy@v1
  env:
    # ...
    IMAGE: ${{ steps.docker-push.outputs.image }}
```

## Dependency

This action depends on [nais/login](https://github.com/nais/login) to authenticate with the registry.
