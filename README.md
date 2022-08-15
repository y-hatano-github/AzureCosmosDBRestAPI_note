# What is this?
This is note to myself.  
A note on how to access the CosmosDB REST API with curl.
  
I referred to the following page:  
https://stackoverflow.com/questions/61426902/use-bash-azure-cli-and-rest-api-to-access-cosmosdb-how-to-get-token-and-hash

# How to access CosmosDB REST API with curl.

```
# az login --tenant {tenant_name}
az login

resourceGroup="{resource group name}"
resourceName="{resource name of cosmos DB}"
comsosDbName="{cosmos DB name}"
baseUrl="https://$resourceName.documents.azure.com/"
verb="get"
containerName="{cosmos DB's container name}"
resourceType="docs"
resourceLink="dbs/$comsosDbName/colls/$containerName/docs"
resourceId="dbs/$comsosDbName/colls/$containerName"
masterKey=$(az cosmosdb keys list --name $resourceName --query primaryMasterKey --output tsv --resource-group $resourceGroup)
now=$(env LANG=en_US TZ=GMT date '+%a, %d %b %Y %T %Z')
signature="$(printf "%s" "$verb\n$resourceType\n$resourceId\n$now" | tr '[A-Z]' '[a-z]')\n\n"
hexKey=$(printf "$masterKey" | base64 --decode | hexdump -v -e '/1 "%02x"')
hashedSignature=$(printf "$signature" | openssl dgst -sha256 -mac hmac -macopt hexkey:$hexKey -binary | base64)
authString="type=master&ver=1.0&sig=$hashedSignature"
urlEncodedAuthString=$(printf "$authString" | sed 's/=/%3d/g' | sed 's/&/%26/g' | sed 's/+/%2b/g' | sed 's/\//%2f/g')
url="$baseUrl$resourceLink"

curl --request $verb \
-H "x-ms-date: $now" \
-H "x-ms-version: 2018-12-31" \
-H "x-ms-documentdb-isquery: true" \
-H "Content-Type: application/query+json" \
-H "Authorization: $urlEncodedAuthString" \
-H "x-ms-documentdb-isquery: True" \
$url
```

about `x-ms-version`
https://docs.microsoft.com/ja-jp/rest/api/cosmos-db/