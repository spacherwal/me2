---
name: bitbucket-codedeploy-cicd
description: >
  Complete guide to set up and debug a CI/CD pipeline using Bitbucket Pipelines,
  AWS S3, AWS CodeDeploy, and EC2 Ubuntu. Use this skill when setting up automated
  deployments for a Next.js app (or any Node.js app) from Bitbucket to an EC2 server,
  debugging CodeDeploy lifecycle failures, fixing npm lock file mismatches, or
  onboarding a fresh EC2 instance to an existing deployment group.
  Covers full flow: IAM setup → S3 bucket → CodeDeploy app → agent install →
  Bitbucket pipeline → deployment scripts → Slack notifications → known failure patterns.
type: skill
---


================================================================================
CI/CD PIPELINE SETUP & DEBUGGING GUIDE
Bitbucket Pipelines → S3 → AWS CodeDeploy → EC2 Ubuntu
================================================================================

Written for: QA / DevOps engineers
App:         Next.js (livguard-ecomm)
Server:      Ubuntu 24.04 LTS on AWS EC2 (ap-south-1)
Branch:      release → auto deploy to QA


--------------------------------------------------------------------------------
ARCHITECTURE
--------------------------------------------------------------------------------

    Push to release branch
            |
    Bitbucket Pipeline
            |
      Step 1: Package source code → zip
            |
      Step 2: Upload zip → S3 bucket
            |
      Step 3: Trigger AWS CodeDeploy
            |
      CodeDeploy pulls zip from S3
            |
      Runs lifecycle scripts on EC2:
        ApplicationStop   → stop_app.sh
        AfterInstall      → install_deps.sh
        ApplicationStart  → start_app.sh
        ValidateService   → validate.sh
            |
      Slack notification (success or failure)


--------------------------------------------------------------------------------
PORT REFERENCE (for this project)
--------------------------------------------------------------------------------

    Port 8080   Next.js app (PM2)
    Port 80     Nginx
    Port 443    Nginx HTTPS


--------------------------------------------------------------------------------
PHASE 1 — AWS PREREQUISITES
--------------------------------------------------------------------------------

STEP 1 — Create S3 Bucket

    AWS Console → S3 → Create Bucket
    Name:                livguard-ecomm-deployments  (must be globally unique)
    Region:              ap-south-1
    Block public access: ON
    Click Create


STEP 2 — Create IAM User for Bitbucket

    AWS Console → IAM → Users → Create User
    Name: bitbucket-deployer

    Attach this inline policy:
    ----------------------------------------
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
          "Resource": [
            "arn:aws:s3:::livguard-ecomm-deployments",
            "arn:aws:s3:::livguard-ecomm-deployments/*"
          ]
        },
        {
          "Effect": "Allow",
          "Action": [
            "codedeploy:CreateDeployment",
            "codedeploy:GetDeployment",
            "codedeploy:GetDeploymentConfig",
            "codedeploy:RegisterApplicationRevision",
            "codedeploy:GetApplicationRevision"
          ],
          "Resource": "*"
        }
      ]
    }
    ----------------------------------------

    After creating user:
    Go to Security Credentials → Create Access Key
    Save Access Key ID and Secret Access Key for Bitbucket variables


STEP 3 — Create IAM Role for EC2

    AWS Console → IAM → Roles → Create Role
    Trusted entity: EC2
    Attach policies:
        AmazonS3ReadOnlyAccess
        AWSCodeDeployFullAccess
    Name: livguard-qa-ec2-role

    Attach to EC2:
    EC2 → Instance → Actions → Security → Modify IAM Role → select above role

    Verify role is attached (run on server):
        TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
          -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
        curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
          http://169.254.169.254/latest/meta-data/iam/info
    Should return JSON with InstanceProfileArn. If 404 = no role attached.


STEP 4 — Create CodeDeploy Application

    AWS Console → CodeDeploy → Applications → Create
    Name:     livguard-ecomm
    Platform: EC2/On-premises


