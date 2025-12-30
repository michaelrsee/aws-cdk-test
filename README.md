
# aws-cdk-test

Simple AWS CDK (TypeScript) project that deploys a minimal AWS Lambda function. This repository demonstrates how to build and deploy the Lambda using the AWS CDK and how to run deployments from GitHub Actions.

The `cdk.json` file tells the CDK Toolkit how to execute the app.

Reference: https://docs.aws.amazon.com/cdk/v2/guide/hello-world.html

## Prerequisites

- **Node.js**: v16+ recommended
- **AWS CDK v2** installed (`npm install -g aws-cdk` or use `npx`)
- **AWS credentials** configured locally (or provided via GitHub Secrets for CI)

## Useful commands

- `npm install`     install dependencies
- `npm run build`   compile TypeScript to JavaScript
- `npm run watch`   watch for changes and compile
- `npm run test`    run jest unit tests
- `npx cdk synth`   emit the synthesized CloudFormation template
- `npx cdk deploy`  deploy the stack to your AWS account/region
- `npx cdk diff`    compare deployed stack with current state

## Quickstart (local)

1. Install deps: `npm install`
2. Build: `npm run build`
3. Synthesize: `npx cdk synth`
4. Deploy: `npx cdk deploy`

## GitHub Actions (CI/CD) — use GitHub OIDC

Prefer GitHub OIDC to avoid long-lived AWS keys in repository secrets. With OIDC, GitHub Actions can assume an IAM role in your AWS account using short-lived tokens. High-level steps:

- Create an IAM OIDC identity provider for `token.actions.githubusercontent.com` in your AWS account.
- Create an IAM role with a trust policy that allows the GitHub OIDC provider to assume the role for your repository or organization. Constrain the role to the repository and required `sub`/`aud` conditions for least privilege.
- Grant the role the minimal permissions required to run CDK deploy (for example, `CDK bootstrap`/CloudFormation and Lambda permissions, or scoped to a deployment role).

Example GitHub Actions job snippet (store only the role ARN and region as repo secrets; do NOT store AWS access keys):

```yaml
name: Deploy
on:
	push:
		branches: [ main ]
jobs:
	deploy:
		runs-on: ubuntu-latest
		permissions:
			id-token: write
			contents: read
		steps:
			- uses: actions/checkout@v4
			- name: Configure AWS credentials (OIDC)
				uses: aws-actions/configure-aws-credentials@v2
				with:
					role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
					aws-region: ${{ secrets.AWS_REGION }}
			- name: Install dependencies
				run: npm ci
			- name: Build
				run: npm run build
			- name: CDK Deploy
				run: npx cdk deploy --require-approval=never
```

Notes:

- `AWS_ROLE_TO_ASSUME` should be the role ARN you created for GitHub OIDC (for example, `arn:aws:iam::123456789012:role/GitHubActionsCdkRole`). Storing the role ARN in a repo secret is fine; it is not a secret credential like access keys.
- The workflow requires `permissions: id-token: write` to request the OIDC token.
- Do not store `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` in GitHub secrets when using OIDC.
- For instructions on creating the OIDC provider and role trust policy, see the AWS docs and GitHub's OIDC documentation. Test the workflow in a non-production account first.

## Notes

- Keep AWS credentials secure in GitHub Secrets.
- For more information on CDK TypeScript projects see the AWS CDK hello-world guide: https://docs.aws.amazon.com/cdk/v2/guide/hello-world.html
