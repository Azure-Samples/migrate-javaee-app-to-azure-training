# 05 - Setup GitHub Actions

__This guide is part of the [migrate Java EE app to Azure training](../README.md)__

In this section, learn how to setup a GitHub Actions workflow file and the necessary secrets to continuously deploy your Java EE app.

---

## Overview of GitHub Actions

GitHub Actions makes it easy to automate your software development workflows. Your automation can be triggered on events like when a commit is pushed, a comment is made, a pull request is opened, or on a schedule. These workflows are stored as `.yml` files in the `.github/workflows` directory of your repository.

- [GitHub Actions documentation](https://docs.github.com/actions)

## The workflow file

The workflow shown below has a single job to build the WAR file using Maven and deploy it using the webapps-deploy action. Copy and paste this file into the `.github/workflows/` directory as `build-and-deploy.yml`.

```yaml
# Docs for the Azure Web Apps Deploy action: https://github.com/Azure/webapps-deploy
# More GitHub Actions for Azure: https://github.com/Azure/actions

name: Build and deploy WAR app to Azure Web App

on:
    push:
        branches:
        - master
    workflow_dispatch:

jobs:
    build-and-deploy:
        runs-on: ubuntu-latest

        steps:
        - uses: actions/checkout@master

        - name: Set up Java version
          uses: actions/setup-java@v1
          with:
            java-version: '8'

        - name: Build with Maven
          run: mvn clean install

        - name: Deploy to Azure Web App
          uses: azure/webapps-deploy@v2
          with:
            app-name: 'freebergJBoss'
            slot-name: 'production'
            publish-profile: ${{ secrets.APP_SERVICE_PUBLISH_PROFILE }}
            package: '${{ github.workspace }}/target/*.war'
```

Next, browse to your web app in the Azure Portal and download the publish profile by clicking **Get publish profile**. Then copy and paste the contents of that file into a GitHub secret named **APP_SERVICE_PUBLISH_PROFILE**.

## Modify the workflow file (Optional)

- Two jobs
- Add workflow for PR checks

---

⬅️ Previous guide: [Step 04 - Monitor Java EE Application](../step-04-monitor-java-ee-app/README.md)

➡️ Next guide: [Conclusion](../step-99-conclusion/README.md)