STEP 5 — Create Deployment Group

    Inside CodeDeploy app → Create Deployment Group
    Name:              livguard-qa-group
    Service Role:      create role with AWSCodeDeployRole policy
    Deployment type:   In-place
    Environment:       EC2 Instances
    Tag Key:           Name
    Tag Value:         lipl-qa-ubuntu   ← MUST match EC2 tag exactly (case sensitive)
    Deployment config: CodeDeployDefault.AllAtOnce
    Load Balancer:     uncheck unless needed

    IMPORTANT: Tag value must match EC2 exactly.
               "lipl-qa-Ubuntu" != "lipl-qa-ubuntu" and will cause failures.


STEP 6 — Install CodeDeploy Agent on EC2

    Run on server (from any directory):

        sudo apt update
        sudo apt install ruby-full wget -y
        cd /home/ubuntu
        wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
        chmod +x ./install
        sudo ./install auto
        sudo systemctl start codedeploy-agent
        sudo systemctl enable codedeploy-agent

    Verify agent is running:
        sudo systemctl status codedeploy-agent

    Check agent logs:
        sudo tail -100 /var/log/aws/codedeploy-agent/codedeploy-agent.log

    Healthy agent log looks like:
        [codedeploy-agent]: poll_host_command returning 200
    This means agent is talking to CodeDeploy successfully.


