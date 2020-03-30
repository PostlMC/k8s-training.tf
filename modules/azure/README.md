# Microsoft Azure

It looks like Azure is the only cloud provider that has k8s 1.17.x available at the time of this writing, so it got bumped up the priority list since that's what the CKA/CKAD exams are based on.

## Service Principal

The `azurerm_kubernetes_cluster` resource apparently requires a service principal, and that'll need to be set up before the plan can be run.  

Service principal creation via the [Azure CLI][az_cli]:

``` shell
SPID=$(az ad sp create-for-rbac -n "http://sp-aks-manager" --sdk-auth \
    --skip-assignment | \
    tee sp-aks-manager.json | jq -r '.clientId')
```

_The following CLI examples are useful, but are not required for the plan to run:_

Assign the **...** role to the new service principal only for the single
  resource group specified in the `${RG}` environment variable:

``` shell
az role assignment create --assignee ${SPID} --resource-group ${RG} \
    --role "..."
```

**TODO:** Figure out the minimal role required above to make this work.  For now, whatever default permissions are assigned work (I would think none!), but I want to _know for sure_ that the SP is set up correctly.

View the service principal:

``` shell
az ad sp list --all | jq ".[]|select(.appId==\"${SPID}\")"
```

Delete the service principal:

``` shell
az ad sp delete --id ${SPID}
```

[az_cli]: https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest
