# Install Tiledesk with Kubernetes and Helm

[Tiledesk](https://www.tiledesk.com/) is an open source live chat platform.

## Introduction

This chart bootstraps a Tiledesk deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.14+
- Helm 3.2+
- A Kubernetes Cluster. This guide is tested with: Minikube with [Ingress plugin enabled](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#enable-the-ingress-controller) (This [Minkube Tiledesk Guide](./docs/MINIKUBE.md) can be helpful) and Google Kubernetes Engine (GKE)


## Cluster Requirements
To install the Tiledesk chart, you need an existing Kubernetes cluster.
The requirements of the single pods can vary dependent on the model size and the number of users. We recommend providing at least the following resources:


| Deployment                     | CPU | Memory |
|--------------------------------|-----|--------|
| tiledesk-server                | 1   | 1GB    |
| tiledesk-dashboard             | 0,2 | 200MB  |
| tiledesk-cds                   | 0,2 | 200MB  |
| chat21-server                  | 0,5 | 512MB  |
| chat21-http-server             | 0,2 | 200MB  |
| chat21-web-widget              | 0,2 | 200MB  |
| chat21-ionic                   | 0,2 | 200MB  |
| rabbitmq                       | 0,5 | 1GB    |
| redis                          | 0,5 | 8GB    |
| mongodb                        | 0,5 | 4GB    |

We recommend a size at least 30 GiB for the database volume claim.

N1-standard-1 type is the minimum cluster type if you choose GKE.

## Adding repo

```console
helm repo add tiledesk https://tiledesk.github.io/tiledesk/helm/releases
```

## Installing the Chart

To install the chart with the release name `my-tiledesk`:

```console
helm install my-tiledesk tiledesk/tiledesk
```

 **PAY-ATTENTION: helm release-name must contains 'tiledesk' word as suffix or prefix (e.g. *my-tiledesk*)**

The command deploys Tiledesk on the Kubernetes cluster in the default namespace. 

# Open the dashboard
Open the browser at http://<YOUR_INGRESS_IP>*/dashboard/* (ATTENTION the final slash is required) and signin as admin with :

* email: admin@tiledesk.com
* password: superadmin

You can get the Ingress Ip from the Address column running ```kubectl get ingress my-tiledesk-proxy-nginx```

# Accessing Logs
This section describes how to get logs from the running containers.

1. Get the name of the pod which you want to get the logs of with: ```kubectl get pods```
If namespace is used run : ```kubectl --namespace <your namespace> get pods ```
The output should be similar to this
```
NAME                                                  READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-5c798c9ffc-t5wmb             1/1     Running   0          50d
nginx-ingress-default-backend-7db6cc5bf-qgdtb         1/1     Running   0          50d
tiledesk-helm-1593793077-dashboard-6f4654c465-f566q   1/1     Running   0          51d
tiledesk-helm-1593793077-ionic-56b474d986-jgqsl       1/1     Running   0          51d
tiledesk-helm-1593793077-mongodb-6d4fdf5965-kbcdg     1/1     Running   0          51d
tiledesk-helm-1593793077-server-5c4cbf75f5-v2d67      1/1     Running   0          48d
tiledesk-helm-1593793077-webwidget-7445fb45cc-gdpmd   1/1     Running   0          51d
```
tiledesk-helm-1593793077 is for example the name of the Tiledesk deployment.

2. To get the logs of the container run: ```kubectl --namespace <your namespace> logs -f <name of the pod>```

## Ingress

The default Tiledesk Ingress is configured whitout the hostname, so the rule applies to all inbound HTTP traffic through the IP address specified. There are other ingress hosts in the configuration but they are disabled by default. See the values.yaml file.

# Parameters
## Common Parameters

| Name                    | Description                                                       | Value                                                                        |
|-------------------------|-------------------------------------------------------------------|------------------------------------------------------------------------------|
| mongodb.enabled         | Enable MongoDB® chart                                             | true                                                                         |
| MONGODB_URI             | The mongodb connection uri for Tiledesk server                    | mongodb://{{tiledesk.fullname}}-mongodb/tiledesk                             |
| CHAT21_MONGODB_URI      | The mongodb connection uri for chat21 messaging system            | mongodb://{{tiledesk.fullname}}-mongodb/chat21                               |
| WEB_CLICK_ACTION        | The Web click action url for web push notification                |                                                                              |
| EMAIL_ENABLED           | Enable email module                                               | false                                                                        |
| EMAIL_HOST              | The smtp email host                                               |                                                                              |
| EMAIL_USERNAME          | The smtp email username                                           |                                                                              |
| EMAIL_PASSWORD          | The smtp email password                                           |                                                                              |
| EMAIL_FROM_ADDRESS      | The smtp email from address                                       |                                                                              |
| EMAIL_BASEURL           | The dashboard base url used by email template                     | http://console.tiledesk.local/dashboard                                      |
| CACHE_ENABLED           | Enable the redis cache module                                     | true                                                                         |
| SUPER_PASSWORD          | Specify the Super Admin password                                  | superadmin                                                                   |
| CLOUDAMQP_URL           | The Rabbit MQ connection string                                   | amqp://{{QUEUE_CREDENTIAL}}@{{tiledesk.fullname}}-rabbitmq:5672?heartbeat=60 |
| server.resources        | Tiledesk Server pods' resource requests and limits                | {}                                                                           |
| dashboard.resources     | Tiledesk Dashboard pods' resource requests and limits             | {}                                                                           |
| ionic.resources         | Web Chat pods' resource requests and limits                       | {}                                                                           |
| webwidget.resources     | Widget pods' resource requests and limits                         | {}                                                                           |
| c21httpsrv.resources    | Chat21 Messaging HTTP endpoint pods' resource requests and limits | {}                                                                           |
| c21srv.resources        | Chat21 Messaging engine pods' resource requests and limits        | {}                                                                           |
| ingress.enabled         | Set to true to enable ingress record generation                   | true                                                                         |
| server.image.repository | Tiledesk server image repository                                  | tiledesk/tiledesk-server                                                     |
| server.image.repository | Tiledesk server image tag                                         | <LAST STABLE>                                                                |

# Other configurations (Optional)

## Create an nginx Ingress controller
If your cluster doesn't have a built-in nginx ingress controller (ex. GKE) you must mannually deploy it with :

```console
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx/
helm install ingress-nginx ingress-nginx/ingress-nginx
```

More info here: https://kubernetes.github.io/ingress-nginx/deploy/#using-helm



## Configure a Custom Domain

You can configure the Custom Domain passing the Ingress parameters inline without modifing the value.yaml file for the hosts as follow:

```console
helm upgrade --recreate-pods my-tiledesk tiledesk/tiledesk --set EXTERNAL_BASE_URL=http://console.YOUR_CUSTOM_DOMAIN --set ingress.hosts.hostconsole.enabled=true --set ingress.hosts.hostconsole.name=console.YOUR_CUSTOM_DOMAIN
```
Remember to create a DNS entry for your "console.YOUR_CUSTOM_DOMAIN" domain or add it to your /etc/hosts file (just for local testing).

If you want you can disable the first ingress host "all inbound HTTP traffic through the IP address" adding : ```--set ingress.hosts.hostip.enabled=false```. So you only have to log in using the domain name and not the ip.

Add ```--recreate-pods``` option to the above command if you want to be sure all the pods are restarted with the new values.

Complete example with console.tiledesk.local domain:

```console
helm upgrade --recreate-pods my-tiledesk tiledesk/tiledesk --set EXTERNAL_BASE_URL=http://console.tiledesk.local --set ingress.hosts.hostconsole.enabled=true --set ingress.hosts.hostconsole.name=console.tiledesk.local --set ingress.hosts.hostip.enabled=false
```

## Configure TLS

See [here](https://github.com/Tiledesk/tiledesk-deployment/blob/master/helm/docs/tls.md) how to configure TLS certificate for the Tiledesk installation. 


## Push Notifications

Push notifications are available only with a configured TLS certificate.


# Useful instructions

## Upgrade the Chart


To upgrade the `my-tiledesk` deployment:

```console
helm upgrade --recreate-pods my-tiledesk tiledesk/tiledesk
```
##  Passing inline environment variables

To install the `my-tiledesk` deployment passing the parameters inline without modifing the value.yaml you can for example :

```console
helm install my-tiledesk tiledesk/tiledesk --set EMAIL_ENABLED=true --set EMAIL_HOST=YOUR_EMAIL_HOST 
```

## Security

You can also set your own jwt secret key to sign mqtt tokens into Tiledesk system to communicate with RABBITMQ over AMQP and MQTT protocols.

Steps to configure your own secret are:
- First of all, download *[tdchatserver](https://github.com/chat21/tdchatserver)* chat21 git project or use:
```
git repo clone tdchatserver
```
- Follow the instructions into README file in the same repo and take note to:
    - JWT_RABBIT_SECRET value (choose by you)
    - OBSERVER TOKEN, CHAT21 HTTP SERVER TOKEN and RABBIT ADMIN (WEB CONSOLE) TOKEN values generated after repo is launched

- Replace the following variables with the created ones:
    - **CHAT21_JWT_SECRET** --> JWT_RABBIT_SECRET
    - **CHAT21_ADMIN_TOKEN** --> 'ignored:' + CHAT21 HTTP SERVER TOKEN
    - **QUEUE_CREDENTIAL** --> 'ignored:' + CHAT21 HTTP SERVER TOKEN
    - **CHAT21_SRV_RABBITMQ_CREDENTIAL** --> 'ignored:' + OBSERVER TOKEN
    - **JWT_KEY** --> JWT_RABBIT_SECRET 
    - **CHAT21_HTTPSRV_RABBITMQ_CREDENTIAL** --> 'ignored:' + CHAT21 HTTP SERVER TOKEN
    - **<<"value">>** into rabbitmq.advancedConfiguration --> JWT_RABBIT_SECRET
    
- Upgrade the `my-tiledesk` deployment:

```console
helm upgrade --recreate-pods my-tiledesk tiledesk/tiledesk
```

## Uninstalling the Chart

To uninstall/delete the `my-tiledesk` deployment:

```console
helm delete my-tiledesk
```

The command removes all the Kubernetes components associated with the chart and deletes the release.
