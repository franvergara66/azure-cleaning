# Script to clean azure container registry images

Any image that is older than a month should be deleted and if the latest image is older than a month, then retain the latest image and delete the remaining images.

Below is the code snippet

```sh
# Set your Azure Container Registry name
ACR_NAME="tvsmazp360acrdev01"
# Get the list of repositories in the ACR
REPO_LIST=$(az acr repository list --name $ACR_NAME --output tsv)
ONE_MONTH_AGO=$(date -d "1 month ago" +%s)
echo "One month -----> $ONE_MONTH_AGO"
# Iterate over each repository
for REPO_NAME in $REPO_LIST; do
    TAG_LIST=$(az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --orderby time_asc --output tsv)
    LATEST_TAG=$(az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --orderby time_desc --top 1 --output tsv)
    echo "Latest Tag for $REPO_NAME : $LATEST_TAG"
    LATEST_TIMESTAMP=$(az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --detail --orderby time_desc --top 1 --query [].lastUpdateTime --output tsv)
    TIMESTAMP_LOOP=$(az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --detail --orderby time_asc --query [].lastUpdateTime --output tsv)
    LATEST_TIMESTAMP_SECONDS=$(date -d "$LATEST_TIMESTAMP" +%s)
    echo "$REPO_NAME ------>  $LATEST_TIMESTAMP ------->  $LATEST_TIMESTAMP_SECONDS"
    TAG_ARRAY=($TAG_LIST)
    array_length=${#TAG_ARRAY[@]}
    i=0
    echo "Number of images for the repository $REPO_NAME -------> $array_length"
    if [ $LATEST_TIMESTAMP_SECONDS -lt $ONE_MONTH_AGO ]; then
        echo "Latest tag is older than one month: $LATEST_TAG"
        while [ $i -lt $((array_length - 1)) ]; do
          echo "Delete: ${TAG_ARRAY[$i]} for repository $REPO_NAME"
          #az acr repository delete --name $ACR_NAME --image $REPO_NAME:$TAG --yes
          ((i++))
        done
    else
        for TIMESTAMP in $TIMESTAMP_LOOP; do
           echo $TIMESTAMP
        TIMESTAMP_SECONDS=$(date -d "$TIMESTAMP" +%s)
        if [ $TIMESTAMP_SECONDS -lt $ONE_MONTH_AGO ]; then
            echo "Delete ${TAG_ARRAY[$i]} for repository $REPO_NAME ------> $TIMESTAMP_SECONDS"
            #az acr repository delete --name $ACR_NAME --image $REPO_NAME:$TAG --yes
            ((i++))
        fi
    done
    fi
done
 
#az acr manifest list-metadata -r tvsmazp360acrdev01 -n admin-service --orderby time_desc --top 1

```



-- az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --orderby time_asc --output tsv - Used to get list of tags

-- az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --orderby time_desc --top 1 --output tsv - gets latest tag

-- az acr repository show-tags --name $ACR_NAME --repository $REPO_NAME --detail --orderby time_desc --top 1 --query [].lastUpdateTime --output tsv - gets the timestamp of the latest tag

A Cron job is needed to run this every month.
