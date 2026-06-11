# ຄູ່ມືການ Deploy LaoShop ຂຶ້ນ AWS Amplify

> ໂປຣເຈັກນີ້ແມ່ນ **Static HTML + AWS Amplify Gen 2 Backend** (Auth + Data/DynamoDB)  
> ການ Deploy ໃຊ້ **AWS Amplify Hosting** ເຊິ່ງ handle ທັງ frontend ແລະ backend ໃຫ້ອັດຕະໂນມັດ

---

## ສາລະບານ

1. [ກຽມເຄື່ອງມືທີ່ຕ້ອງການ](#1-ກຽມເຄື່ອງມືທີ່ຕ້ອງການ)
2. [ກຽມ AWS Account](#2-ກຽມ-aws-account)
3. [ສ້າງ GitHub Repository](#3-ສ້າງ-github-repository)
4. [ເຊື່ອມ Amplify ກັບ GitHub](#4-ເຊື່ອມ-amplify-ກັບ-github)
5. [ຕັ້ງຄ່າ Build Settings](#5-ຕັ້ງຄ່າ-build-settings)
6. [ຕັ້ງຄ່າ IAM Role ສຳລັບ CI/CD](#6-ຕັ້ງຄ່າ-iam-role-ສຳລັບ-cicd)
7. [ຕັ້ງຄ່າ GitHub Secrets](#7-ຕັ້ງຄ່າ-github-secrets)
8. [ທົດສອບການ Deploy](#8-ທົດສອບການ-deploy)
9. [ເບິ່ງ URL ທີ່ Live](#9-ເບິ່ງ-url-ທີ່-live)
10. [ແກ້ໄຂບັນຫາທົ່ວໄປ](#10-ແກ້ໄຂບັນຫາທົ່ວໄປ)

---

## 1. ກຽມເຄື່ອງມືທີ່ຕ້ອງການ

ກ່ອນ Deploy ຕ້ອງຕິດຕັ້ງໂຄງການເຫຼົ່ານີ້ໃນເຄື່ອງ:

### Node.js ແລະ npm
```bash
# ກວດສອບວ່າມີແລ້ວບໍ່
node --version   # ຕ້ອງ >= 18.0.0
npm --version    # ຕ້ອງ >= 8.0.0
```
ຖ້າຍັງບໍ່ມີ → ໂຫຼດຈາກ https://nodejs.org (ເລືອກ LTS version)

### AWS CLI
```bash
# ກວດສອບ
aws --version

# ຖ້າຍັງບໍ່ມີ — ໂຫຼດຈາກ:
# https://aws.amazon.com/cli/
# ຫຼື Windows: winget install Amazon.AWSCLI
```

### Git
```bash
git --version
# ຖ້າຍັງບໍ່ມີ → https://git-scm.com/downloads
```

### Amplify CLI (ampx)
```bash
# ຕິດຕັ້ງ (ມາພ້ອມ package ແລ້ວ ບໍ່ຕ້ອງ install global)
npm install   # run ໃນໂຟລເດີໂປຣເຈັກ

# ກວດສອບ
npx ampx --version
```

---

## 2. ກຽມ AWS Account

### 2.1 ສ້າງ AWS Account (ຖ້າຍັງບໍ່ມີ)
1. ໄປທີ່ https://aws.amazon.com
2. ກົດ **Create an AWS Account**
3. ໃສ່ Email, Password, Account name
4. ໃສ່ Credit Card (ໃຊ້ Free Tier ບໍ່ເສຍຄ່າ 12 ເດືອນ)
5. Verify ໂທລະສັບ

### 2.2 ສ້າງ IAM User ສຳລັບ Deploy

> **ເປັນຫຍັງ?** ບໍ່ຄວນໃຊ້ root account ໂດຍກົງ — ສ້າງ user ທີ່ມີສິດສະເພາະ

1. ເຂົ້າ AWS Console → ຄົ້ນຫາ **IAM**
2. ກົດ **Users** → **Create user**
3. ຕັ້ງຊື່: `amplify-deploy-user`
4. ກົດ **Next** → ເລືອກ **Attach policies directly**
5. ຄົ້ນຫາ ແລະ ເລືອກ policies ເຫຼົ່ານີ້:
   - `AdministratorAccess-Amplify`
   - `AWSCloudFormationFullAccess`
   - `IAMFullAccess`
   - `AmazonDynamoDBFullAccess`
   - `AmazonCognitoFullAccess`
6. ກົດ **Create user**

### 2.3 ສ້າງ Access Keys
1. ກົດເຂົ້າ user `amplify-deploy-user`
2. ກົດ tab **Security credentials**
3. ກົດ **Create access key**
4. ເລືອກ **Command Line Interface (CLI)**
5. **ສຳຄັນ:** ກົດ **Download .csv** — ເກັບໄວ້ ຫາຍແລ້ວສ້າງໃໝ່ບໍ່ໄດ້!
6. ຈົດ:
   - `Access key ID`: ຄ້າຍ `AKIAIOSFODNN7EXAMPLE`
   - `Secret access key`: ຄ້າຍ `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`

### 2.4 ຕັ້ງຄ່າ AWS CLI ໃນເຄື່ອງ
```bash
aws configure
# AWS Access Key ID: [ໃສ່ Access key ID]
# AWS Secret Access Key: [ໃສ່ Secret access key]
# Default region name: ap-southeast-1   ← Singapore (ໃກ້ ລາວ ທີ່ສຸດ)
# Default output format: json
```

ກວດສອບ:
```bash
aws sts get-caller-identity
# ຕ້ອງໄດ້ຕອບ JSON ທີ່ມີ Account ID
```

---

## 3. ສ້າງ GitHub Repository

### 3.1 ສ້າງ Repo ໃໝ່
1. ໄປທີ່ https://github.com → ກົດ **New repository**
2. ຕັ້ງຊື່: `laoshop` (ຫຼືຊື່ທີ່ຕ້ອງການ)
3. ເລືອກ **Private** (ປອດໄພກວ່າ)
4. **ບໍ່ຕ້ອງ** tick Add README (ມີໂຄດຢູ່ແລ້ວ)
5. ກົດ **Create repository**

### 3.2 Push ໂຄດຂຶ້ນ GitHub

ເປີດ Terminal ໃນໂຟລເດີ `Shop`:

```bash
# ກວດສອບວ່າ git ຕັ້ງຄ່າຊື່ user ແລ້ວ
git config --global user.name "ຊື່ຂອງທ່ານ"
git config --global user.email "email@example.com"

# ກວດສອບ git status
git status

# ເພີ່ມໄຟລ໌ທັງໝົດ (node_modules ຖືກ exclude ໂດຍ .gitignore ແລ້ວ)
git add .

# Commit
git commit -m "initial commit: LaoShop with Amplify backend"

# ເຊື່ອມກັບ GitHub (copy URL ຈາກ GitHub)
git remote add origin https://github.com/YOUR_USERNAME/laoshop.git

# Push
git branch -M main
git push -u origin main
```

> ຖ້າ GitHub ຂໍ password → ໃຊ້ **Personal Access Token** ແທນ:  
> GitHub → Settings → Developer settings → Personal access tokens → Generate new token  
> ເລືອກ scope: `repo` → Generate → copy token → ໃຊ້ແທນ password

---

## 4. ເຊື່ອມ Amplify ກັບ GitHub

### 4.1 ເຂົ້າ AWS Amplify Console
1. ເຂົ້າ https://console.aws.amazon.com
2. ຄົ້ນຫາ **AWS Amplify** → ກົດ **Amplify**
3. ຕ້ອງໃຫ້ Region ຖືກ: ເລືອກ **Asia Pacific (Singapore) ap-southeast-1**

### 4.2 ສ້າງ App ໃໝ່
1. ກົດ **Create new app**
2. ເລືອກ **GitHub** → ກົດ **Next**
3. ກົດ **Authorize AWS Amplify** — GitHub ຈະຂໍ permission
4. ເລືອກ Repository: `laoshop`
5. ເລືອກ Branch: `main`
6. ກົດ **Next**

### 4.3 ຈົດ App ID
ຫຼັງສ້າງ app ສຳເລັດ ທ່ານຈະເຫັນ URL ຄ້າຍນີ້:
```
https://ap-southeast-1.console.aws.amazon.com/amplify/apps/d1234abcd5678
                                                                  ↑
                                                          App ID = d1234abcd5678
```
**ຈົດ App ID ໄວ້** — ຈຳເປັນໃນຂັ້ນຕອນຕໍ່ໄປ

---

## 5. ຕັ້ງຄ່າ Build Settings

### 5.1 ສ້າງ amplify.yml

ສ້າງໄຟລ໌ `amplify.yml` ໃນ root ຂອງໂປຣເຈັກ:

```bash
# ໃນໂຟລເດີ Shop
```

**ເນື້ອໃນ `amplify.yml`:**
```yaml
version: 1
backend:
  phases:
    build:
      commands:
        - npm ci --cache .npm --prefer-offline
        - npx ampx pipeline-deploy --branch $AWS_BRANCH --app-id $AWS_APP_ID
frontend:
  phases:
    build:
      commands:
        - echo "Static site — no build needed"
  artifacts:
    baseDirectory: /
    files:
      - index.html
      - "**/*.css"
      - "**/*.js"
      - "**/*.png"
      - "**/*.jpg"
  cache:
    paths:
      - .npm/**/*
      - node_modules/**/*
```

> **ຄຳອະທິບາຍ:**
> - `backend.phases.build` — Deploy Amplify backend (Auth + DynamoDB) ກ່ອນ
> - `frontend.artifacts.baseDirectory: /` — ຟາກ root ເພາະ `index.html` ຢູ່ໃນ root
> - `files: index.html` — ບອກ Amplify ວ່າ static files ແມ່ນຫຍັງ
> - `cache.paths` — Cache node_modules ໃຫ້ build ໄວຂຶ້ນໃນຄັ້ງຕໍ່ໄປ

### 5.2 ກວດສອບ Build Settings ໃນ Console

1. ໃນ Amplify Console → ກົດ app ຂອງທ່ານ
2. ກົດ **App settings** → **Build settings**
3. ກວດໃຫ້ແນ່ໃຈວ່າ Build spec ຖືກ (ຫຼືເລືອກ "Use a YML file in my repository")

---

## 6. ຕັ້ງຄ່າ IAM Role ສຳລັບ CI/CD

> **ເປັນຫຍັງ?** GitHub Actions ຕ້ອງການ permission ເພື່ອ deploy ກັບ AWS  
> ການໃຊ້ **OIDC Role** ປອດໄພກວ່າການໃຊ້ static Access Keys

### ວິທີທີ 1: OIDC (ແນະນຳ — ປອດໄພກວ່າ)

#### ຂັ້ນທີ 1: ສ້າງ OIDC Provider ໃນ AWS
1. ໄປທີ່ **IAM** → **Identity providers** → **Add provider**
2. ເລືອກ **OpenID Connect**
3. ຕັ້ງຄ່າ:
   - Provider URL: `https://token.actions.githubusercontent.com`
   - Audience: `sts.amazonaws.com`
4. ກົດ **Add provider**

#### ຂັ້ນທີ 2: ສ້າງ IAM Role
1. **IAM** → **Roles** → **Create role**
2. ເລືອກ **Web identity**
3. Identity provider: `token.actions.githubusercontent.com`
4. Audience: `sts.amazonaws.com`
5. ກົດ **Next**
6. Attach policies:
   - `AdministratorAccess-Amplify`
   - `AWSCloudFormationFullAccess`
   - `IAMFullAccess`
   - `AmazonDynamoDBFullAccess`
   - `AmazonCognitoFullAccess`
7. ຕັ້ງຊື່ Role: `GitHubActions-AmplifyDeploy`
8. ກົດ **Create role**

#### ຂັ້ນທີ 3: ຈຳກັດ Trust Policy
ກົດເຂົ້າ Role ທີ່ສ້າງ → **Trust relationships** → **Edit trust policy**

ແກ້ Condition ໃຫ້ຊີ້ repo ຂອງທ່ານ:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/laoshop:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

> ແທນ `YOUR_ACCOUNT_ID` ດ້ວຍ AWS Account ID ຂອງທ່ານ (ຫາໄດ້ຈາກ `aws sts get-caller-identity`)  
> ແທນ `YOUR_GITHUB_USERNAME` ດ້ວຍ username GitHub ຂອງທ່ານ

**ຈົດ Role ARN:** ຄ້າຍ `arn:aws:iam::123456789012:role/GitHubActions-AmplifyDeploy`

---

### ວິທີທີ 2: Access Keys (ງ່າຍກວ່າ ແຕ່ security ຕ່ຳກວ່າ)

ຖ້າຕ້ອງການ deploy ໄວ ໃຊ້ Access Keys ທີ່ສ້າງໃນ [ຂັ້ນຕອນທີ 2.3](#23-ສ້າງ-access-keys) ໂດຍກົງ  
(ຂ້າມຂັ້ນຕອນ OIDC ທັງໝົດ ແລ້ວໄປ [ຕັ້ງ Secrets](#7-ຕັ້ງຄ່າ-github-secrets) ໂດຍໃຊ້ method ທີ 2)

---

## 7. ຕັ້ງຄ່າ GitHub Secrets

Secrets ແມ່ນ ຄ່າທີ່ລັບ ທີ່ GitHub ເກັບໃຫ້ ແລ້ວສ່ງໃຫ້ workflow ໃນຕອນ run

### ວິທີຕັ້ງ Secrets
1. ເຂົ້າ GitHub repository ຂອງທ່ານ
2. ກົດ **Settings** (icon gear ດ້ານເທິງ)
3. ກົດ **Secrets and variables** → **Actions**
4. ກົດ **New repository secret**

### Secrets ທີ່ຕ້ອງຕັ້ງ

#### ກໍລະນີ OIDC (ວິທີທີ 1):

| Secret Name | ຄ່າ | ຕົວຢ່າງ |
|-------------|-----|---------|
| `AWS_DEPLOY_ROLE_ARN` | ARN ຂອງ Role ທີ່ສ້າງ | `arn:aws:iam::123456789012:role/GitHubActions-AmplifyDeploy` |
| `AWS_REGION` | Region ທີ່ deploy | `ap-southeast-1` |
| `AMPLIFY_APP_ID` | App ID ຈາກ Amplify Console | `d1234abcd5678` |

#### ກໍລະນີ Access Keys (ວິທີທີ 2):

ແກ້ `.github/workflows/deploy.yml` ຕອນ Configure AWS credentials:
```yaml
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
```

ຈາກນັ້ນຕັ້ງ Secrets:

| Secret Name | ຄ່າ |
|-------------|-----|
| `AWS_ACCESS_KEY_ID` | Access key ID ຈາກ .csv |
| `AWS_SECRET_ACCESS_KEY` | Secret access key ຈາກ .csv |
| `AWS_REGION` | `ap-southeast-1` |
| `AMPLIFY_APP_ID` | App ID ຈາກ Amplify Console |

---

## 8. ທົດສອບການ Deploy

### 8.1 Trigger Pipeline ຄັ້ງທຳອິດ

```bash
# ໃນ terminal ທ້ອງຖິ່ນ — ແກ້ໄຂໄຟລ໌ຫຍັງໜຶ່ງ ຫຼືສ້າງ amplify.yml
git add amplify.yml
git commit -m "add amplify.yml build config"
git push origin main
```

### 8.2 ເບິ່ງ Status ໃນ GitHub Actions
1. GitHub repo → **Actions** tab
2. ຈະເຫັນ workflow ສອງອັນ:
   - **CI** — ກວດສອບ TypeScript
   - **Deploy to AWS Amplify** — deploy backend
3. ກົດເຂົ້າ deploy workflow → ເບິ່ງ logs ໃນ real-time

```
✅ Checkout                  — clone code
✅ Setup Node.js             — ຕິດຕັ້ງ Node
✅ Install dependencies      — npm ci
✅ Configure AWS credentials — login AWS
✅ Deploy Amplify backend     — deploy Auth + DynamoDB
```

### 8.3 ເບິ່ງ Status ໃນ Amplify Console
1. **AWS Amplify** Console → ກົດ app ຂອງທ່ານ
2. ທ່ານຈະເຫັນ build ກຳລັງ run:
   ```
   Provision  →  Build  →  Deploy  →  Verify
   ```
3. ໃຊ້ເວລາ ~5-10 ນາທີ ໃນຄັ້ງທຳອິດ

---

## 9. ເບິ່ງ URL ທີ່ Live

ຫຼັງ deploy ສຳເລັດ:

### URL ຂອງ App
1. **Amplify Console** → App ຂອງທ່ານ
2. ດ້ານເທິງຈະເຫັນ URL:
   ```
   https://main.d1234abcd5678.amplifyapp.com
   ```
3. ກົດ URL — ຈະເຫັນ LaoShop live!

### ເຊື່ອມ Custom Domain (ທາງເລືອກ)
ຖ້າຕ້ອງການ domain ຂອງຕົນເອງ ເຊັ່ນ `laoshop.com`:
1. **App settings** → **Custom domains** → **Add domain**
2. ໃສ່ domain ຂອງທ່ານ
3. Amplify ຈະ guide ໃຫ້ຕັ້ງ DNS records

---

## 10. ແກ້ໄຂບັນຫາທົ່ວໄປ

### ❌ "npm ci" ລົ້ມເຫຼວ
**ສາເຫດ:** `package-lock.json` ຍັງບໍ່ທັນມີ  
**ວິທີແກ້:**
```bash
npm install   # ສ້າງ package-lock.json
git add package-lock.json
git commit -m "add package-lock.json"
git push
```

### ❌ "credentials not found" ຫຼື "Access Denied"
**ສາເຫດ:** Secrets ຕັ້ງຜິດ ຫຼື IAM policy ບໍ່ພຽງພໍ  
**ວິທີແກ້:**
1. ກວດ Secrets ທີ່ GitHub → Settings → Secrets — ໃຫ້ຊື່ຖືກ
2. ກວດ Role ARN ວ່າ copy ຄົບ
3. ກວດ Trust Policy ໃຫ້ username/repo ຖືກ

### ❌ "amplify app not found"
**ສາເຫດ:** `AMPLIFY_APP_ID` ຜິດ  
**ວິທີແກ້:** ກວດ App ID ໃນ Amplify Console URL

### ❌ TypeScript error ໃນ CI
**ສາເຫດ:** Code TypeScript ມີ type error  
**ວິທີແກ້:**
```bash
# ທົດສອບ locally ກ່ອນ
npx tsc --noEmit -p amplify/tsconfig.json
# ແກ້ error ທັງໝົດ ຈາກນັ້ນ push ໃໝ່
```

### ❌ Build ໃຊ້ເວລານານ
**ສາເຫດ:** node_modules ບໍ່ຖືກ cache  
**ວິທີແກ້:** ກວດ `cache:` section ໃນ `amplify.yml`

---

## ສຳຄັນ — Security Checklist

- [ ] `node_modules/` ຢູ່ໃນ `.gitignore` ✅ (ມີຢູ່ແລ້ວ)
- [ ] AWS Credentials ບໍ່ paste ໄວ້ໃນ code
- [ ] Secrets ຕັ້ງໃນ GitHub Secrets ເທົ່ານັ້ນ
- [ ] `amplify_outputs.json` ຢູ່ໃນ `.gitignore` ✅ (ມີຢູ່ແລ້ວ)
- [ ] IAM Role ມີ least privilege (ສິດທີ່ຈຳເປັນເທົ່ານັ້ນ)

---

## Flow ສຸດທ້າຍ

```
ທ່ານ git push origin main
         │
         ▼
   GitHub Actions
   ┌─────────────┐
   │  CI Workflow │  ← ກວດ TypeScript
   └─────────────┘
         │ ✅
         ▼
   ┌──────────────────┐
   │  Deploy Workflow  │  ← Deploy backend ກັບ AWS
   └──────────────────┘
         │ ✅
         ▼
   Amplify Console
   ┌─────────────────────────┐
   │  Build Frontend + Verify │  ← Deploy index.html
   └─────────────────────────┘
         │ ✅
         ▼
   🌐 https://main.dXXXX.amplifyapp.com
   LaoShop Live!
```

---

*ຖ້າຕິດບັນຫາ ກວດ logs ໃນ GitHub Actions ຫຼື Amplify Console ກ່ອນ — ສ່ວນໃຫຍ່ແຈ້ງ error ທີ່ຊັດເຈນ*
