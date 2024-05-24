# Basic CD - Continuous Delivery

This repo contains enough code to demonstrate the building and deployment of a containerized application in a Kubernetes container - through a CI/CD pipeline.

The method used in this repository is very primitive and is purpose is just to show how CI/CD pipelines work - in Gitlab.


You can fork this repository, to experiment with it. You will need to setup the following repository secrets for this to work. 

* CONTAINER_REGISTRY_TOKEN (e.g `dckr_pat_ABCkjhgldiuhg7876t7hvo7rt`) (Type: Variable)
* KUBECONFIG_FILE (contains the contents of `~/.kube/config` or any kube config file) (Type: File)

Go to `GitHub -> Your repository -> Settings -> Secrets and Variables -> Actions -> Repository secrets`. Define your variables with correct types, and un-check the "Protect variable" option.

You also need the following variables in the CI environment, but they can be configured in the CI file - as they are not sensitive in nature.

* CONTAINER_REGISTRY_URL (e.g. `docker.io`, `gcr.io`, etc)
* CONTAINER_REGISTRY_USERNAME (e.g. `kamranazeem`)

If you want the resultant deployment to be accessible over internet, then you would need to ensure that:
* the Kubernetes cluster you are deploying to has an ingress controller, e.g. Traefik, nginx, etc. - usually behind some load balancer with a public IP (e.g. `1.2.3.4`)
* you have a DNS domain in your control (`demo.wbitt.com`), and can create A record to point to the load balancer of that cluster. e.g. `example1.demo.wbitt.com` points to the IP `1.2.3.4`

In addition to above, if you have SSL certificate setup correctly too - e.g. with "letsencrypt", then:
* you can use the HTTPS version of the deployment manifest file. 
* and, you can configure your `.gitlab-ci.yml` file to choose which file to use.


If you don't have ingresscontroller at all in your target Kubernetes cluster, simply disable/remove the ingress object definition from the `dep-svc-ing-http.template.yaml` file; and, then configure the `gitlab-ci.yaml` file to use the correct file.

## Successful run:

Verify from local computer:

```
$ kubectl config use-context my-k3s-cluster 
Switched to context "my-k3s-cluster".


$ kubectl -n default get deployment,svc,ingress

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/basic-cd-demo-wbitt-com   1/1     1            1           2m6s

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes                ClusterIP   10.32.0.1      <none>        443/TCP   9d
service/basic-cd-demo-wbitt-com   ClusterIP   10.32.63.154   <none>        80/TCP    2m6s

NAME                                                CLASS     HOSTS                     ADDRESS         PORTS   AGE
ingress.networking.k8s.io/basic-cd-demo-wbitt-com   traefik   basic-cd.demo.wbitt.com   192.168.0.241   80      2m7s

$ kubectl -n default get deployment basic-cd-demo-wbitt-com -o yaml | grep -w "image:"

        image: docker.io/kamranazeem/helloworld:db06a8a0
```

Access the application from home computer:

```
$ curl http://basic-cd.demo.wbitt.com
Hello world!
```