--------------------------------------------------------------------------------
PHASE 2 — SLACK WEBHOOK SETUP
--------------------------------------------------------------------------------

    1. Go to https://api.slack.com/apps
    2. Create New App → From scratch
    3. Name: Livguard Deployments, pick workspace
    4. Go to Incoming Webhooks → Activate → Add New Webhook to Workspace
    5. Choose channel (e.g. #deployments)
    6. Copy the Webhook URL:
       https://hooks.slack.com/services/T00000/B00000/XXXXXXX


--------------------------------------------------------------------------------
PHASE 3 — BITBUCKET SETUP
--------------------------------------------------------------------------------

STEP 7 — Enable Pipelines

    Bitbucket → Repository Settings → Pipelines → Enable


STEP 8 — Create Deployment Environment

    Bitbucket → Repository Settings → Deployments → Add Environment
    Name: Staging  (must match deployment: value in yml exactly)


STEP 9 — Add Repository Variables

    Bitbucket → Repository Settings → Pipelines → Repository Variables

    Variable                Value                           Secured
    -------------------     ---------------------------     -------
    AWS_ACCESS_KEY_ID       from IAM user (Step 2)          YES
    AWS_SECRET_ACCESS_KEY   from IAM user (Step 2)          YES
    AWS_DEFAULT_REGION      ap-south-1                      NO
    S3_BUCKET               livguard-ecomm-deployments      NO
    CODEDEPLOY_APP_NAME     livguard-ecomm                  NO
    CODEDEPLOY_GROUP_NAME   livguard-qa-group               NO
    SLACK_WEBHOOK_URL       from Slack setup                YES

    NOTE: API keys and tokens (API_URL, API_TOKEN etc.) stay ONLY on the server
          in .env.local — never put them in Bitbucket variables.
          The build runs on the server using the server's .env.local file.


--------------------------------------------------------------------------------
PHASE 4 — REPO FILE STRUCTURE
--------------------------------------------------------------------------------

Add these files to your repo root:

    your-repo/
    ├── appspec.yml               ← root level (REQUIRED by CodeDeploy)
    ├── bitbucket-pipelines.yml   ← root level
    └── scripts/
        ├── stop_app.sh
        ├── install_deps.sh
        ├── start_app.sh
        └── validate.sh

IMPORTANT: Always check NVM and Node paths on your server before writing scripts:
    which pm2             → gives exact PM2 binary path
    ls ~/.nvm/nvm.sh      → confirms NVM location
    node --version        → confirms Node version

For this server:
    NVM:  /root/.nvm/nvm.sh
    PM2:  /root/.nvm/versions/node/v25.5.0/bin/pm2
    NPM:  /root/.nvm/versions/node/v25.5.0/bin/npm
    Node: v25.5.0

CodeDeploy runs scripts in a restricted shell — NVM is NOT auto-loaded.
Always hardcode full binary paths in scripts. Never rely on $PATH or `which`.


--------------------------------------------------------------------------------
FILE: appspec.yml
--------------------------------------------------------------------------------

version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/livguard-ecomm
    overwrite: true

permissions:
  - object: /var/www/livguard-ecomm
    owner: root
    group: root
    mode: 755
    type:
      - directory
  - object: /var/www/livguard-ecomm/scripts
    owner: root
    group: root
    mode: 755
    type:
      - file

hooks:
  ApplicationStop:
    - location: scripts/stop_app.sh
      timeout: 60
      runas: root

  AfterInstall:
    - location: scripts/install_deps.sh
      timeout: 300
      runas: root

  ApplicationStart:
    - location: scripts/start_app.sh
      timeout: 120
      runas: root

  ValidateService:
    - location: scripts/validate.sh
      timeout: 60
      runas: root


--------------------------------------------------------------------------------
FILE: scripts/stop_app.sh
--------------------------------------------------------------------------------

#!/bin/bash

echo ">>> Stopping livguard-ecomm-qa PM2 process..."

export NVM_DIR="/root/.nvm"
export PATH="/root/.nvm/versions/node/v25.5.0/bin:$PATH"

PM2=/root/.nvm/versions/node/v25.5.0/bin/pm2

if $PM2 describe livguard-ecomm-qa > /dev/null 2>&1; then
    $PM2 stop livguard-ecomm-qa
    echo ">>> App stopped successfully."
else
    echo ">>> App not running, skipping stop."
fi

exit 0


--------------------------------------------------------------------------------
FILE: scripts/install_deps.sh
--------------------------------------------------------------------------------

#!/bin/bash

echo ">>> Starting install and build on server..."

export NVM_DIR="/root/.nvm"
export PATH="/root/.nvm/versions/node/v25.5.0/bin:$PATH"

NPM=/root/.nvm/versions/node/v25.5.0/bin/npm

cd /var/www/livguard-ecomm

if [ ! -f ".env.local" ]; then
    echo ">>> ERROR: .env.local not found at /var/www/livguard-ecomm/.env.local"
    echo ">>> Please create it manually on the server before deploying."
    exit 1
fi

echo ">>> .env.local found. Proceeding..."

echo ">>> Installing dependencies..."
$NPM install          # Use npm install NOT npm ci (see lessons learned below)

if [ $? -ne 0 ]; then
    echo ">>> ERROR: npm install failed!"
    exit 1
fi

echo ">>> Building Next.js app..."
$NPM run build

if [ $? -ne 0 ]; then
    echo ">>> ERROR: npm build failed!"
    exit 1
fi

echo ">>> Install and build completed successfully."
exit 0


--------------------------------------------------------------------------------
FILE: scripts/start_app.sh
--------------------------------------------------------------------------------

#!/bin/bash

echo ">>> Starting livguard-ecomm-qa..."

export NVM_DIR="/root/.nvm"
export PATH="/root/.nvm/versions/node/v25.5.0/bin:$PATH"

PM2=/root/.nvm/versions/node/v25.5.0/bin/pm2

cd /var/www/livguard-ecomm

if $PM2 describe livguard-ecomm-qa > /dev/null 2>&1; then
    echo ">>> Reloading existing PM2 process..."
    $PM2 reload livguard-ecomm-qa --update-env
else
    echo ">>> Starting new PM2 process..."
    $PM2 start node_modules/.bin/next \
        --name "livguard-ecomm-qa" \
        -- start -p 8080
fi

$PM2 save
echo ">>> App started successfully."
exit 0


--------------------------------------------------------------------------------
FILE: scripts/validate.sh
--------------------------------------------------------------------------------

#!/bin/bash

echo ">>> Validating deployment..."

sleep 15

HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080)

if [ "$HTTP_STATUS" -eq 200 ] || \
   [ "$HTTP_STATUS" -eq 301 ] || \
   [ "$HTTP_STATUS" -eq 302 ]; then
    echo ">>> Validation successful! HTTP status: $HTTP_STATUS"
    exit 0
else
    echo ">>> ERROR: App not responding! HTTP status: $HTTP_STATUS"
    exit 1
fi


--------------------------------------------------------------------------------
FILE: bitbucket-pipelines.yml
--------------------------------------------------------------------------------

NOTE: image is python:3.11 NOT amazon/aws-cli
      Reason: python:3.11 is Debian-based so apt-get works.
              amazon/aws-cli uses Amazon Linux (yum) which conflicts with Ubuntu workflow.
      awscli is installed via pip in each step.
      Each step is a FRESH container — tools do not carry over between steps.

----------------

image: node:20

