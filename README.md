# APIMImportViaDevOpsPS

This Repo contains the Powershell to automatically on-board  an API via a valid swagger file into APIM 

The script can be run after as task in Azure DevOps after the API backend service is deployed . 

The script will create the API with a API version set and path versioning scheme. 

The script will also create NamedValues and Product for API , if they are not found.

It then applies the policy to set backend url value retrieval from namved value pair on API scope. 

Script also enables the CORS policy on the APIM. 

Run the power shell via piepline task Azure Powershell @5 in Azure DevOps and you must be good-to-go.

A yaml template file based on Azure DevOps has been provided as well. 
