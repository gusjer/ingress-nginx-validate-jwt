# ingress-nginx-validate-jwt

[![codecov](https://codecov.io/gh/IvanJosipovic/ingress-nginx-validate-jwt/branch/main/graph/badge.svg?token=hh1FWYrH5r)](https://codecov.io/gh/IvanJosipovic/ingress-nginx-validate-jwt)
[![GitHub](https://img.shields.io/github/stars/ivanjosipovic/ingress-nginx-validate-jwt?style=social)](https://github.com/IvanJosipovic/ingress-nginx-validate-jwt)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/ingress-nginx-validate-jwt)](https://artifacthub.io/packages/helm/ingress-nginx-validate-jwt/ingress-nginx-validate-jwt)
![Downloads](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fraw.githubusercontent.com%2Fipitio%2Fbackage%2Frefs%2Fheads%2Findex%2FIvanJosipovic%2Fingress-nginx-validate-jwt%2Fingress-nginx-validate-jwt%25252Fingress-nginx-validate-jwt.json&query=%24.downloads&label=downloads)
[![Docker Pulls](https://img.shields.io/docker/pulls/ivanjosipovic/ingress-nginx-validate-jwt?label=docker%20pulls)](https://hub.docker.com/r/ivanjosipovic/ingress-nginx-validate-jwt)

| :exclamation:  This project has been superseded by [OIDC-Guard](https://github.com/IvanJosipovic/OIDC-Guard), which provides authentication and authorization for both APIs and web applications, supporting JWT, cookie authentication, and more! |
|-----------------------------------------|

## What is this?

This project is an API server which is used along with the [nginx.ingress.kubernetes.io/auth-url](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md#external-authentication) annotation for ingress-nginx and enables per Ingress customizable JWT validation.

### Supports AMD64 and ARM64

## Install

```bash
helm repo add ingress-nginx-validate-jwt https://ivanjosipovic.github.io/ingress-nginx-validate-jwt

helm repo update

helm install ingress-nginx-validate-jwt \
ingress-nginx-validate-jwt/ingress-nginx-validate-jwt \
--create-namespace \
--namespace ingress-nginx-validate-jwt \
--set openIdProviderConfigurationUrl="https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"
```

### Options

- openIdProviderConfigurationUrl
  - OpenID Provider Configuration Url for your Identity Provider
- logLevel
  - Logging Level (Trace, Debug, Information, Warning, Error, Critical, and None)
- [Helm Values](charts/ingress-nginx-validate-jwt/values.yaml)

## Configure Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/auth-url: http://ingress-nginx-validate-jwt.ingress-nginx-validate-jwt.svc.cluster.local:8080/auth?tid=11111111-1111-1111-1111-111111111111&aud=22222222-2222-2222-2222-222222222222&aud=33333333-3333-3333-3333-333333333333
spec:
```

## Parameters

The /auth endpoint supports configurable parameters in the format of \{claim\}=\{value\}. In the case the same claim is called more than once, the traffic will have to match only one.

For example, using the following query string
/auth?  
tid=11111111-1111-1111-1111-111111111111  
&aud=22222222-2222-2222-2222-222222222222  
&aud=33333333-3333-3333-3333-333333333333  

Along with validating the JWT token, the token must have a claim tid=11111111-1111-1111-1111-111111111111 and one of aud=22222222-2222-2222-2222-222222222222 or aud=33333333-3333-3333-3333-333333333333

### How to query arrays
The /auth endpoint is able to query arrays. We'll use the following JWT token in the example.
```json
{
  "email": "johndoe@example.com",
  "groups": ["admin", "developers"],
}
```

Using the following query string we can limit this endpoint to only tokens with an admin group
/auth?  
groups=admin

### Inject claims as headers
The /auth endpoint supports a custom parameter called "inject-claim". The value is the name of claim which will be added to the response headers.

For example, using the following query string
/auth?  
tid=11111111-1111-1111-1111-111111111111  
&aud=22222222-2222-2222-2222-222222222222  
&inject-claim=email

The /auth response will contain header email=someuser@domain.com

### Inject claims as headers with custom name
The value should be in the following format, "\{claim name\},\{header name\}".

For example, using the following query string
/auth?  
tid=11111111-1111-1111-1111-111111111111  
&aud=22222222-2222-2222-2222-222222222222  
&inject-claim=email,mail

The /auth response will contain header mail=someuser@domain.com

Example Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app
  annotations:
    nginx.ingress.kubernetes.io/auth-url: http://ingress-nginx-validate-jwt.ingress-nginx-validate-jwt.svc.cluster.local:8080/auth?aud=11111111-11111-1111111111&inject-claim=groups,JWT-Claim-Groups&inject-claim=scope,JWT-Claim-Scope
    nginx.ingress.kubernetes.io/auth-response-headers: JWT-Claim-Groups, JWT-Claim-Scope
```

## Design

![alt text](https://raw.githubusercontent.com/IvanJosipovic/ingress-nginx-validate-jwt/main/docs/validate-jwt.png)

## Metrics

Metrics are exposed on :8080/metrics

| Metric Name  | Description |
|---|---|
| ingress_nginx_validate_jwt_authorized | Number of Authorized operations ongoing |
| ingress_nginx_validate_jwt_unauthorized | Number of Unauthorized operations ongoing |
| ingress_nginx_validate_jwt_duration_seconds | Histogram of JWT validation durations |

## Building locally
```
cd src/ingress-nginx-validate-jwt
docker build -t ingress-nginx-validate-jwt -f Dockerfile .
docker run ingress-nginx-validate-jwt -e "OpenIdProviderConfigurationUrl=https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"
```
