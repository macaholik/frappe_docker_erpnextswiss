export APPS_JSON_BASE64=$(base64 -w 0 apps.json)

docker build \
 --build-arg=FRAPPE_PATH=https://github.com/macaholik/frappe \
 --build-arg=FRAPPE_BRANCH=version-15-authfix \
 --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
 --tag=erpnext:v15 \
 --file=images/custom/Containerfile .


docker compose --env-file custom.env \
    -f compose.yaml \
    -f overrides/compose.mariadb.yaml \
    -f overrides/compose.redis.yaml \
    -f overrides/compose.noproxy.yaml \
    config > compose.custom.yaml

docker compose -p frappe -f compose.custom.yaml up -d

docker compose -p frappe exec backend bench new-site panties --mariadb-user-host-login-scope='172.%.%.%'
docker compose -p frappe exec backend bench --site panties install-app erpnext