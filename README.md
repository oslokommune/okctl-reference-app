# Running locally

Run:

```shell
make run
```

Test that the application works:
```shell
curl http://localhost:8080
```

Run tests
```shell
./gradlew test
```

# Run in okctl cluster

See: [Documentation](https://www.okctl.io/set-up-a-reference-application-full-example/)

Example iac-repo [here](https://github.com/oslokommune/okctl-reference-iac)

# Configuration

## Terraform
See [documentation](tf/README.md) for configuration of Github OIDC connector and IAM roles in your AWS account.

## .gihub/workflows/docker-build-push-dev.yaml variables
The `.github/workflows/docker-build-push-dev.yaml` uses a OIDC setup for allowing github to push images to AWS ECR.

OIDC is used because we want to:
* Avoid use of long-lived access keys, rather use one-time tokens between github and aws
* Don't authenticate via a user (by using AWS access keys)
* Good control on `assume-role` in terms of what a role have access to and can do in your account

If you have other needs in terms of pushing resources to AWS: update the `build-and-push` step (and the corresponding role in terraform)

### Variables to update in .gihub/workflows/docker-build-push-\*.yaml
* `branches`: on which branch(es) do you want to push the image to ECR
* `aws-region`: the location of your infrastructure
* `ECR_REPOSITORY`: Name of ECR repository in AWS, created by okctl application
* `repository`: The {organization}/{okctl-config} repo containing your okctl IAC code
* `ref`: branch name you are using (main/master) for your IAC repo
* `DEPLOYMENT_YAML_FILE`: path to your application deployment patch
* The git commit message at the end of `jobs.update-tag`

#### Variables in okct-reference-app repo
The following secrets are here named after environment (DEV) and the application it is connected to (REFERENCE_APP)
* `secrets.DEV_REFERENCE_APP_ECR_PUSH_ROLE` - environment secret
    * The full ARN for the role to assume when pushing the image, generated by terraform
    * ex: `arn:aws:iam::123456789:role/github_actions/okctl-reference-app-github-ecr-push`
* `secrets.CLUSTER_DEPLOY_KEY` - repository secret
    * See: https://docs.github.com/en/developers/overview/managing-deploy-keys
    * See: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key
    * Note: keypair must be passwordless in order for the process to work
    * Add public key for CLUSTER_DEPLOY_KEY to your IAC repo as a deploy key, remember to tick the "write access" box
    * Add private key for CLUSTER_DEPLOY_KEY to your (this) okctl-reference-app repo as a repository secret
