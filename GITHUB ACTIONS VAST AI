# 🚀 GitHub Actions → Vast.ai Auto-Deployment Manual

This guide describes how to set up automatic deployments from a GitHub repository to a Vast.ai instance when changes are pushed to a selected branch.

---

## 🧭 Overview

Whenever you push code to a specific branch (e.g. `db_bug_fix_development`),  
GitHub Actions will automatically connect to your Vast.ai instance over SSH,  
fetch the latest commits from your repository, and update the project folder.

---

## 🧩 1. Vast.ai Instance Preparation

1. **Check your instance details**
   - Public IP (e.g. `74.99.152.106`)
   - External SSH port mapping (e.g. `50036 → 22`)
   - Username (`root` or `ubuntu`)

   You can verify this in your Vast.ai dashboard under **Instances → Connect**.

2. **Verify connectivity**
   ```bash
   ssh root@74.99.152.106 -p 50036
   ```

3. **Ensure the instance trusts your CI key**

   * Open `/root/.ssh/authorized_keys`
   * Add your CI/CD **public key**
   * Set correct permissions:

     ```bash
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
     ```

---

## 🔑 2. Generate a Dedicated CI/CD Keypair

On your local machine or on Vast:

```bash
ssh-keygen -t ed25519 -f vast_ci_key -C "github_actions"
```

* Copy the **public key** (`vast_ci_key.pub`) to Vast’s `/root/.ssh/authorized_keys`
* Copy the **private key** content to your clipboard (you’ll store it in GitHub next)

---

## ⚙️ 3. Create GitHub Secrets

In your repository:
**Settings → Secrets and variables → Actions → New repository secret**

| Secret Name     | Example Value               | Description                        |
| --------------- | --------------------------- | ---------------------------------- |
| `VAST_HOST`     | `74.99.152.106`             | Vast.ai public IP                  |
| `VAST_PORT`     | `50036`                     | External SSH port shown in Vast.ai |
| `VAST_USER`     | `root`                      | Default Vast username              |
| `VAST_SSH_KEY`  | *(contents of private key)* | The CI key’s private key           |
| `MY_GITHUB_PAT` | `ghp_XXXXXXXXXXXX`          | GitHub Personal Access Token (PAT) |

> **PAT scopes:**
> `repo`, `read:packages`, (optional) `workflow`

---

## 🔄 4. Update the Git Remote on Vast.ai (One-Time Setup)

From inside your Vast instance:

```bash
cd /workspace/clair-V1.2-restructure
git remote set-url origin https://<USERNAME>:<TOKEN>@github.com/<OWNER>/<REPO>.git
git fetch origin db_bug_fix_development
```

Example:

```bash
git remote set-url origin https://TheKnownGunman:ghp_xxx@github.com/phonx-ai/Clair-V1.2-restructure.git
```

---

## 🧰 5. Create the GitHub Actions Workflow

Create a file at:

```
.github/workflows/deploy_to_vast.yml
```

Paste this:

```yaml
name: Deploy to Vast.ai

on:
  push:
    branches:
      - db_bug_fix_development   # Change to your target branch later

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup SSH for Vast.ai
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VAST_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -p ${{ secrets.VAST_PORT }} -H ${{ secrets.VAST_HOST }} >> ~/.ssh/known_hosts

      - name: Test SSH connectivity
        run: |
          ssh -p ${{ secrets.VAST_PORT }} -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no \
            ${{ secrets.VAST_USER }}@${{ secrets.VAST_HOST }} "echo '✅ Connected to Vast.ai instance successfully.'"

      - name: Deploy latest branch
        env:
          GITHUB_PAT: ${{ secrets.MY_GITHUB_PAT }}
        run: |
          ssh -p ${{ secrets.VAST_PORT }} -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no \
            ${{ secrets.VAST_USER }}@${{ secrets.VAST_HOST }} "
              cd /workspace/clair-V1.2-restructure &&
              git remote set-url origin https://TheKnownGunman:${GITHUB_PAT}@github.com/phonx-ai/Clair-V1.2-restructure.git &&
              git fetch origin db_bug_fix_development &&
              git reset --hard origin/db_bug_fix_development &&
              echo '✅ Deployment completed successfully.'
            "
```

---

## 🧪 6. Test the Workflow

1. Push a test commit to `db_bug_fix_development`
2. Open your repo’s **Actions** tab → watch “Deploy to Vast.ai”
3. If it passes:

   * Check your Vast instance folder:

     ```bash
     cd /workspace/clair-V1.2-restructure
     ls -l
     ```
   * Confirm your code is updated.

---

## 🔁 7. Optional Enhancements

### Auto-Restart Your App

Append this at the end of the deploy command:

```bash
systemctl restart clair.service
# or
docker compose down && docker compose up -d --build
```

### Deploy Other Branches

Duplicate the workflow file and change:

```yaml
branches:
  - production
```

and the deployment path if needed.

### Multi-Repo Setup

Reuse the same secrets for multiple repos;
only the `branch` and `destination folder` need to change.

---

## ✅ 8. Quick Troubleshooting

| Error                                              | Likely Cause                    | Fix                                      |
| -------------------------------------------------- | ------------------------------- | ---------------------------------------- |
| `Connection timed out`                             | Missing `-p` port or wrong host | Check Vast external port mapping         |
| `Permission denied (publickey)`                    | Public key not added on Vast    | Add CI key’s `.pub` to `authorized_keys` |
| `error in libcrypto`                               | Bad key format in GitHub secret | Re-add correctly formatted PEM           |
| `could not read Username for 'https://github.com'` | Remote uses HTTPS without token | Set remote with PAT                      |
| `rsync: command not found`                         | Not installed                   | `apt install rsync -y` on Vast           |

---

## 🧠 9. Summary of Required Data

| Field         | Example                                                  | Source                                                   |
| ------------- | -------------------------------------------------------- | -------------------------------------------------------- |
| Vast IP       | `74.99.152.106`                                          | Vast.ai console                                          |
| Vast Port     | `50036`                                                  | Vast.ai console                                          |
| Vast Username | `root`                                                   | Default or custom                                        |
| GitHub Token  | `ghp_xxx`                                                | [GitHub Tokens Page](https://github.com/settings/tokens) |
| Repo URL      | `https://github.com/phonx-ai/Clair-V1.2-restructure.git` | GitHub                                                   |

---

### ✅ Success Message

When everything works, you’ll see this in the Actions logs:

```
✅ Connected to Vast.ai instance successfully.
✅ Deployment completed successfully.
```

---

## 🧾 Version Log

* **v1.0 (Nov 2025):** Standardized for Phonx-AI and Clair App deployments
* Verified components:

  * GitHub Actions (Ubuntu runner)
  * Vast.ai SSH port mapping
  * HTTPS + PAT git authentication
  * Branch-based auto-deployment

---

**Author:** Benjamin Mutebi
**Project:** Phonx-AI / Clair-V1.2
**Purpose:** Reliable and secure auto-deployment from GitHub to Vast.ai instances

```

---

Would you like me to add a short **“Multiple Repos Setup Template”** section to the bottom — showing how to reuse the same secrets and just swap branch/path for new repos (e.g. `insurance-ai`, `data-operations`, etc.)?
```
