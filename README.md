# app-example-gce

Example Grouparoo project for deploying Grouparoo on `GCE` with `Google Cloud Run`.

Goal: To create a scalable and flexible Grouparoo deployment that:

- Has no servers to directly manage
- Can be auto-scaled as needed
- Will be automatically deployed when the code changes
- Is a [12-factor app](https://12factor.net/) with all configuration stored in the Environment

## Repository Configuration

1. Create a new Grouparoo project. Learn more @ https://www.grouparoo.com/docs/installation.

```
npm install -g grouparoo
grouparoo init .
```

2. Install the Grouparoo plugins you want, e.g.: `grouparoo install @grouparoo/postgres`. Learn more @ https://www.grouparoo.com/docs/installation/plugins

## Deployment Steps

1. Configure your VPC
2. Create the Grouparoo Database

   - Use Postgres
   - Use `Provisioned` servers, we recommend `xxx`
   - Be sure to `link` the database to your Cloud Run app

3. Create the Grouparoo Redis Server

   - Create 1 replica
   - We recommend `xxx` servers
   - Disable "in transit encryption" for redis

4. Create the Elastic Beanstalk Project

   - Application
     - `Node.js 14 running on 64bit Amazon Linux 2`
   - Environment:
     - Choose to deploy the sample project first
     - `t2.micro` instances should be OK
     - Set your Environment variables to connect to Redis and Postgres

5. Configure Monitoring
6. Configure Load Balancer, SSL, and DNS

## Notes
