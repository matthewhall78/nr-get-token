# Generating Build Configuration Templates

You will likely not need to run the new template generation sections as that the base templates should already be in git. You should be able to skip those steps.

## New Node Builder Template

*If you are creating a new build configuration template, you will likely use the following commands:*

```sh
oc new-build -n k8vopl-tools registry.access.redhat.com/rhscl/nodejs-8-rhel7:latest~https://github.com/bcgov/nr-get-token.git#master --context-dir=frontend --name=get-token-frontend --dry-run -o yaml > openshift/frontend.bc.yaml
sed -i '' -e 's/kind: List/kind: Template/g' openshift/frontend.bc.yaml
sed -i '' -e 's/items:/objects:/g' openshift/frontend.bc.yaml
```

*Note: You need to remove any secrets and credentials that are auto-inserted into the frontend.bc.yaml file.*

## Process and Apply Builder Template

```sh
oc process -n k8vopl-tools -f openshift/frontend.bc.yaml -o yaml | oc create -n k8vopl-tools -f -
```

## New Caddy Static Image Template

*If you are creating a new build configuration template, you will likely use the following commands:*

```sh
oc new-build -n k8vopl-tools --docker-image=docker-registry.default.svc:5000/bcgov/s2i-caddy:v1-stable --source-image=frontend:latest --source-image-path=/opt/app-root/src/dist:tmp -D $'FROM docker-registry.default.svc:5000/bcgov/s2i-caddy:v1-stable\nCOPY tmp/dist/ /var/www/html/\nCMD /tmp/scripts/run' --dry-run --name=get-token-frontend-static -o yaml > openshift/frontend-static.bc.yaml
sed -i '' -e 's/kind: List/kind: Template/g' openshift/frontend-static.bc.yaml
sed -i '' -e 's/items:/objects:/g' openshift/frontend-static.bc.yaml
```

*Note: You need to remove any secrets and credentials that are auto-inserted into the frontend-static.bc.yaml file.*

## Process and Apply Static Image Template

```sh
oc process -n k8vopl-tools -f openshift/frontend-static.bc.yaml -o yaml | oc create -n k8vopl-tools -f -
```

## Tag the latest build and migrate it to the correct project namespace

```sh
oc tag -n k8vopl-dev k8vopl-tools/frontend-static:latest frontend-static:dev --reference-policy=local
```

## Create new Application Deployment

*If you are creating a new application deployment template, you will likely use the following commands:*

```sh
oc new-app -n k8vopl-dev --image-stream=frontend-static:dev --name=get-token-frontend --dry-run -o yaml > openshift/frontend-static.dc.yaml
```

## Process and Apply the Application Deployment

```sh
oc process -n k8vopl-dev -f openshift/frontend-static.dc.yaml -o yaml | oc create -n k8vopl-dev -f -
oc create -n k8vopl-dev route edge frontend --service=frontend --port=2015-tcp
```

## Templating Work in Progress

The above commands will need to be templated. We can expect something like the following in part of the commands:

```sh
'--name=${NAME}${SUFFIX}' '--context-dir=${GIT_DIR}'
```
