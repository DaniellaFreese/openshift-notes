# Group Sync Operator 
[Group Sync Operator](https://github.com/redhat-cop/group-sync-operator) is maintained by the RedHat Communities of Practice. 

It is used on Openshift to sync groups from external OIDC providers to the cluster. 

## Deployment on Openshift 
The operator can be deployed on openshift via Helm Charts or via Operator Hub. More details on the [github page](https://github.com/redhat-cop/group-sync-operator#deploying-the-operator-1)

## [Azure](https://github.com/redhat-cop/group-sync-operator#azure) 
1. Notes on Authentication to Azure: 

Followed the documentation provided by the team. Needed to make the following adjustments to the App Registration to the 3 Microsoft Graph API's: According to [Issue #51 Document required graph-api permissions](https://github.com/redhat-cop/group-sync-operator/issues/51), you need to make sure to use Application Scope Instead of Delegated. 


2. Sample Group Sync Custom Resource for Azure
```
apiVersion: redhatcop.redhat.io/v1alpha1
kind: GroupSync
metadata:
  name: azure-groupsync
spec:
  providers:
  - name: azure
    azure:
      credentialsSecret:
        name: azure-group-sync
        namespace: group-sync-operator
      userNameAttributes: 
        - upn 
      groups: 
        - group1
        - group2 
        - group3 
```