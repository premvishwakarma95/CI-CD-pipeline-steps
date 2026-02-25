# CI/CD Pipeline Steps to setup in github
- CI -> stands for `Continuously Integration` And CD -> stands for `Continuously Deployment` OR `(Continuous Delivery)`.
- In this Repositiory I have implemented all the things
- https://github.com/premvishwakarma95/simple-node-app-cicd

---

## 🚀 AUTO DEPLOY NODE APP TO AWS EC2 (BEGINNER MODE) Using CI/CD.
### We will use:
- `GitHub Actions` → automation
- `AWS EC2` → server
- `PM2` → keep Node app running

---

### 🧠 BIG PICTURE (1 minute)
```code
git push
   ↓
GitHub Actions
   ↓
SSH into EC2
   ↓
Pull latest code
   ↓
npm install
   ↓
pm2 restart app
```

---

### STEP 1️⃣ Prepare EC2 (DO THIS ON SERVER)
1. SSH into EC2
```bash
ssh ubuntu@<EC2_PUBLIC_IP>
```
2. Install Node.js
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
Check:
```bash
node -v
npm -v
```
3. Install PM2
```bash
sudo npm install -g pm2
```
4. Clone your repo ONCE
```bash
git clone https://github.com/premvishwakarma95/simple-node-app-cicd.git
cd simple-node-app-cicd
npm install
```
5. Start app with PM2
```bash
pm2 start index.js --name simple-node-app
pm2 save
pm2 status
```
✅ App is running on EC2 now.

---

### STEP 2️⃣ Setup SSH access for GitHub Actions (IMPORTANT) IF You have not added while creating EC2 Instance
1. On your LOCAL machine
Generate SSH key (if not already):
```bash
ssh-keygen -t rsa -b 4096
```
2. Copy public key to EC2
```bash
cat ~/.ssh/id_rsa.pub
```
On EC2:
```bash
nano ~/.ssh/authorized_keys
```
Paste key → save.
3. Copy PRIVATE key (important)
```bash
cat ~/.ssh/id_rsa
```
⚠️ Keep this safe — we’ll put it in GitHub Secrets.

---

### STEP 3️⃣ Add GitHub Secrets

In your GitHub repository:

1. Go to **Settings → Secrets and variables → Actions**
2. Click **New repository secret**
3. Add the following secrets:

| Name         | Value                               |
|--------------|-------------------------------------|
| EC2_HOST     | EC2 public IP                       |
| EC2_USER     | ubuntu                              |
| EC2_SSH_KEY  | SSH private key (keep secret)       |
| APP_PATH     | /home/ubuntu/simple-node-app-cicd   |

> ⚠️ **Important:**  
> Never commit your private SSH key to the repository.

---

### STEP 4️⃣ Create DEPLOY workflow
```code
.github/workflows/deploy.yml
```
Paste this 👇
```yaml
name: Deploy to EC2

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd ${{ secrets.APP_PATH }}
            git pull origin main
            npm install
            pm2 restart simple-node-app || pm2 start index.js --name simple-node-app
```

---

### STEP 5️⃣ Commit & Push
```bash
git add .github/workflows/deploy.yml
git commit -m "Add auto deploy to EC2"
git push
```
Now whatever we will push code in main or master branch all changes would automatically will be deployed.

---

### STEP 6️⃣ WATCH MAGIC ✨
1. GitHub → Actions
2. Click Deploy to EC2
3. Watch logs
4. Open:
```bash
http://<EC2_PUBLIC_IP>:3000
```
🎉 Your app is auto-deployed

---

### 🧠 What you just achieved (REAL SKILL)
You can now say: - “I implemented CI/CD using GitHub Actions with auto-deployment to AWS EC2 using PM2.”
- This is production-level knowledge.
