# Grouparoo `app-example-gcp`

_An example Grouparoo project for deploying Grouparoo on Google Cloud Platform (`GCP`) with `Google App Engine` and `Cloud Build`._

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

1. Configure a new GCP project for Grouparoo, or choose an existing one
2. Create the Grouparoo Database

   - Choose `Postgres CLoud SQL`
   - A `standard` server should be OK, we recommend `4 vCPU, 15 GB`
   - Enable the Private IP address
   - Once the database is running, create a new database like `grouparoo_production`
   - Note and build `DATABASE_URL` in a scratch document

3. Create the Grouparoo Redis Server

   - Choose `Memorystore - Redis`
   - Create 1 replica
   - `1GB` or capacity should be enough
   - "Enable Auth" but disable "in transit encryption" for redis
   - Once redis is up and running, note the AUTH string. You'll use this as the "password" in `REDIS_URL`. The username can be anything, like `"redis"`
   - Note and build `REDIS_URL` in a scratch document

4. Enable Google App Engine

   - Enable the Google App Engine API for this Service [here](https://console.cloud.google.com/appengine/start). You do not need to deploy anything yet

5. Enable Google Cloud Build

   - Enable The Google Cloud Build API [here](https://console.cloud.google.com/cloud-build).
   - The Cloud Build service will also need permissions to deploy App Engine. [Do that here](https://console.cloud.google.com/cloud-build/settings) by enabling `App Engine Admin` and `Service Account User`.
   - Create a new Trigger from your git repo
   - Configure Environment Variables. Note how in `cloudbuild.yaml` we are using Substitution Variables. We are going to store the values we want for the Grouparoo Environment variables within Google Cloud Build so they are baked into the image we deploy, in a custom `.env` file for production. In Google Cloud Build, store the values for your Environment Variables, e.g.: `DATABASE_URL` should be stored as `_DATABASE_URL` (note the leading `_`). The full list is in `cloudbuild.yaml`.

6. Deploy your app by running the CLoud Build Trigger manually

7. Configure Monitoring
8. Configure Load Balancer, SSL, and DNS

## Notes

In this example, we are going to deploy a single server type that acts as both a WEB and BACKGROUND WORKER service for simplicity. To split your services, you'll need to explore the [`service` directive in `app.yaml`](https://cloud.google.com/appengine/docs/standard/nodejs/config/appref). We are keeping alive 2 instances with manual scaling.

Along the way, you may encounter an API that is not yet enabled for for your GCP Project. You'll need to enable many. If you are having trouble, read the logs from Cloud Build to see if there's a link to enable an API (e.g: "App Engine Admin API").

You may want to modify logging behavior with:

- `GROUPAROO_LOGS_STDOUT_DISABLE_TIMESTAMP=true`- Cloud Run adds timestamps to all log messages
- `GROUPAROO_LOGS_STDOUT_DISABLE_COLOR=true`- Cloud Run will not render log messages in color

If you are deleting your App Engine service, be sure to all delete your [Cloud Build](https://console.cloud.google.com/cloud-build/) trigger, or else the app will be re-created again on the next build.

If you add new Environment Variables, you need to add them to both the Cloud Build Trigger's Substitution Variables and `cloudbuild.yml`. Unlike other PaaS platforms, if you change a value in Substitution Variables, you need to rebuild your application, and then re-deploy it.

## Attribution

- The method to securely source and store environment variables was inspired by [this article](https://medium.com/@brian.young.pro/how-to-add-environmental-variables-to-google-app-engine-node-js-using-cloud-build-5ce31ee63d7).

---

Visit https://github.com/grouparoo/app-examples to see other Grouparoo Example Projects.
