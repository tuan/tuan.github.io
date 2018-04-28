---
title: "Deploy Angular App to Azure"
date: 2018-04-27T19:54:06-07:00
draft: false
---

This is a [good guide](http://stackoverflow.com/questions/37487046/deploy-angular-2-with-azure-webapp) that explains basic steps to deploy a Angular app to Azure. However, there are a few extra items that you might want to do to have your app run smoothly in production.

1.  Generate deployment.cmd

    Instead of using azure-cli, use [kuduscript](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script):

    ```
    npm install kuduscript -g
    ```

    and

    ```
    kuduscript -y --node
    ```

2.  Set up web.config

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <rewrite>
                <rules>
                    <clear />
                    <rule name="Redirect to https" stopProcessing="true">
                        <match url=".*" />
                        <conditions>
                            <add input="{HTTPS}" pattern="off" ignoreCase="true" />
                        </conditions>
                        <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" redirectType="Permanent" appendQueryString="false" />
                    </rule>
                    <rule name="Angular" stopProcessing="true">
                        <match url=".*" />
                        <conditions logicalGrouping="MatchAll">
                            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                        </conditions>
                        <action type="Rewrite" url="index.html" />
                    </rule>
                </rules>
                <outboundRules>
                    <rule name="AdjustCacheForHTMLPages" preCondition="IsIndexHTML">
                        <match serverVariable="RESPONSE_Cache-Control" pattern=".*" />
                        <action type="Rewrite" value="no-cache, no-store, must-revalidate" />
                    </rule>
                    <preConditions>
                        <preCondition name="IsIndexHTML">
                            <add input="{REQUEST_FILENAME}" pattern="index\.html" />
                        </preCondition>
                    </preConditions>
                </outboundRules>
            </rewrite>
        </system.webServer>
    </configuration>
    ```

    * **Redirect to https** rule is to make sure `https` is always used.
    * **Angular** rule is to make sure all routes are handled by your angular app.
    * **AdjustCacheForHTMLPages** rule is to make sure your `index.html` is not cached.
      `angular-cli`/`webpack` generates assets with unique guid to bust the cache, but if your `index.html` is cached, this won't work.

3.  Create a staging slot

    You will eventually need a [staging slot](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-staged-publishing), otherwise whenever you deploy, your app will not be accessible. This is because if you deploy directly to your `production` slot, your app might not be functional while `angular-cli` is building the assets in `/dist` directory. With the staging slot [set up to auto swap with production](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-staged-publishing#configure-auto-swap), you only swap the old assets with new assets after the build in your staging slot is completed.
