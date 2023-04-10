# cicd_github_actions

The following code setups a CI-CD pipeline for EC2 (AWS) through Github Actions.
You can add a new step to run tests if you have any.

```
on:
push:
branches: ["main"]

jobs:
build:
runs-on: ubuntu-latest

    steps:
      - name: Clone Repo
        uses: actions/checkout@v3

      - name: Configure Node
        uses: actions/setup-node@v2
        with:
          node-version: "18.x"

      - name: Install Dependencies
        run: npm i
        working-directory: ./<CLIENT FOLDER>

      - name: Build Project
        run: CI=false npm run build
        working-directory: ./<CLIENT FOLDER>

      - name: Copy Build to EC2
        run: |
          echo "${{ secrets.EC2_PRIVATE_KEY }}" > key.pem
          chmod 400 key.pem
          scp -o StrictHostKeyChecking=no -i key.pem -r ./<BACKEND FOLDER>/ ubuntu@<ELASTIC IP ADDRESS>:~/
          scp -o StrictHostKeyChecking=no -i key.pem -r ./<CLIENT FOLDER>/build ubuntu@<ELASTIC IP ADDRESS>:~/<BACKEND FOLDER>/
      - name: Restart PM2
        uses: appleboy/ssh-action@master
        with:
          host: <ELASTIC IP ADDRESS>
          username: ubuntu
          port: 22
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script: |
            cd <BACKEND FOLDER>
            npm i
            pm2 restart all
      - name: Remove private key file
        run: rm key.pem

```

1. Replace following with your project's corresponding value.

   - **\<CLIENT FOLDER>**
   - **\<BACKEND FOLDER>**,
   - **\<ELASTIC IP ADDRESS>**

2. Make sure to add ec2 private key (pem file) content in a github secret named "EC2_PRIVATE_KEY" (without quotes).

3. The following project structure is considered

```
    <ROOT FOLDER>
    |
    |--- CLIENT FOLDER
    |--- BACKEND FOLDER
    |
```
