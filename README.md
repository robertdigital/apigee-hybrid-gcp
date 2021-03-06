# apigee-hybrid-gcp

This project allows you to install apigee hybrid runtime in Google Cloud Platform using GCP's deployment manager. 

## Prerequisite

### gcloud
- Install gcloud sdk from https://cloud.google.com/sdk/downloads
- Initialize your account
- Set gcloud to your gcp project
```
  gcloud config set project <project-id>
```

### Enable APIS

Enable Following APIS in the GCP Project

- deploymentmanager.googleapis.com
- Runtime Configuration API
- cloudresourcemanager.googleapis.com
- IAM API
- Apigee

### IAM Roles

- Allocate Owner access to default cloudservices account :

Go to IAM & Admin -> IAM from GCP Console. You will find a IAM account like 00000000@cloudservices.gserviceaccount.com
Allocate Owner access to the account


- Allocate access to default compute services accouunt -

Go to IAM & Admin -> IAM from GCP Console. You will find a IAM account like 00000000-compute@developer.gserviceaccount.com	

Allocate following roles to the account -

```
Kubernetes Engine Admin
Editor
Project IAM Admin
```


## Getting Started
- Keep Key/Certificate pair in config/ directory

- Edit apigee-cluster.yaml and edit apigee variables. These apigee variables are same as overrides.yaml except following changes
  
  - *sslKeyPath is key, sslRootCAPath is root and sslCertPath is crt in these yaml properties*
  - *serviceAccount path doesn't need to be set to any values. These values will be set when you deploy using deployment manager*
  - *gcpExternalIp property can be used to specify external LB for mart or ingress*


```
imports:
- path: config/ingress-server.key
  name: ingress-server.key
- path: config/ingress-server.crt
  name: ingress-server.crt
- path: config/mart-server.key
  name: mart-server.key
- path: config/mart-server.crt
  name: mart-server.crt
- path: config/cassandra.key
  name: cassandra.key
- path: config/cassandra.crt
  name: cassandra.crt
- path: config/cassandra-root.crt
  name: cassandra-root.crt

 apigee:
      envs:
        - name: test
          crt: ingress-server.crt
          key: ingress-server.key
          hostAlias: amer-cs-hybrid-demo1.hybrid-apigee.net
          pollInterval: 60
          gcpExternalIp: 104.198.154.187
        - name: prod
          crt: ingress-server.crt
          key: ingress-server.key
          hostAlias: amer-cs-hybrid-demo1.hybrid-apigee.net
          pollInterval: 60
          gcpExternalIp: 104.198.154.187
      mart:
        nodeSelector:
          key: cloud.google.com/gke-nodepool
          value: apigee-runtime
        crt: mart-server.crt
        key: mart-server.key
        hostAlias: amer-cs-hybrid-mart.hybrid-apigee.net
        gcpExternalIp: 35.202.16.238
      cassandra:
        pullPolicy: Always
        nodeSelector:
          key: cloud.google.com/gke-nodepool
          value: apigee-data
        storage:
          type: gcepd
          capacity: 500Mi
          gcepd:
            replicationType: none
        root: cassandra-root.crt
        crt: cassandra.crt
        key: cassandra.key
```
   

- Deploy to GCP

    ```
    ./deploy.sh "RESOURCE_NAME"
    ```
    RESOURCE_NAME is the name you give to your deployments. All the GCP resources will be tagged under that RESOURCE. All the GCP resources are created with name  having prefix of "RESOURCE_NAME".
    e.g :

```sh
 /deploy.sh my-hybrid
The fingerprint of the deployment is 8FaDn2YGgCmgZxD-Pi71hA==
Waiting for create [operation-1564328751326-58ebfab88e35c-6bbd8f1a-317d7e41]...done.
Create operation operation-1564328751326-58ebfab88e35c-6bbd8f1a-317d7e41 completed successfully.
NAME                             TYPE                                                                          STATE      ERRORS  INTENT
get-iam-policy    gcp-types/cloudresourcemanager-v1:cloudresourcemanager.projects.getIamPolicy  COMPLETED  [] 
hybrid                                container.v1.cluster    COMPLETED  []
my-hybrid-admin-reader                iam.v1.serviceAccount   COMPLETED  []
my-hybrid-admin-writer   iam.v1.serviceAccount                COMPLETED  []
my-hybrid-apigee-cluster-config  runtimeconfig.v1beta1.config COMPLETED  []
my-hybrid-apigee-cluster-vm   compute.v1.instance             COMPLETED  []
my-hybrid-apigee-cluster-waiter  runtimeconfig.v1beta1.waiter COMPLETED  []
my-hybrid-cassandra-backups      iam.v1.serviceAccount        COMPLETED  []
my-hybrid-logs-writer            iam.v1.serviceAccount        COMPLETED  []
my-hybrid-metrics-writer         iam.v1.serviceAccount        COMPLETED  []
patch-iam-policy gcp-types/cloudresourcemanager-v1:cloudresourcemanager.projects.setIamPolicy COMPLETED  []
```


## Undeploy and Clean the deployment
```sh
./clean.sh "RESOURCE_NAME"
```
e.g :
```sh
./clean.sh my-hybrid
```

## Troubleshootig



## License

Apache 2.0 - See [LICENSE](LICENSE) for more information.