definitions:
  steps:
    - step: &package-and-upload
        name: "Package Source & Upload to S3"
        image: python:3.11
        script:
          - |
            set -e

            echo ">>> Creating deployment package from source code..."
            apt-get update && apt-get install -y zip && pip install awscli --quiet
            DEPLOYMENT_ID="livguard-ecomm-$(date +%Y%m%d%H%M%S)-${BITBUCKET_BUILD_NUMBER}"

            zip -r "$DEPLOYMENT_ID.zip" . \
              -x "*.git*" \
              -x ".next/*" \
              -x "node_modules/*"

            echo ">>> Uploading to S3..."
            aws s3 cp "$DEPLOYMENT_ID.zip" "s3://$S3_BUCKET/deployments/$DEPLOYMENT_ID.zip"

            echo "DEPLOYMENT_ID=$DEPLOYMENT_ID" >> deployment.env
            echo ">>> Upload complete! Package: $DEPLOYMENT_ID.zip"
        artifacts:
          - deployment.env

    - step: &deploy
        name: "Deploy to QA via CodeDeploy"
        image: python:3.11
        deployment: Staging
        script:
          - |
            set -e
            pip install awscli --quiet
            source deployment.env
            echo ">>> Triggering CodeDeploy deployment for $DEPLOYMENT_ID..."

            DEPLOY_OUTPUT=$(aws deploy create-deployment \
              --application-name $CODEDEPLOY_APP_NAME \
              --deployment-group-name $CODEDEPLOY_GROUP_NAME \
              --s3-location bucket=$S3_BUCKET,key=deployments/$DEPLOYMENT_ID.zip,bundleType=zip \
              --deployment-config-name CodeDeployDefault.AllAtOnce \
              --description "Bitbucket build $BITBUCKET_BUILD_NUMBER - branch $BITBUCKET_BRANCH" \
              --ignore-application-stop-failures \
              --region $AWS_DEFAULT_REGION \
              --output json)

            CODEDEPLOY_ID=$(echo $DEPLOY_OUTPUT | python3 -c "import sys, json; print(json.load(sys.stdin)['deploymentId'])")
            echo ">>> Deployment ID: $CODEDEPLOY_ID"

            aws deploy wait deployment-successful \
              --deployment-id $CODEDEPLOY_ID \
              --region $AWS_DEFAULT_REGION

            echo ">>> Deployment successful!"
        after-script:
          - |
            if [ $BITBUCKET_EXIT_CODE -eq 0 ]; then
              curl -X POST -H 'Content-type: application/json' \
                --data "{\"attachments\": [{\"color\": \"#36a64f\",\"blocks\": [{\"type\": \"header\",\"text\": {\"type\": \"plain_text\",\"text\": \"✅ Deployment Successful\"}},{\"type\": \"section\",\"fields\": [{\"type\": \"mrkdwn\",\"text\": \"*App:*\nlivguard-ecomm\"},{\"type\": \"mrkdwn\",\"text\": \"*Branch:*\n$BITBUCKET_BRANCH\"},{\"type\": \"mrkdwn\",\"text\": \"*Build:*\n#$BITBUCKET_BUILD_NUMBER\"},{\"type\": \"mrkdwn\",\"text\": \"*Server:*\nqa.lockthedeal.com\"},{\"type\": \"mrkdwn\",\"text\": \"*Commit:*\n${BITBUCKET_COMMIT:0:7}\"}]},{\"type\": \"actions\",\"elements\": [{\"type\": \"button\",\"text\": {\"type\": \"plain_text\",\"text\": \"View Pipeline\"},\"url\": \"https://bitbucket.org/$BITBUCKET_REPO_FULL_NAME/pipelines/results/$BITBUCKET_BUILD_NUMBER\"}]}]}]}" \
                $SLACK_WEBHOOK_URL
            else
              curl -X POST -H 'Content-type: application/json' \
                --data "{\"attachments\": [{\"color\": \"#e01e5a\",\"blocks\": [{\"type\": \"header\",\"text\": {\"type\": \"plain_text\",\"text\": \"❌ Deployment Failed\"}},{\"type\": \"section\",\"fields\": [{\"type\": \"mrkdwn\",\"text\": \"*App:*\nlivguard-ecomm\"},{\"type\": \"mrkdwn\",\"text\": \"*Branch:*\n$BITBUCKET_BRANCH\"},{\"type\": \"mrkdwn\",\"text\": \"*Build:*\n#$BITBUCKET_BUILD_NUMBER\"},{\"type\": \"mrkdwn\",\"text\": \"*Failed at:*\nDeploy Stage\"},{\"type\": \"mrkdwn\",\"text\": \"*Commit:*\n${BITBUCKET_COMMIT:0:7}\"}]},{\"type\": \"actions\",\"elements\": [{\"type\": \"button\",\"text\": {\"type\": \"plain_text\",\"text\": \"View Logs\"},\"url\": \"https://bitbucket.org/$BITBUCKET_REPO_FULL_NAME/pipelines/results/$BITBUCKET_BUILD_NUMBER\"}]}]}]}" \
                $SLACK_WEBHOOK_URL
            fi

