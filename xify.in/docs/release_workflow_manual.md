[← Back to Documentation Index](../README.md#detailed-documentation-index)

# Release Workflow Manual (Hybrid Git)

This manual outlines the standard procedure for developing features and promoting them through the environments.

## 1. Development (Dev)
All new features start in the **Dev** worktree.

1.  **Enter Dev**: `cd [REPO_ROOT]/worktrees/xify_opssim_dev`
2.  **Code**: Make your changes.
3.  **Compress Assets**:
    ```bash
    brotli -kv *.{html,js,css,gltf,bin,hdr,png,jpg,jpeg,svg}
    gzip -kv *.{html,js,css,gltf,bin,hdr,png,jpg,jpeg,svg}
    ```
4.  **Commit & Push**:
    ```bash
    git add .
    git commit -m "feat: your feature description"
    git push origin dev
    ```

## 2. Testing (Stage)
Once ready for testing, move the code to the **Stage** branch.

1.  **Merge Dev to Stage**: (From the Dev folder)
    ```bash
    git fetch origin
    git merge origin/stage --no-edit  # Bring in any remote stage changes
    git push origin dev:stage        # Push your combined code to stage branch
    ```
2.  **Update Staging Site**:
    ```bash
    cd [REPO_ROOT]/worktrees/xify_opssim_stage
    git fetch origin
    git reset --hard origin/stage
    ```
3.  **Verify**: Test at `https://xify.in/Apps/xify_opssim_stage/index.html`.
4.  **QA Testing**: The **Quality Team** must now verify the features on Staging.
5.  **Pass Criteria**: Once QA testing is successful and results are documented, proceed to create the Pull Request.

## 3. Production (Master)
After verification, promote to **Production** via Pull Request.

1.  **Create & Merge PR**: Use the `gh` CLI from the **Dev** folder:
    ```bash
    cd [REPO_ROOT]/worktrees/xify_opssim_dev
    gh pr create --base master --head stage --title "Release: [Feature Name]" --body "Summary of changes"
    # Merge the PR using the number provided:
    gh pr merge <PR_NUMBER> --merge --admin
    ```
2.  **Update Live Site**:
    ```bash
    cd [REPO_ROOT]/www/Home/Apps/xify_opssim
    git pull origin master
    ```
3.  **Final Compression**: Run compression on the live folder:
    ```bash
    brotli -kv *.{html,js,css,gltf,bin,hdr,png,jpg,jpeg,svg}
    gzip -kv *.{html,js,css,gltf,bin,hdr,png,jpg,jpeg,svg}
    ```

## 4. Version Locking (Infrastructure)
The final critical step is to "lock" the production version in the main repository.

1.  **Lock Version**:
    ```bash
    cd [REPO_ROOT]
    git add www/Home/Apps/xify_opssim
    git commit -m "build: lock production app to latest release"
    git push xify.in master
    ```

---
*Note: For the **Viewer**, follow the same pattern using `[REPO_ROOT]/worktrees/xify_opssim_view_dev` and the `Viewer_Dev`/`viewer_stage` branches.*
