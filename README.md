# multi-tenant-saas-ui

Needs the following environment in order to install:
```
REGION="aws region"

ADMIN_APIGATEWAYURL # Serves all UI apps
ADMIN_USERPOOLID
ADMIN_APPCLIENTID
```


```shell
cd clients/Admin || exit # stop execution if cd fails

echo "Configuring environment for Admin Client"
cat <<EoF >./src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: '$ADMIN_APIGATEWAYURL',
};

EoF
cat <<EoF >./src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: '$ADMIN_APIGATEWAYURL',
};
EoF

cat <<EoF >./src/aws-exports.ts
const awsmobile = {
    "aws_project_region": "$REGION",
    "aws_cognito_region": "$REGION",
    "aws_user_pools_id": "$ADMIN_USERPOOLID",
    "aws_user_pools_web_client_id": "$ADMIN_APPCLIENTID",
};

export default awsmobile;
EoF

# yarn install && yarn build
npm install --legacy-peer-deps && npm run build

# echo "aws s3 sync --delete --cache-control no-store dist s3://${ADMIN_SITE_BUCKET}"
# aws s3 sync --delete --cache-control no-store dist "s3://${ADMIN_SITE_BUCKET}"
# Sync and invalidate all in the end

if [[ $? -ne 0 ]]; then
  exit 1
fi

echo "Completed configuring environment for Admin Client"

# Configuring application UI
echo "aws s3 ls s3://${APP_SITE_BUCKET}"
if ! aws s3 ls "s3://${APP_SITE_BUCKET}"; then
  echo "Error! S3 Bucket: $APP_SITE_BUCKET not readable"
  exit 1
fi

cd ../

CURRENT_DIR=$(pwd)
echo "Current Dir: $CURRENT_DIR"

cd Application || exit # stop execution if cd fails

echo "Configuring environment for App Client"

cat <<EoF >./src/environments/environment.prod.ts
export const environment = {
  production: true,
  regApiGatewayUrl: '$ADMIN_APIGATEWAYURL',
};
EoF

cat <<EoF >./src/environments/environment.ts
export const environment = {
  production: true,
  regApiGatewayUrl: '$ADMIN_APIGATEWAYURL',
};
EoF

yarn install && yarn build
# npm install --legacy-peer-deps && npm run build

# echo "aws s3 sync --delete --cache-control no-store dist s3://${APP_SITE_BUCKET}"
# aws s3 sync --delete --cache-control no-store dist "s3://${APP_SITE_BUCKET}"
# Sync and invalidate all in the end

if [[ $? -ne 0 ]]; then
  exit 1
fi

echo "Completed configuring environment for App Client"

# Configuring landing UI

echo "aws s3 ls s3://${LANDING_APP_SITE_BUCKET}"
if ! aws s3 ls "s3://${LANDING_APP_SITE_BUCKET}"; then
  echo "Error! S3 Bucket: $LANDING_APP_SITE_BUCKET not readable"
  exit 1
fi

cd ../

CURRENT_DIR=$(pwd)
echo "Current Dir: $CURRENT_DIR"

cd Landing || exit # stop execution if cd fails

echo "Configuring environment for Landing Client"

cat <<EoF >./src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiGatewayUrl: '$ADMIN_APIGATEWAYURL'
};
EoF

cat <<EoF >./src/environments/environment.ts
export const environment = {
  production: false,
  apiGatewayUrl: '$ADMIN_APIGATEWAYURL'
};
EoF

yarn install && yarn build
# npm install --legacy-peer-deps && npm run build

# echo "aws s3 sync --delete --cache-control no-store dist s3://${LANDING_APP_SITE_BUCKET}"
# aws s3 sync --delete --cache-control no-store dist "s3://${LANDING_APP_SITE_BUCKET}"
alto sync ui-stack -a
alto invalidate ui-stack -a



if [[ $? -ne 0 ]]; then
  exit 1
fi

cd ../..

echo "Completed configuring environment for Landing Client"

echo "Admin site URL: https://$ADMIN_SITE_URL"
echo "Application site URL: https://$APP_SITE_URL"
echo "Landing site URL: https://$LANDING_APP_SITE_URL"
echo "Successfully completed deployment"

```