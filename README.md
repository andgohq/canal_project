# P\*R\*O\*P

## Prerequisites

- Docker & docker compose
  - https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
  - https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04
- npm
  - `curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`
  - `sudo apt install nodejs`

## Development 
### Frontend & Backend

1. Pull this repo
2. `cd PROP`
3. copy `.env.example` to `.env`
4. in `frontend/src/environments/environment.ts` set the env vars for development: 
   - set `production` to `false`
   - set `baseUrl` to `http://localhost:5004`
   - set `apiKey` to `DUMMY`
   - set `fileSizeLimit` to the file size limit
5. `./dc_script.sh`
6. wait for the script to finish
7. open the website on `https://localhost:4200/`
8. make changes to the frontend/backend code. Hot reload is enabled

NOTE:
- If you want to run the frontend/backend separately, you can do so by running `sudo ./dc_script.sh --frontend-only` or `sudo ./dc_script.sh --backend-only`

## Production
### Build
#### Frontend (static)

1. Pull this repo
2. `cd PROP`
3. Ensure the `.env` file in the root directory contains the following variables:
   - `PRODUCTION=true`
   - `BASE_URL` (set to the API Gateway URL)
   - `BACKEND_APIKEY` (set to the Python backend API key)
   - `FILE_SIZE_LIMIT` (set to the desired file size limit)
4. `cd frontend`
5. `npm install`
6. Build production version
   1. non-prop environments: `npm run build:prod`
   2. prop environment: `npm run build:prod:prop`
8. Host the resulting `/docs` folder on a hosting platform

#### Backend

1. Pull this repo
2. `aws sso login --profile [PROFILE]` (if not already logged in)
3. In the `.env` file: 
   - set `AWS_ACCOUNT_ID` and `AWS_REGION`
4. `cd PROP`
5. `./build_and_upload_backend_images_production.sh [AWS SSO PROFILE]`
6. wait for the script to finish

NOTE:
- the script builds the Docker images and uploads them to AWS ECR repositories
  - access to that repository / AWS account is required


### Deploy
#### Frontend

1. Host the `docs` folder created during the build process on a hosting platform

#### Backend

Pre-requisites:
- make sure that the VPC endpoints `ecr.dkr`, `ecr.api` and `aws_ec2_instance_connect_endpoint` are created using the repository `PROP-infra`

1. SSH into the EC2 instance via the management console
   1. access to the EC2 instance / AWS account is required
2. copy `compose_deploy.yml` to the instance
3. ensure the `.env` file is present and contains the following variables:
   - `AWS_ACCOUNT_ID`
   - `AWS_REGION`
   - `BACKEND_APIKEY`
   - `MAFFT_ARRAY_COUNT`
   - `MAFFT_ARRAY_LENGTH`
   - `CLUSTALW_ARRAY_COUNT`
   - `CLUSTALW_ARRAY_LENGTH`
   - `NO_ALIGNMENT_ARRAY_COUNT`
   - `NO_ALIGNMENT_ARRAY_LENGTH`
   - `FILE_SIZE_LIMIT`
4. `aws ecr get-login-password  | docker login --username AWS --password-stdin 151247720829.dkr.ecr.ap-northeast-1.amazonaws.com`
5. `docker compose -f compose_deploy.yml up -d`

## How to update 
### MAFFT / ClustalW variables
- array length and count variables can be set in `./.env.fixedVariables`
  - can set:
    - MAFFT array length & count
    - ClustalW array length & count
### frontend only
1. `sudo ./dc_script.sh --frontend-only`

### backend only
1. `sudo ./dc_script.sh --backend-only`

### re-build / update everything 
1. Rebuild: `sudo ./dc_script.sh`

## Usage via browser

- local environment: `https://localhost:4200/`

- static hosting: domain provided by the hosting platform


This software is released under the MIT License, see LICENSE.txt.