pipelines:
  branches:
    release:
      - step: *package-and-upload
      - step: *deploy
  custom:
    deploy-to-qa-manual:
      - step: *package-and-upload
      - step: *deploy


--------------------------------------------------------------------------------
LESSONS LEARNED — REAL FAILURES FROM APR 21 2026
--------------------------------------------------------------------------------

LESSON 1 — CodeDeploy Bootstrap Loop
--------------------------------------
Error:
    "CodeDeploy agent was not able to receive the lifecycle event"
    Failing at: ApplicationStop

Why it happens:
    During ApplicationStop, CodeDeploy runs scripts from the PREVIOUS
    deployment archive on disk. If that archive was cleaned up (e.g. first
    ever deployment, or after a streak of failed deployments), the directory
    no longer exists. Agent reports it can't receive the event — misleading
    wording for a missing prior archive.

Fix:
    Add --ignore-application-stop-failures to aws deploy create-deployment
    in bitbucket-pipelines.yml. Safe long-term — once a deployment succeeds,
    future ApplicationStop hooks run normally.

When to apply:
    - Fresh EC2 instance added to deployment group for first time
    - After a streak of failed deployments that were cleaned up


LESSON 2 — npm ci Fails on EC2
--------------------------------
Error:
    npm ci can only install packages when your package.json and
    package-lock.json are in sync.
    Missing: prom-client, @opentelemetry/api, tdigest, bintrees

Why it happens:
    Packages were added to package.json manually (or via npm install on
    server) without committing the updated package-lock.json to the repo.
    npm ci strictly requires them to be in sync.

Fix (short term):
    Use npm install instead of npm ci in install_deps.sh.

Fix (long term / team rule):
    NEVER commit package.json changes without also committing the
    regenerated package-lock.json in the same commit.
    Run: npm install → commit both files together.


LESSON 3 — NVM Not Loaded in CodeDeploy Shell
-----------------------------------------------
Error:
    Scripts work when run manually but fail under CodeDeploy.
    pm2: command not found
    npm: command not found

Why it happens:
    CodeDeploy runs lifecycle scripts in a restricted shell that does NOT
    load NVM or .bashrc. So pm2 and npm are not on PATH.

Fix:
    Hardcode full binary paths in every script. Never use `which pm2` or
    rely on PATH. Always use:
        PM2=/root/.nvm/versions/node/v25.5.0/bin/pm2
        NPM=/root/.nvm/versions/node/v25.5.0/bin/npm

Check your server's actual paths:
    which pm2     → copy exact output into scripts
    which npm     → copy exact output into scripts


LESSON 4 — Wrong Docker Image in Pipeline
-------------------------------------------
Error:
    bash: zip: command not found
    (when using amazon/aws-cli image)

Why it happens:
    amazon/aws-cli image uses Amazon Linux (yum package manager).
    zip is not pre-installed.
    apt-get does not work on it.

Fix:
    Use python:3.11 image instead. It is Debian-based so apt-get works.
    Install awscli via pip: pip install awscli --quiet


LESSON 5 — Variables Lost Between Pipeline Script Lines
---------------------------------------------------------
Error:
    DEPLOYMENT_ID is empty in aws s3 cp command

Why it happens:
    Each - line in Bitbucket pipeline script runs in a separate shell.
    Variables set on one line die before the next line runs.

