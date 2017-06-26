Use the Azure CLI to get the remote deployment URL for your API App, then configure your local Git deployment to be able to push to the remote.

```bash
giturl=$(az webapp deployment source config-local-git -n $app_name \ -g myResourceGroup --query [url] -o tsv)

git remote add azure $giturl
```

Push to the Azure remote to deploy your app. You are prompted for the password you created earlier when you created the deployment user. Make sure that you enter the password you created in earlier in the quickstart, and not the password you use to log in to the Azure portal.

```bash
git push azure master
```
