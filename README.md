# Workshop with Yan Cui for Serverless

https://github.com/theburningmonk/prsls-trint/

## Extra steps

- In step 4, to export the outputs:

       npx sls export-env --all

- In step 6, to debug the search-restaurants page locally, install https://github.com/jeremydaly/serverless-cloudside-plugin

       npm i --save-dev serverless-cloudside-plugin

- In step 12, how you might create the param:

  1. Create the KMS key

          aws kms create-key --description "$SLS_STAGE's key" --tags TagKey=Owner,TagValue=$SLS_STAGE --region $AWS_REGION | tee KmsKey.json

  2. Then use the `KeyId` from above to set the value of the param:

          for x in get search; do
              aws ssm put-parameter --name /mod03/$SLS_STAGE/$x-restaurants/config --type SecureString --key $(jq -Mr '.KeyMetadata.KeyId' KmsKey.json) --tags Key=Owner,Value=$SLS_STAGE --value '{"defaultResults": 8}' --region $AWS_REGION
          done

  3. Delete the keys

         for x in get search; do
              aws ssm delete-parameter --name /mod03/$SLS_STAGE/$x-restaurants/config
         done


  99. Delete KMS key

         aws kms disable-key --key-id $KeyId
         aws kms schedule-key-deletion --key-id $KeyId --pending-window-in-days 7