Fix:
    Wrap all related commands in a single | block:
        - |
          DEPLOYMENT_ID="xyz"
          zip -r "$DEPLOYMENT_ID.zip" .    # works because same shell
          aws s3 cp "$DEPLOYMENT_ID.zip"   # works


LESSON 6 — Bitbucket Deployment Environment Mismatch
------------------------------------------------------
Error:
    The environment 'qa' doesn't match any environment in your settings.

Why it happens:
    deployment: qa in yml but Bitbucket has no environment named 'qa'.

Fix:
    Either:
    A) Create environment in Bitbucket → Settings → Deployments → Add (name: qa)
    B) Change yml to match existing environment name (e.g. deployment: Staging)


--------------------------------------------------------------------------------
DIAGNOSTIC CHECKLIST — WHEN DEPLOYMENT FAILS
--------------------------------------------------------------------------------

Run in this order:

1. Is agent running?
       sudo systemctl status codedeploy-agent

2. What is agent doing?
       sudo tail -100 /var/log/aws/codedeploy-agent/codedeploy-agent.log
   Healthy: shows poll_host_command returning 200

3. Which lifecycle hook failed?
       AWS Console → CodeDeploy → Deployments → click Deployment ID
       → Deployment lifecycle events → click instance → View events → find RED event

4. Read exact script error:
       sudo cat $(ls -td /opt/codedeploy-agent/deployment-root/*/logs/scripts.log 2>/dev/null | head -1)
   Or:
       sudo tail -200 /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log

5. Test scripts manually on server:
       bash /var/www/livguard-ecomm/scripts/stop_app.sh
       bash /var/www/livguard-ecomm/scripts/install_deps.sh
       bash /var/www/livguard-ecomm/scripts/start_app.sh
       bash /var/www/livguard-ecomm/scripts/validate.sh
   Scripts should all pass manually before triggering pipeline.

6. Check IAM role is attached to EC2:
       TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
         -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
       curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
         http://169.254.169.254/latest/meta-data/iam/info
   Should return JSON with InstanceProfileArn. If 404 = no role.

7. Check EC2 tag matches deployment group exactly:
       AWS Console → EC2 → Instance → Tags tab
       Key: Name   Value: lipl-qa-ubuntu   (case sensitive)


--------------------------------------------------------------------------------
KNOWN GAPS TO IMPROVE IN FUTURE
--------------------------------------------------------------------------------

Gap                         Risk            Future Fix
------------------------    -----------     ----------------------------------
Build runs on EC2 not CI    Slow, no        Move npm build to Bitbucket pipeline
                            rollback if     with env vars stored securely
                            build fails

No auto-rollback            Failed deploy   Add --auto-rollback-configuration
                            stays live      to aws deploy create-deployment

validate.sh uses sleep 15   Fragile if      Replace with retry loop:
instead of retry loop       app starts      while ! curl localhost:8080; do
                            slow              sleep 3; done

AllAtOnce deployment        Risky if        Switch to CodeDeployDefault.HalfAtATime
config                      multiple EC2s   when scaling beyond 1 server

package-lock.json often     npm ci fails    Enforce team rule: always commit
out of sync with            on deploy       package-lock.json with package.json
package.json


--------------------------------------------------------------------------------
QUICK REFERENCE
--------------------------------------------------------------------------------

Check agent:        sudo systemctl status codedeploy-agent
Agent logs:         sudo tail -100 /var/log/aws/codedeploy-agent/codedeploy-agent.log
Script logs:        sudo cat $(ls -td /opt/codedeploy-agent/deployment-root/*/logs/scripts.log 2>/dev/null | head -1)
Restart agent:      sudo systemctl restart codedeploy-agent
Test stop script:   bash /var/www/livguard-ecomm/scripts/stop_app.sh
Test build script:  bash /var/www/livguard-ecomm/scripts/install_deps.sh
Test start script:  bash /var/www/livguard-ecomm/scripts/start_app.sh
Test validate:      bash /var/www/livguard-ecomm/scripts/validate.sh
PM2 path:           /root/.nvm/versions/node/v25.5.0/bin/pm2
NPM path:           /root/.nvm/versions/node/v25.5.0/bin/npm
App directory:      /var/www/livguard-ecomm
App port:           8080
Env file:           /var/www/livguard-ecomm/.env.local

================================================================================
END OF SKILL FILE
================================================================================