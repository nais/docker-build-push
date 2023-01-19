# Usage

```yaml
- uses: nais/docker-push@v0
  id: docker-push
  with:
    config: ${{ vars.DOCKER_PUSH_CONFIG }} # required
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
