# Updated Instructions (2022-03-20)

The following was tested on a test k3s cluster [deployed using multipass](https://gist.github.com/nicc777/0f620c9eb2958f58173224f29b23a2ff)

A directory called `runtime` is also now in the `.gitignore`, and it is therefore a "safe place" to temporarily store sensitive information (credentials).

## Preparation

1. Clone this repo
2. `cd` into the cloned repo
3. Now run:

```shell
mkdir runtime

cp -vf templates/* runtime/

cd runtime/
```

From here the `runtime/` directory is assumed to be the current working directory.

## Prepare secrets

Secret values must be base64 encoded. Assuming the AWS credentials is in the environment variables `$AWS_ACCESS_KEY_ID` and `$AWS_SECRET_ACCESS_KEY` run the following to obtain the base64 values:

```shell
echo -n "$AWS_ACCESS_KEY_ID" | base64 -

echo -n "$AWS_SECRET_ACCESS_KEY" | base64 -
```

Update the the file `aws-secrets.yml` with the base64 values for `aws-access-key-id` and `aws-secret-access-key`

## Update Deployment

In the file `ecr-cron.yml` endit the environment variables to suite your environment.

Remember to also add the namespaces that will require the ECR credentials.

## Deploy

The following was tested with the namespace `infrastructure` - any namespace will do.

```shell
kubectl apply -f aws-secrets.yml -n infrastructure

kubectl apply -f aws-role.yml -n infrastructure

kubectl apply -f ecr-cron.yml -n infrastructure
```

To test that the secret key was correctly set:

```shell
echo "-->"`kubectl get secret aws-secrets -o jsonpath="{.data.aws-secret-access-key}" -n infrastructure | base64 -d`"<--"
```

_**Note**_: Ensure no newline is present with teh output.

## Force an initial Job Run

The following command will run the job:

```shell
kubectl create job --from=cronjob.batch/aws-registry-credential-cron aws-registry-credential-cron-001 -n infrastructure
```

To test if the job was successful, the following commands could aid:

```shell
kubectl get pods -n infrastructure

kubectl logs pod/aws-registry-credential-cron-XXXXXXXX--X-XXXXX -n infrastructure

kubectl describe secret aws-registry -n infrastructure
```

To get the full ECR config:

```shell 
kubectl get secret aws-registry -o jsonpath="{.data}" -n infrastructure
```

_**Note**_: The value in `.dockerconfigjson` is also base64 encoded. Decode with: `echo "__THE_VALUE__ | base64 -d"`

# OLD Readme from Original Repo

The Docker Image contains the aws-cli and kubectl. It is used to update the AWS ECR credentials periodically in a kubernetes cluster.

## Setup

You need to set your credentials in the aws-secrets.yml. Also you need to set your AWS_ACCOUNT, AWS_REGION and NAMESPACES in ecr-cron.yml.
Afterwords run: `aws.sh`

Afterwords you should be able to see the cron job with: `kubectl get cronjobs -n infrastructure`

## Thanks

This is based on the work of @xynova.

