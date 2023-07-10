# umami

Umami is a simple, fast, privacy-focused alternative to Google Analytics.

# Umami on Bunny Shell

This project is a Next.js full-stack application that provides a privacy-focused alternative to Google Analytics. It uses PostgreSQL as the database and is deployed on Bunnyshell's Environment as a Service (EaaS) platform.

## Getting Started
To get started with the Umami application on Bunnyshell, follow these steps:

## Prerequisites
You need a Bunnyshell account. If you don't have one, sign up at Bunnyshell website.
Make sure you have Docker installed on your local machine.

## Deployment Steps
1. Fork or clone this repository to your local machine.

2. Create a Bunnyshell environment:
   - Login to your Bunnyshell account.
   - Navigate to the Bunnyshell UI and create a new environment.
   - Provide the following environment configuration:
### BunnyShell configuration yaml file: <a href="bunnyshell.yml">bunnyshell.yml</a>

```yml
kind: Environment
name: nextjs-aws-rds
type: primary
urlHandle: fjirhnsa
environmentVariables:
    DB_NAME: mydb
components:
    -
        kind: Terraform
        name: db-rds
        gitRepo: 'https://github.com/tush-tr/umami.git'
        gitBranch: terraform-aws-rds
        gitApplicationPath: /aws-rds-tf
        runnerImage: 'hashicorp/terraform:1.5.1'
        deploy:
            - 'cd aws-rds-tf'
            - '/bns/helpers/terraform/get_managed_backend > zz_backend_override.tf'
            - 'terraform init -input=false -no-color'
            - |
                terraform apply -input=false -auto-approve -no-color \
                  -var="prefix=bns{{ env.unique }}" \
                  -var="instance_class=db.t3.micro" \
                  -var="allocated_storage=20" \
                  -var="engine=postgres" \
                  -var="engine_version=15.3" \
                  -var="db_name={{ env.vars.DB_NAME }}" \
                  -var="username=bunnyshell" \
                  -var="db_port=5432" \
                  -var="vpc_cidr=10.0.0.0/16" \
                  -var='allowed_cidrs=["54.246.59.74/32", "34.83.72.16/32","0.0.0.0/0"]'\
                  -var="publicly_accessible=true"
            - 'BNS_TF_STATE_LIST=`terraform show -json`'
            - |
                DB_HOST=$(terraform output --raw rds_endpoint | awk -F: '{print $1}')
            - 'DB_RDS_NAME=`terraform output --raw rds_name`'
            - 'DB_PORT=`terraform output --raw rds_port`'
            - 'DB_USERNAME=`terraform output --raw rds_username`'
            - 'DB_PASSWORD=`terraform output --raw rds_password`'
        destroy:
            - 'cd components/tf-aws-rds'
            - '/bns/helpers/terraform/get_managed_backend > zz_backend_override.tf'
            - 'terraform init -input=false -no-color'
            - |
                terraform destroy -input=false -auto-approve -no-color \
                  -var="prefix=bns{{ env.unique }}" \
                  -var="instance_class=db.t3.micro" \
                  -var="allocated_storage=20" \
                  -var="engine=postgres" \
                  -var="engine_version=15.3" \
                  -var="db_name={{ env.vars.DB_NAME }}" \
                  -var="username=bunnyshell" \
                  -var="db_port=5432" \
                  -var="vpc_cidr=10.0.0.0/16" \
                  -var='allowed_cidrs=["54.246.59.74/32", "34.83.72.16/32"]'\
                  -var="publicly_accessible=true"
        exportVariables:
            - DB_HOST
            - DB_PORT
            - DB_USERNAME
            - DB_PASSWORD
            - DB_RDS_NAME
        environment:
            AWS_ACCESS_KEY_ID: AWS_ACCESS_KEY
            AWS_REGION: eu-north-1
            AWS_SECRET_ACCESS_KEY: AWS_SECRET_KEY
    -
        kind: Application
        name: app
        gitRepo: 'https://github.com/tush-tr/umami.git'
        gitBranch: master
        gitApplicationPath: /
        dockerCompose:
            image: 'ghcr.io/umami-software/umami:postgresql-latest'
            environment:
                APP_SECRET: 0Pg7YLrQwbJa5XZN
                DATABASE_TYPE: postgresql
                DATABASE_URL: 'postgresql://{{ components.db-rds.exported.DB_USERNAME }}:{{ components.db-rds.exported.DB_PASSWORD }}@{{ components.db-rds.exported.DB_HOST }}:5432/{{ components.db-rds.exported.DB_RDS_NAME }}'
                POSTGRES_SSL: '1'
            ports:
                - '3000:3000'
        hosts:
            -
                hostname: 'app-{{ env.base_domain }}'
                path: /
                servicePort: 3000
```

3. Save the environment configuration as bunnyshell.yaml in your public repository.

4. Deploy the application:
   - Commit and push the changes to your repository.
   - Bunnyshell will automatically detect the changes and trigger the deployment process.
   - Wait for the deployment to complete.
5. Access the Umami application:

   - Once the deployment is successful, you can access the Umami application at http://app-{{ env.base_domain }}:3000.
   - The application will use the PostgreSQL database provisioned by the Bunnyshell environment.

## Video Demonstration

<iframe width="560" height="315" src="https://www.youtube.com/embed/RI8D-nN9uwU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

