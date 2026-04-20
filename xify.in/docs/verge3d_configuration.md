[← Back to Documentation Index](../README.md#detailed-documentation-index)

# Verge3D Configuration

This document outlines the Verge3D configuration for Xify.in, including how to handle external application directories and Docker volume mounting.

## Settings Configuration

The primary configuration for Verge3D is located at:
`[verge3d_config/settings.json](../verge3d_config/settings.json)`

### External Applications Directory

To allow Verge3D to discover applications outside its default `applications` folder (e.g., development worktrees), update the `extAppsDirectory` setting:

```json
{
    ...
    "extAppsDirectory": "[REPO_ROOT]/worktrees/",
    ...
}
```

## Docker Volume Mounting

Because Verge3D runs inside a Docker container, any host directory specified in `extAppsDirectory` must also be mounted into the container.

### [MODIFY] [docker-compose.yaml](../docker-compose.yaml)

Add the directory to the `volumes` section of the `verge3d` service:

```yaml
  verge3d:
    image: python:3.9-slim
    container_name: xify-verge3d
    ...
    volumes:
      - ./verge3d:/verge3d
      - ./www/Home/Apps:/verge3d/applications
      - ./www/Home/Apps:[REPO_ROOT]/www/Home/Apps
      - ./worktrees:[REPO_ROOT]/worktrees  <-- New Mount
      - ./verge3d_config:/root/.config/verge3d_for_blender
```

## Applying Changes

After modifying the configuration or `docker-compose.yaml`, restart the Verge3D service to apply the changes:

```bash
docker-compose up -d verge3d
```

## Multi-App Support

The `worktrees` directory can contain multiple application folders (e.g., `OPSSIM_Hub`, `xify_opssim_dev`). Verge3D will automatically scan this directory and list all found applications in the App Manager at [https://xify.in/app-manager/](https://xify.in/app-manager/).
