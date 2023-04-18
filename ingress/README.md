# Ingress With Traefik & SSL Certificate

## Create Secret

SSL certificates in kubernetes are stored as secrets.
To do this, create a TLS secret with the following command.

```bash

kubectl create secret tls example-co-zm-ssl --key private.key --cert cert.crt -n longhorn-system

```

The above command creates a TLS secret in the longhorn-system namespace.

We need to create another secret to store the basic authentication users, this can be done as:

```bash

htpasswd -nb arthur P@55w0rd | openssl base64

```

The add the output in the `secret.yaml` file, to look like:

```yaml

apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
  namespace: longhorn-system
data:
  users: |
    YXJ0aHVyOiRhcHIxJER1dXdPUmtMJGlsOFJiZnpiNjgzdGpPU0dLSGxNczAKCg==

```

With all the certififcates out of the way, we can now create the two middlewares, one for redirecting to https if traffic originates from http
and the other to request a basic authentication prompt for users to provide username and password to access the resource on the url.

## Basic Authentication

Create a basic authentication middleware with the following contents:

```yaml

apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: basic-auth
  namespace: longhorn-system
spec:
  basicAuth:
    secret: basic-auth-secret

```

> **Note**: This middleware contains the basic authentication secret that stores the username password combination for allowed users.

## HTTP to HTTPS Middleware

Creata middleware to route all HTTP to HTTPS with below content.

```yaml

apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: enforce-https
  namespace: longhorn-system
spec:
  redirectScheme:
    scheme: https
    permanent: true

```

> **Note**: The above middleware contains the scheme to redirect to,which is https and the permanent attribute set to true to make this a permanent 301 redirect.

With all that done, you can create an ingress that contains the following contents.

```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    traefik.ingress.kubernetes.io/router.middlewares: longhorn-system-enforce-https@kubernetescrd,longhorn-system-basic-auth@kubernetescrd
spec:
  tls:
    - secretName: example-co-zm-ssl
  rules:
  - host: storage.example.co.zm
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: longhorn-frontend
            port: 
              number: 80

```

The above ingress contains annotations that add two middlewares to this ingress.

**NOTE:** The middleware has a syntax that requires that the middleware semantic follow a pattern of:

```bash

< namespace >-< middleware >@kubernetescrd

```

Which for the basic-auth middleware which is in the longhorn-system namespace translates to `longhorn-system-basic-auth@kubernetescrd` 
The middlewares can be chained together by comma separating them.

THe `tls` part contains a list of TLS secrets, in our case, we only have one.

The `rules` dictate how route matching is processed, what service to connect to and the port on the service being connected to.

To apply these configurations, use `kubectl apply -f`

```bash

kubectl apply -f secret.yaml -f basic-auth-middleware.yaml -f https-middleware.yaml  -f longhorn-ingress.yaml

```

## Conclusion

This is with the assumption that your kubernetes cluster has traefik as the ingress controller and longhorn as the block storage controller already setup.
Otherwise,you might need tomodify this toi suit the ingress controller being used and the service being connected to.