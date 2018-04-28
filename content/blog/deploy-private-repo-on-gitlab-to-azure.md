---
title: "Deploy Private Repo on Gitlab to Azure"
date: 2018-04-27T19:52:21-07:00
draft: false
---

Here are the steps to deploy your private repository on gitlab to Azure:

### Get Kudu's public deployment key

[Kudu](https://github.com/projectkudu/kudu) is the engine behind git deployments in Azure App Service.
For Kudu to be able to read from a private git repository, we need to first get its public deployment key.

1.  Get the deployment trigger uri
    On Azure portal, go to your Web app's Properties blade. It looks something like this

    ```
    https://$mysite:BigRandomPassword@mysite.scm.azurewebsites.net/deploy
    ```

2.  Based on the url, create the following url

    ```
    https://$mysite:BigRandomPassword@mysite.scm.azurewebsites.net/api/sshkey?ensurePublicKey=1
    ```

3.  Open the url created above in browser to get your deployment key. See [For private repos, set up a deploy key](https://github.com/projectkudu/kudu/wiki/Continuous-deployment)

### Add Kudu's deployment key to Gitlab project

Add Kudu's deployment key to Gitlab project.

> If you are a project master or owner, you can add a deploy key in the project settings under the section 'Repository'. Specify a title for the new deploy key and paste a public SSH key. After this, the machine that uses the corresponding private SSH key has read-only or read-write (if enabled) access to the project.
> <cite>https://docs.gitlab.com/ce/ssh/README.html#deploy-keys</cite>

### Set up deployment source

Follow [this guide](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-continuous-deployment), but choose **External Repository** instead of GitHub.

### Set up auto deployment

Register Kudu's deployment trigger url that you found earlier with Gitlab by going to Integrations page for your project

> Navigate to the webhooks page by going to the Integrations page from your project's settings which can be found under the wheel icon in the upper right corner.
> <cite>https://docs.gitlab.com/ce/user/project/integrations/webhooks.html#webhooks</cite>

After this set up is done, every time you push your changes to your remote Gitlab repository, Azure will redeploy your changes automatically.
