Param (
   [string] $p_apiPrefix= "Demo-Conf-API",
   [string] $p_rgName="rg-csup-dev",
   [string] $p_apimInstance="apim-csup-dev",
   [string] $p_apiVersion="v1",
   [string] $p_swaggerPath=".\demo-conf-swagger.yaml", #this file must be in the repo to check 
   [string] $p_productName="demo-product",
   [string] $p_backendURL="https://conferenceapi.azurewebsites.net/"
)
$Error.Clear()

try
{
    $apiPrefix=$p_apiPrefix

    $rgName=$p_rgName

    $apimInstance=$p_apimInstance

    $apiVersion=$p_apiVersion

    $apiPath= "/" + $apiPrefix
    
    $swaggerPath=$p_swaggerPath

    $productName=$p_productName

    $newNVName=$apiPrefix.ToLower()+"-url"

    Write-Output $apiPrefix
    Write-Output $rgName
    Write-Output $apimInstance
    Write-Output $apiVersion
    Write-Output $apiPath
    Write-Output $swaggerPath

    $PolicyString='<policies> <inbound> <base /> <cors> <allowed-origins> <origin>*</origin></allowed-origins><allowed-methods><method>GET</method><method>POST</method> <method>DELETE</method><method>PUT</method> <method>HEAD</method> </allowed-methods> </cors><set-backend-service base-url="{{' + $newNVName + '}}" /> </inbound> <backend><base /></backend><outbound><base /></outbound><on-error><base /></on-error></policies>'
    Write-Output $PolicyString.ToString()
    $ApiMgmtContext = New-AzApiManagementContext -ResourceGroupName $rgName -ServiceName $apimInstance

    #find or create product
    try{
            $specificProduct=Get-AzApiManagementProduct -Context $ApiMgmtContext -ProductId $productName
            Write-Output $specificProduct
            
            }
    catch
    {
        Write-Output "Product not found"
        }
    if($Error)
    {
        Write-Output "Creating new Product."
        $specificProduct=New-AzApiManagementProduct -Context $ApiMgmtContext -ProductId $productName -Title $productName.Replace("-"," ").ToUpper() -Description "Subscribers need a subscription key to access to the API. Administrator approval is required." -LegalTerms "SwissRe Compliance applicable." -ApprovalRequired $True -State "Published"
         Write-Output $specificProduct "after create"
        $Error.Clear()
    }
    
   $specificAPI= Get-AzApiManagementApi -Context $ApiMgmtContext -APiId $apiPrefix.ToLower()
   Write-Output $speicificAPI
 }
 catch 
 {
    Write-Output "API not found"
 }
 if($Error)
    {
        Write-Output "API was not found , create new API with new Version set"

        $apiVersionSetIdout=New-AzApiManagementApiVersionSet -Context $ApiMgmtContext -Name $apiPrefix.Replace("-"," ").ToUpper() -Scheme Segment  -Description "created via azure devops"
        
        Write-Output $apiVersionSetIdout.ApiVersionSetId.ToString(),

        $nvCreateOutput=New-AzApiManagementNamedValue -Context $ApiMgmtContext -NamedValueId $newNVName -Name $newNVName -Value $p_backendURL
        
        $newAPI= Import-AzApiManagementApi -Context $ApiMgmtContext -SpecificationPath $swaggerPath -SpecificationFormat OpenApi -Path $apiPath.ToLower() -ApiVersionSetId $apiVersionSetIdout.ApiVersionSetId.ToString() -ApiVersion $apiVersion -ApiId $apiPrefix.ToLower()
        

        Write-Host "API created from new Swagger. Now applying product"

        Add-AzApiManagementApiToProduct -Context $ApiMgmtContext -ProductId $specificProduct.ProductId -ApiId $newAPI.ApiId

        Write-Host "Product " $specificProduct.Title  "applied on API " $newAPI.Name
        
        $applyPolicyOutput=Set-AzApiManagementPolicy -Context $ApiMgmtContext -ApiId $newAPI.ApiId -Policy $PolicyString

        Write-Output $applyPolicyOutput
    }
else
{
    Write-Output $specificAPI.ApiId.ToString()  "API found , updating ..."

    $existingAPIVersionSetId= $specificAPI.ApiVersionSetId.ToString()

    try {
            $updateAPIOutput= Import-AzApiManagementApi   -Context $ApiMgmtContext -SpecificationPath $swaggerPath -SpecificationFormat OpenApi -Path $apiPath.ToLower() -ApiVersionSetId $existingAPIVersionSetId -ApiVersion $apiVersion -ApiId $apiPrefix.ToLower()
            Add-AzApiManagementApiToProduct -Context $ApiMgmtContext -ProductId $specificProduct.ProductId -ApiId $updateAPIOutput.ApiId
        }
    catch {
                Write-Error  "Unable to update API " 
    }        
    if($Error)
    {
        Write-Output "Failed to update API with name :" $specificAPI.ApiId.ToString()
        }
    else
    {
         Write-Output "Update Complete for API :"  $updateAPIOutput.ApiId.ToString()
    }
}
