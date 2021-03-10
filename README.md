# app-example-gce

Example Grouparoo project for deploying Grouparoo on `GCE` with `Google Cloud Run` and `Cloud Build`.

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

1. Configure a new GCE project for Grouparoo, or choose an existing one
2. Create the Grouparoo Database

   - Choose `Postgres CLoud SQL`
   - A `standard` server should be OK, we recommend `4 vCPU, 15 GB`
   - Enable the Private IP address

3. Create the Grouparoo Redis Server

   - Choose `Memorystore - Redis`
   - Create 1 replica
   - `1GB` or capacity should be enough
   - "Enable Auth" but disable "in transit encryption" for redis

4. Create the Cloud Run Service

   - Environment:

     - Choose `Cloud Run (Fully Managed)`
     - Choose `Continuously deploy new revisions from a source repository` and configure the Cloud Build project
     - Choose to use the Node.js Buildpack
     - 256MiB of memory and 1 CPU should be enough for each instance.
     - "Allow all traffic" and "Allow Unauthenticated Invocations" (it's a public service)
     - Set your Environment variables to connect to Redis and Postgres

   - Set the Environment variables to link up the application to the Postgres and Redis server, as well as enable the web server and workers.
     - You'll need to create a VPC connector to access redis and the internal IPs of postgres. [Learn more here](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access#creating_a_connector). Create one in the VPC network panel and then attach it to you Cloud Run Deployment
       - When choosing a CIDR, choose a wide range, like `10.8.1.0/24`, if `10.8.0.0/24` is the default network range.
     - You won't get a user name for redis, so use "redis", ie: `REDIS_URL=redis://redis:{password}@{host}:{port}/0`

5. Configure Monitoring
6. Configure Load Balancer, SSL, and DNS

## Notes

- You may want to modify logging behavior with:
  - `GROUPAROO_LOGS_STDOUT_DISABLE_TIMESTAMP=true`- GCE adds timestamps to all log messages
  - `GROUPAROO_LOGS_STDOUT_DISABLE_COLOR=true`- GCE will not render log messages in color

---

Vist https://github.com/grouparoo/app-examples to see other Grouparoo Example Projects.
