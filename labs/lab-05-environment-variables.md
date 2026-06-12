# Lab 05 — Environment Variables

## What This Lab Covers

This lab covers how to pass configuration into a Docker container at runtime using environment variables. The core idea is straightforward: values that might change between environments, like colors, URLs, ports, or API endpoints, should not be hardcoded into your application. They belong outside the code so you can change them without touching the application itself.

This lab covers the `-e` flag for passing environment variables at runtime and using `docker inspect` to check what environment variables are set on a running container.

---

## Why Configuration Belongs Outside Your Code

If you hardcode a value directly into your application, changing it means modifying the code, rebuilding the image, and redeploying the container. That is a lot of work for a configuration change.

Moving that value into an environment variable means the application reads it at runtime instead. The code stays the same. The image stays the same. You just pass a different value when you start the container. That is what makes containers portable across different environments. Development, staging, and production can all run the same image with different configuration passed in at startup.

This is also why the ENV instruction exists in a Dockerfile, which was covered in Lab 03. ENV sets a default value baked into the image at build time. The `-e` flag at runtime overrides it. Both work together: ENV gives you a sensible default, `-e` gives you the flexibility to change it without rebuilding.

---

## Commands Covered

**`-e` (environment variable at runtime)**

The `-e` flag passes an environment variable into a container when it starts. The application inside the container can then read it the same way it would read any environment variable.

    docker run -e APP_COLOR=blue my-web-app

To run multiple containers with different configurations, pass a different value each time.

    docker run -e APP_COLOR=blue my-web-app
    docker run -e APP_COLOR=green my-web-app
    docker run -e APP_COLOR=red my-web-app

Each container runs the same image but behaves differently based on the value passed in. No code changes, no rebuilds.

You can pass multiple environment variables in a single command by chaining `-e` flags.

    docker run -e APP_COLOR=blue -e APP_PORT=5000 my-web-app

---

**`docker inspect` (checking environment variables)**

`docker inspect` was covered in Lab 01 but it becomes especially useful here. If you need to check what environment variables are actually set on a running container, inspect gives you that information under the config section of its output.

    docker inspect my-web-app

Look for the `Env` field inside the config section. It lists every environment variable the container is running with, including ones set by the base image itself.

---

## AWS Connection

**ECS task definitions.** In ECS, environment variables are defined directly in the task definition under the container definition section. Every time ECS starts a task, those values get passed into the container the same way `-e` does locally. This is how production applications are configured differently across environments without rebuilding the image.

**AWS Systems Manager Parameter Store and Secrets Manager.** Passing sensitive values like database passwords or API keys directly with `-e` is fine locally but not acceptable in production. In AWS, those values live in Parameter Store or Secrets Manager and ECS pulls them in at task startup. The container still receives them as environment variables. The difference is where they are stored and how they are accessed. Understanding `-e` is the foundation that makes that pattern readable.

**Lambda.** Lambda functions also support environment variables configured the same way. The function reads them at runtime without any changes to the code itself.

---

## Troubleshooting

*To be updated after completing the lab.*

---

## Key Observations

- Hardcoding configuration into application code is the problem this entire concept exists to solve. The moment I have to rebuild and redeploy an image just because a URL or a color changed, something is wrong with how the configuration is managed.

- The `-e` flag and the ENV Dockerfile instruction serve different purposes. ENV sets a default baked into the image. The `-e` flag overrides it at runtime. Knowing which one to reach for depends on whether the value is a sensible default or something that changes per environment.

- `docker inspect` being useful here is a good reminder that it is not just for debugging networking issues. Any time I need to confirm what a container is actually running with, inspect is the answer.

- In production the values passed via `-e` locally are the same values that live in AWS Secrets Manager or Parameter Store. The mechanism is identical. Only the source of the value changes.
