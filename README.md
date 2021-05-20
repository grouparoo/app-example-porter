# Grouparoo `app-example-porter`

_**An example Grouparoo project for deploying Grouparoo to Porter**_

Goal: To create Grouparoo deployment that:

- Relies on AWS's hosted Kubernetes service
- Is automatically deployed when the git project changes
- Is configured entirely via Environment Variables (is a 12-factor app)

## How We Got Here

> Some of the setup steps have already been done for you to create this app. Here's what we've taken care of:

1. Create a new Grouparoo project. Learn more @ https://www.grouparoo.com/docs/installation.

```
npm install -g grouparoo
grouparoo init .
```

2. Install the Grouparoo plugins you want, e.g.: `grouparoo install @grouparoo/postgres`. Learn more @ https://www.grouparoo.com/docs/installation/plugins

3. Create the `Procfile` that Porter/Kubernetes/Docker will use to launch your app.

```
web: cd node_modules/@grouparoo/core && ./bin/start
```

## Running this Repo

Assuming you have node.js installed (v12+):

1. `git clone https://github.com/grouparoo/app-example-heroku.git`
2. `cd app-example-heroku`
3. `npm install`
4. `cp .env.example .env`
5. `npm start`

## Deploying this Repo

Deploy with Porter!  You will need to launch 2 addtional conatiners which can be found in the "community add-ons" section of the Porter Dashboard:
* A redis container
* A postgres container


## Notes

Since we are using Buildpacks, set the `engines` field in `package.json` to lock in a specififc node.js version.

In this example, we deployed a single server type that acts as both a WEB and BACKGROUND WORKER service for simplicity. For more robust deployments, you may want to split these tasks to different servers.

Even though it's not needed by the buildpack, we use a `Procfile` to run the application directly, bypassing `npm start`. In this way signals are property passed to the app.

---

Visit https://github.com/grouparoo/app-examples to see other Grouparoo Example Projects.


