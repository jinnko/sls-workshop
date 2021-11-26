# Workshop with Yan Cui for Serverless

https://github.com/theburningmonk/prsls-trint/

## Extra steps

- In step 4, to export the outputs:

       npx sls export-env --all

- In step 6, to debug the search-restaurants page locally, install https://github.com/jeremydaly/serverless-cloudside-plugin

       npm i --save-dev serverless-cloudside-plugin

- In step 12, how you might create the param:

  1. Create the SSM Parameters

          for x in get search; do
              aws ssm put-parameter --region $AWS_REGION --type String --tags Key=Owner,Value=$SLS_STAGE --name /mod03/$SLS_STAGE/$x-restaurants/config --value '{"defaultResults": 8}'
          done

  2. If you need to update a key:

          aws ssm put-parameter --region $AWS_REGION --type String --name /mod03/$SLS_STAGE/search-restaurants/config --value '{"defaultResults": 8}' --overwrite

- For step 13

  1. Create the KMS key

          aws kms create-key --description "$SLS_STAGE's key" --tags TagKey=Name,TagValue=$SLS_STAGE --region $AWS_REGION | tee KmsKey.json

  2. Put the Key ARN into param store

          aws ssm put-parameter --region $AWS_REGION --tags Key=Owner,Value=$SLS_STAGE --type String --name /mod03/$SLS_STAGE/kmsArn --value "$(jq -Mr '.KeyMetadata.Arn' KmsKey.json)"

  3. Then use the `KeyId` from above to set the value of the param:

          aws ssm put-parameter --region $AWS_REGION --type SecureString --tags Key=Owner,Value=$SLS_STAGE --key $(jq -Mr '.KeyMetadata.KeyId' KmsKey.json) --name /mod03/$SLS_STAGE/search-restaurants/secretString --value "this isn't really a secret"

  98. Delete the keys

         for x in get search; do
              aws ssm delete-parameter --region $AWS_REGION --name /mod03/$SLS_STAGE/$x-restaurants/config
         done

  99. Delete KMS key

         aws kms disable-key --region $AWS_REGION --key-id $KeyId
         aws kms schedule-key-deletion --region $AWS_REGION --key-id $KeyId --pending-window-in-days 7
