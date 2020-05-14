# Helm lab

## Let's create a new package/chart

```shell
helm create webfrontend
```

This will create a folder named `webfrontend` with the following content:

- charts (directory)
- Chart.yaml
- .helmignore
- templates (directory)
- values.yaml


## Edit your chart values

Make the following updates to `webfrontend/values.yaml`:

- Change the image of your container from `image.repository` to `microsoft/azure-vote-front:v1`
- Change the service type to expose your application from `service.type` to `LoadBalancer`


## Edit your chart version

Update `appVersion` to `v1` in `webfrontend/Chart.yaml`


## Install your chart

```shell
helm install webfrontend webfrontend/
```

## Validate that your chart exists

```shell
helm list
```


## Validate that the k8s resources are created and running

```shell
kubetctl get pods
kubetctl get deployments
kubetctl get get services
```

Try to open the url in your browser.


## In the end your can uninstall your chart

```shell
helm uninstall webfrontend
```
