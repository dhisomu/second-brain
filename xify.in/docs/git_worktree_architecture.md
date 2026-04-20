[← Back to Documentation Index](../README.md#detailed-documentation-index)

# Git Environment & Subdomain Architecture

**Date:** 2026-02-21
**Status:** Active Implementation (Subdomain Pivot)

## 1. Overview
We have transitioned from a symlink-heavy "Hybrid Worktree" model to a robust **Isolated Branch-Stack Model**. This ensures absolute separation of code, database, and infrastructure between Development, Staging, and Production.

## 2. Environment Mapping

| Environment | Subdomain | Branch | Server Directory | Database |
| :--- | :--- | :--- | :--- | :--- |
| **Production** | `xify.in` | `master` | `[REPO_ROOT]/` | `forum.db` |
| **Staging** | `stage.xify.in` | `stage` | `[REPO_ROOT]/` | `forum_stage.db` |
| **Development** | `dev.xify.in` | `dev` | `[REPO_ROOT]/` | `forum_dev.db` |

## 3. Workflow (The "Push-Up" Promotion)

Each environment is a complete, fresh `git clone` of the main repository, checked out to its respective branch.

### 1. Active Development (`dev` branch)
*   **Work Dir**: `[REPO_ROOT]/`
*   **Action**: Implement features, tests, and UI changes here.
*   **Command**: `git commit && git push origin dev`

### 2. Testing & Quality Assurance (`stage` branch)
*   **Work Dir**: `[REPO_ROOT]/`
*   **Action**: Merge `dev` into `stage` to promote changes.
*   **Command**: 
    ```bash
    git checkout stage
    git merge dev
    git push origin stage
    ```
*   **Verification**: Test on `https://stage.xify.in`.

### 3. Production Release (`master` branch)
*   **Work Dir**: `[REPO_ROOT]/`
*   **Action**: Merge `stage` into `master` for final release.
*   **Command**:
    ```bash
    git checkout master
    git merge stage
    git push origin master
    ```

## 4. Large Binary Assets (Verge3D)
Since Verge3D binaries (`.bin`, `.gltf`) and the Manager core are often untracked or too large for Git, they must be manually synchronized or copied when creating a new stack:
1.  Initialize submodules: `git submodule update --init --recursive`
2.  Sync untracked assets: `cp -r [REPO_ROOT]/verge3d [REPO_ROOT]/`

## 5. Benefits of this Architecture
1.  **Zero Pollution**: No symlinks allowed in `www/Home/Apps`. Each environment serves its own physical files.
2.  **Infrastructure Isolation**: A syntax error in `dev`'s `docker-compose.yaml` only crashes the Dev stack.
3.  **Real SSL**: Each subdomain gets its own dedicated Let's Encrypt certificate via Traefik.
4.  **Database Safety**: Separate SQLite files prevent test data from ever touching production.
