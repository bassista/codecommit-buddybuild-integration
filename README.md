# CodeCommit - BuddyBuild Integration
This is an AWS Lambda that allow to integrate CodeCommit Repository to BuddyBuild CI

## Requirements

- docker, docker-compose
- BuddyBuild account
- AWS account
- CodeCommit repository


## Configure CodeCommit Repository on BuddyBuild

1. **On BuddyBuild page:** Create a new project from ssh repository by clicking `Add it with SSH!` action
2. **On BuddyBuild page:** Insert CodeCommit ssh repo in the `git clone URL` like this:
   ```
   ssh://git-codecommit.eu-west-1.amazonaws.com/v1/repos/test-codecommit-repository
   ```
3. **On BuddyBuild page:** Copy ssh public key provided by BuddyBuild.
4. **On AWS page:** Create IAM user Only to permit BuddyBuild to read the Repository. You have to give CodeCommit ReadOnly Access to the user. You can call it something like `BuddyBuildIntegration`
5. **On AWS page:** Go to `Security credentials` of the created `BuddyBuildIntegration` user
6. **On AWS page:** Click on `upload SSH public key` and insert ssh public key copied from the BuddyBuild page (on point [**3.**])
7. **On AWS page:** Copy the generated `SSH key ID`
8. **On BuddyBuild page:** Append just after `ssh://` the copied `SSH key ID` in this way:
   ```
   ssh://[SSH_KEY_ID]@git-codecommit.eu-west-1.amazonaws.com/v1/repos/test-codecommit-repository
   ```
9. **On BuddyBuild page:** Click start build app on BuddyBuild

### Get BuddyBuild tokens:

- Go to `myProfile -> access_token`
- copy the access_token
- copy the app id from url:
  ```
  https://dashboard.buddybuild.com/apps/[APP-ID-YOU-HAVE-TO-COPY]?page=1
  ```

### Set up the AWS Credentials to deploy Lambda
You need an account with Administration Access and use its AWS credentials to deploy the Lambda.

See [Servelress Framework Guide](https://serverless.com/framework/docs/providers/aws/guide/credentials/#creating-aws-access-keys)

## Setup Integration

Copy `.env.template` to `.env` file and fill the fields with your AWS Admin Credentials, BuddyBuild tokens and other configuration variables

1. Create and start local environment
   ```
   $ docker-compose up -d
   ```
2. Install dependencies
   ```
   $ docker-compose exec codecommit-buddybuild npm install
   ```
3. Deploy the configured Lambda on your AWS account for a specific stage (in this example is `prod`)
   ```
   $ docker-compose exec codecommit-buddybuild serverless deploy --stage prod
   ```

**NB:** On each `.env` change, you have to reboot the container with:
```
$ docker-compose up -d
```

### Configure Trigger from CodeCommit to the new Lambda

- Go to `Services -> Lambda`
- Select `codecommit-buddybuild-integration` Lambda
- Go to `triggers` tabs
- Click `Add trigger`
- Select `CodeCommit` trigger
- Choose:
   - Repository Name
   - Trigger name
   - event that triggers the lambda
   - which branch trigger the lambda

### Try the integration and see the Logs
Now you can try to push a commit to the target CodeCommit repository and see the logs with this command:
```
docker-compose exec codecommit-buddybuild serverless logs --function launchBuild --stage prod --startTime 1m -t
```