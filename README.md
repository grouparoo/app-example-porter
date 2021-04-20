# Grouparoo `app-example-porter`

_**An example Grouparoo project for deploying Grouparoo to Porter**_

Goal: To create Grouparoo deployment that:

- Relies on AWS's hosted Kubernetes service
- Is automatically deployed when the git project changes
- Is configured entirely via Environment Variables (is a 12-factor app)

## Notes

In this example, we deployed a single server type that acts as both a WEB and BACKGROUND WORKER service for simplicity. For more robust deployments, you may want to split these tasks to different servers.

You may want to modify logging behavior with:

- `GROUPAROO_LOGS_STDOUT_DISABLE_TIMESTAMP=true`- Don't prepend timestamps to log messages
- `GROUPAROO_LOGS_STDOUT_DISABLE_COLOR=true`- Disable colorizing log message lines
- `GROUPAROO_LOGS_PATH="/var/log/grouparoo"`- Logs are stored in `/var/log/grouparoo` with this setting enabled. You can store them wherever you like.

---

Visit https://github.com/grouparoo/app-examples to see other Grouparoo Example Projects.

```

```
