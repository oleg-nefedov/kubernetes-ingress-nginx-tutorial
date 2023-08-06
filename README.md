
# Kubernetes Ingress Nginx Detailed Tutorial

- [Chapter 1: Introduction to Kubernetes Ingress and Ingress-Nginx](#chapter-1-introduction-to-kubernetes-ingress-and-ingress-nginx)
- [Chapter 2: Installing Ingress-Nginx using Helm](#chapter-2-installing-ingress-nginx-using-helm)  
- [Chapter 3: Configuring Ingress-Nginx](#chapter-3-configuring-ingress-nginx)
- [Chapter 4: Ingress Resources](#chapter-4-ingress-resources)
- [Chapter 5: TLS Termination](#chapter-5-tls-termination)
- [Chapter 6: Additional Features](#chapter-6-additional-features)
- [Chapter 7: Conclusion](#chapter-7-conclusion)

# Chapter 1: Introduction to Kubernetes Ingress and Ingress-Nginx

Kubernetes is a popular open-source platform for automating deployment, scaling, and management of containerized applications. However, by default Kubernetes does not provide any built-in load balancing or proxy functionality for routing external traffic into the cluster.

This is where ingress controllers like ingress-nginx come in. Ingress controllers allow you to easily configure rules for routing requests to Kubernetes services inside the cluster. Ingress resources are Kubernetes objects that define these rules declaratively.

In this tutorial we will cover the following topics:

- What is Kubernetes Ingress and how does it work?
- Overview of ingress-nginx architecture
- Key benefits of using ingress-nginx
- How to install ingress-nginx using Helm
- Configuring ingress-nginx for your cluster  
- Creating simple ingress resources to route traffic
- TLS termination and name-based virtual hosting
- Additional features like authentication, rate limiting and more

## What is Kubernetes Ingress?

Ingress provides load balancing, SSL termination and name-based virtual hosting for Kubernetes services. It acts as a smart router in front of your services and can route requests to the appropriate service based on the request host, path or other rules. 

For example, you may have two services "app.mydomain.com" and "cart.mydomain.com" running in your Kubernetes cluster. Ingress allows you to route requests to these services based on the request host.

Here is a simple example ingress resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress  
spec:
  rules:
  - host: app.mydomain.com
    http:
      paths:
      - path: /
        backend:
          service: 
            name: app-service
            port: 80
  - host: cart.mydomain.com
    http:
      paths:
      - path: /  
        backend:
          service:
            name: cart-service
            port: 80   
```

This ingress will route requests for "app.mydomain.com" to the "app-service" and requests for "cart.mydomain.com" to "cart-service". 

However, ingress itself is just the rules engine. To actually implement an ingress controller like ingress-nginx is needed.

## Ingress-Nginx Architecture

Ingress-nginx is one of the most popular ingress controllers. It is built around Nginx and provides high-performance load balancing and proxying capabilities.

The ingress-nginx controller has two main components:

1. Nginx Ingress Controller Pod - This is responsible for reading Ingress resource information from the Kubernetes API and generating nginx.conf files to route traffic accordingly.

2. Nginx Load Balancer Service - This service exposes the nginx pods to external traffic and provides load balancing. 

Ingress-nginx also provides additional features like SSL/TLS termination, authentication, rate limiting, rewriting URLs and more.

In the next section we will go through installing ingress-nginx in your Kubernetes cluster using Helm.

# Chapter 2: Installing Ingress-Nginx using Helm

Helm is a popular package manager for Kubernetes that allows you to easily install applications and services. We will use the official ingress-nginx Helm chart to install ingress-nginx in our Kubernetes cluster.

## Prerequisites

- Kubernetes cluster (can be local like minikube or hosted like GKE/EKS)
- Helm v3 installed  
- Kubectl installed and configured to connect to your cluster

## Add Ingress-Nginx Repo 

First we need to add the official ingress-nginx helm repo:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
```

This will add the ingress-nginx chart repository so that we can install charts from it.

## Install Ingress-Nginx

Now we can install ingress-nginx by running:

```
helm install my-release ingress-nginx/ingress-nginx
```

This will install ingress-nginx with the release name "my-release".  

We can check that the pods and services were created successfully:

```
kubectl get pods -n ingress-nginx

NAME                                       READY   STATUS    RESTARTS   AGE
my-release-ingress-nginx-controller-5rcvj   1/1     Running   0          2m
my-release-ingress-nginx-controller-mh6rq   1/1     Running   0          2m

kubectl get service -n ingress-nginx  

NAME                                            TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
my-release-ingress-nginx-controller             LoadBalancer   10.108.69.96   xxx.xxx.xxx.xxx   80:31080/TCP,443:31443/TCP   2m46s
my-release-ingress-nginx-controller-admission   ClusterIP      10.111.28.20   <none>           443/TCP                      2m46s
```

The ingress controller pods and service have been created. The service has type LoadBalancer so it will automatically get an external IP. 

Now we have ingress-nginx up and running using Helm!

In the next chapter we will look at how to customize the configuration like number of replicas, resources limits and more.

# Chapter 3: Configuring Ingress-Nginx

In the previous chapter we installed ingress-nginx using the default configuration. Here we will look at some of the options for customizing the installation to suit your requirements.

## Configuration Options

The main configuration options that can be customized include:

- Number of replica pods  
- Resources limits - CPU/Memory
- Node selectors and tolerations
- Customizing Nginx configuration
- Annotations for advanced customization

Let's go through some examples of configuring these options.

## Number of Replicas

By default ingress-nginx runs 2 replica pods. This can be changed by setting the `controller.replicaCount` value.

For example to run 3 replicas:

```
helm install my-release ingress-nginx/ingress-nginx \
  --set controller.replicaCount=3 
```

## Resources  

CPU and memory resource requests and limits can be configured like:

```  
helm install my-release ingress-nginx/ingress-nginx \
  --set controller.resources.requests.cpu="100m" \
  --set controller.resources.requests.memory="64Mi"
```

This sets the CPU request to 100m and memory request to 64Mi for liveness and readiness probes.

## Node Selectors

To schedule ingress-nginx pods on specific nodes like GPU nodes, nodeSelectors can be used:

```
helm install my-release ingress-nginx/ingress-nginx \
  --set controller.nodeSelector."kubernetes\.io/role"=gpu
```

This will add the node selector to schedule pods on nodes with label "kubernetes.io/role=gpu".

## Customizing Nginx Config  

The default nginx configuration can be overridden using the `controller.config` option. A full nginx configmap can be defined here.

## Annotations

Advanced customizations can be done using annotations. For example enabling CORS:

```  
helm install my-release ingress-nginx/ingress-nginx \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-cross-zone-load-balancing"=true
```

In the next chapter we will look at creating ingress resources to route traffic to Kubernetes services.

# Chapter 4: Ingress Resources

Now that we have ingress-nginx installed, we can start creating Ingress resources to configure routing rules.

Some key things to understand about Ingress resources:

- Declaratively defines rules for routing external traffic to Kubernetes services 
- Uses name-based virtual hosting with hostname and path rules
- Can configure TLS/SSL termination
- Works closely with ingress controller like nginx

Let's start with a simple example.  

## Simple Ingress

We have two services "app-service" and "cart-service" that we want to expose externally.

First we will create a simple ingress resource: 

```yaml  
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
spec:
  rules:
  - http:
      paths:
      - path: /app 
        pathType: Prefix
        backend:
          service:
            name: app-service
            port: 80
      - path: /cart
        pathType: Prefix  
        backend:
          service:
            name: cart-service
            port: 80
```

This ingress will route:

- Requests to `/app` path to `app-service`  
- Requests to `/cart` path to `cart-service`

We can test this by accessing the ingress controller's external IP:

```
curl http://<External-IP>/app  # goes to app-service
curl http://<External-IP>/cart # goes to cart-service
```

## Name-based Virtual Hosting 

For name-based virtual hosting, we use the `host` field in the ingress rule.

```yaml
spec:
  rules:
  - host: app.mydomain.com
    http:
      paths:
      - backend:
          service: 
            name: app-service

  - host: cart.mydomain.com  
    http:
      paths:
      - backend:
          service:
            name: cart-service
```

Now requests to `app.mydomain.com` will be routed to `app-service` and requests to `cart.mydomain.com` will go to `cart-service`.

This covers the basic ingress resources. In the next chapter we will look at SSL/TLS termination.

# Chapter 5: TLS Termination 

To enable TLS (SSL) for our ingresses, we need to add some additional configuration for certificate management and tell ingress-nginx to handle TLS termination.

## TLS Certificate

First, we need a TLS certificate and private key to use for TLS termination. For simplicity, we can generate a self-signed certificate, however for actual production usage you should use a certificate from a trusted Certificate Authority.

Here is an example of generating a self-signed certificate and private key on Linux:

```  
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=mydomain.com" 
```

This generates `tls.crt` and `tls.key` files that we can use for TLS termination. 

## TLS Secret 

Next we need to create a Kubernetes secret containing the certificate and private key:

```
kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```

This will create a secret named `tls-secret` that ingress-nginx can use.

## Ingress TLS Configuration

Now we can update our ingress with a TLS configuration section:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
spec:

  tls:
  - hosts:
    - mydomain.com
    secretName: tls-secret

  rules:

  - host: mydomain.com
    http:
      paths:
      - path: /
        backend:
          serviceName: app-service  
          servicePort: 80
```

This will tell ingress-nginx to use our TLS secret for any requests to the host `mydomain.com`. 

We can test this using curl:

```
curl -k https://mydomain.com/
``` 

The `-k` flag tells curl to ignore invalid certificates so it will allow our self-signed cert.

And that covers TLS termination! The next chapter will discuss additional ingress-nginx features.

# Chapter 6: Additional Features

So far we've covered the core functionality of ingress-nginx like traffic routing, name-based virtual hosts, and TLS termination.

Ingress-nginx also includes some powerful additional features like:

- Authentication
- Rate limiting
- Rewrites and redirects
- Custom load balancing algorithms

Let's go through examples of configuring some of these features.

## Authentication

Ingress-nginx supports authentication using external OIDC providers as well as basic auth.

Here is an example to enable basic auth for an ingress resource:

```yaml
apiVersion: networking.k8s.io/v1  
kind: Ingress
metadata:
  name: myingress
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: my-basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
spec:
#...
```

The annotations enable basic auth, referencing a secret `my-basic-auth` that contains the credentials.

## Rate Limiting  

Rate limits can be applied to an ingress to restrict traffic based on client IP address or request frequency. 

For example to add a rate limit rule:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "1" 
spec:
#...
```

This will limit requests per second from a client IP to 1 request per second. There are additional annotations to control rate limiting in more detail.

## Rewrites and Redirects

URL rewriting and redirects can also be applied using annotations. For example:

```
nginx.ingress.kubernetes.io/rewrite-target: /$1 
```

Will rewrite the URL by capturing a regex group. Redirects, regex replacement, and other rewrite options are also available.

This covers some of the many features ingress-nginx provides on top of basic ingress routing!

In the next chapter we will recap some key takeaways.

# Chapter 7: Conclusion

Let's recap what we've covered in this ingress-nginx tutorial:

- Understanding Kubernetes Ingress and how it works
- Installing ingress-nginx using Helm for easy setup  
- Configuring options like number of replicas, resources, and customizing Nginx
- Creating Ingress resources to route traffic to backend services
- Enabling TLS termination for HTTPS traffic 
- Leveraging additional features like authentication, rate limiting, rewriting URLs

Some key takeaways:

- Ingress-nginx is a powerful and flexible ingress controller for Kubernetes
- It simplifies exposing services outside the cluster and handling TLS
- The external proxy and load balancer reduce load on backend services
- Advanced customization is possible using annotations
- Active project with frequent updates and support

If you need more complex routing capabilities, you may look at service meshes like Istio or Linkerd which build on top of ingress.

For managing cross-cluster ingress traffic, tools like Contour or Traefik may be helpful.

But for most common ingress use cases, ingress-nginx is robust, full-featured, and easy to use. 

Some useful links for further reading:

- Ingress-Nginx GitHub Repo: https://github.com/kubernetes/ingress-nginx
- Ingress-Nginx Documentation: https://kubernetes.github.io/ingress-nginx/
- Kubernetes Ingress Documentation: https://kubernetes.io/docs/concepts/services-networking/ingress/  

Thank you for following along with this full tutorial on ingress-nginx! Let us know if you have any other questions.
