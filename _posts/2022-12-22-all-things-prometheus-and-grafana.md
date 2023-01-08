---
layout: post
title: All Things Grafana
date: '2023-01-01T18:47:00.000+05:30'
author: Middlewarebytes
sitemap:
  lastmod: 2023-01-07
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags:
- Kubernetes
- Docker
- Prometheus
- Grafana
- Cloud native
- Harbor
readtime: true
---



## How to configure / enable persistence in Grafana

Just like any other parameters in Grafana, persistence too can  be enabled  or disabled on user's choice by activating the parameter which controls it. When we look at the Grafana chart we could see that the persistence parameter is present but disabled by default as below:

```yaml
persistence:
  type: pvc
  enabled: false
  # storageClassName: default
  accessModes:
    - ReadWriteOnce
  size: 10Gi
  # annotations: {}
  finalizers:
    - kubernetes.io/pvc-protection
  # selectorLabels: {}
  ## Sub-directory of the PV to mount. Can be templated.
  # subPath: ""
  ## Name of an existing PVC. Can be templated.
  # existingClaim:
  ## Extra labels to apply to a PVC.
  extraPvcLabels: {}
```
We can enable it by setting  it to `true` in  our  kube-prometheus-stack chart using a set directive or using a custom values yaml to override.

`--set garafana.persistence.enabled = true`

---

## How to configure LDAP in kube-prometheus-stack


LDAP can be configured in kube-prometheus stack by first enabling the LDAP plugin in the main configuration file as  well as specifying the path of the LDAP  speciif configuration file, the default path is /etc/grafana/ldap.toml


### Please follow the below steps:

1. Create a config file with all the configurations required to connect with  your LDAP server. I have created one sample here and named it as `ldap-toml`

```
[[servers]]
# Ldap server host (specify multiple hosts space separated)
# We use OpenLDAP but you can use any IP address or external host here
host = "IP of the LDAP server here"

# Default port is 389 or 636 if use_ssl = true
port = PORT OF THE LDAP SERVER HERE

# Set to true if LDAP server supports TLS
use_ssl = false 

# Set to true if connect LDAP server with STARTTLS pattern (create connection in insecure, then upgrade to secure connection with TLS)
start_tls = false 

# set to true if you want to skip SSL cert validation
ssl_skip_verify = true

# Search user bind dn
bind_dn = "base user name/email to find in LDAP"

# Search user bind password
bind_password = "base user password"

# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"
search_filter = "(sAMAccountName=%s)"

# An array of base dns to search through
search_base_dns = ["DC=aravali,DC=ind"]

#LDAP Group Admin DN
group_search_base_dns = ["OU=AravaliGroups,DC=aravali,DC=ind"]

#LDAP Group search filter
group_search_filter = "(member:1.2.840.113556.1.4.1941:=%s)"

group_search_filter_user_attribute = "dn"

### If you use LDAP to authenticate users but don’t use role mapping, and prefer to manually assign organizations and roles, you can use the `skip_org_role_sync` configuration option.

[[servers.group_mappings]]
group_dn = "cn=mycloud_aravali_devops,ou=privatecloud,ou=aravaligroups,dc=aravali,dc=ind"
org_role = "Admin"
grafana_admin = true


[[servers.group_mappings]]
    group_dn = "*"
    org_role = "Viewer"


# Specify names of the LDAP attributes your LDAP uses
[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "dn"
email =  "email"
```

2. Create a secret from the config file above from the below command:

```bash
kubectl create secret generic grafana-ldap-toml --from-file=ldap-toml
```

3. Pass the required attributes in order to configure LDAP in the  ( kube-prometheus-stack) chart using  a custom values.yaml file like below, be extra careful about the indentation here.

```yaml
kube-prometheus-stack:
  grafana:
    ldap:
      enabled: true
      existingSecret: "grafana-ldap-toml"
    grafana.ini:
      auth.ldap:
        enabled: true
        allow_sign_up: true 
        config_file: /etc/grafana/ldap.toml
```


4. Installing chart, finally we can install the kube-prometheus-stack helm chart in your cluster

`helm upgrade --install grafana grafana/grafana --namespace monitoring -f grafana-values.yaml`

5. Point your browser to the Grafana url and try to login with an existing LDAP user

6. Voila!! We are into the console :) If we go now to settings -> Users we can see that my username was created with role “Viewer”. Here we can set any role we might want for this user.

---

## How to enable SSL CA certificate in kube-prometheus-stack chart 

Again, ingress resource in Grafana is disabled by default but can be enabled through the custom values yaml which would override the defaut settings. 



```yaml
ingress:
  enabled: false
  # For Kubernetes >= 1.18 you should specify the ingress-controller via the field ingressClassName
  # See https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/#specifying-the-class-of-an-ingress
  # ingressClassName: nginx
  # Values can be templated
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  labels: {}
  path: /

  # pathType is only for k8s >= 1.1=
  pathType: Prefix

  hosts:
    - chart-example.local
  ## Extra paths to prepend to every host configuration. This is useful when working with annotation based services.
  extraPaths: []
  # - path: /*
  #   backend:
  #     serviceName: ssl-redirect
  #     servicePort: use-annotation
  ## Or for k8s > 1.19
  # - path: /*
  #   pathType: Prefix
  #   backend:
  #     service:
  #       name: ssl-redirect
  #       port:
  #         name: use-annotation
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

```

Alternatively, we can create a manifest for ingress resource as above and package it inside templates folder of the kube-prometheus-stack chart, the manifest can be created as below. Please note that this template is just a reference and  you can set it as per your wish.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}
spec:
  ingressClassName: {{ .Release.Name }}
  {{ if .Values.ingress.useTLS}}
  tls:
    - hosts:
        - {{ .Release.Name }}.{{ .Values.ingress.domain_name }}
      secretName: ingress-tls   # Put secret name here which contains the TLS cert for Ingress
  {{ end }}
  rules:
    - host: {{ .Release.Name }}.{{ .Values.ingress.domain_name }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Release.Name }}-grafana
                port:
                  number: 3000
```


---

## Configuring a custom dashboard for  displaying Harbor metrics

In this example, I have used Prometheus and Grafana with Harbor. Harbor is an open source cloud-native registry that can securely manage artifacts across cloud native compute platforms like Kubernetes and Docker. I had created a post on how to deploy Harbor in HA mode in k8s cluster some time back. You can read about it here.

For now let's assume you already have a Harbor instance UP and Running in some K8s  cluster and we want to monitor , visualize  it using Prometheus and Grafana. 

Harbor already exposes some key metrics to monitor and analyze  the performance of the harbor instance, since observablity is a key  requirement in production and using these metrics, we can make informed decisions proactively  as well as during issues. 

Harbor exposes  metrics from following three components :

1. exporter
2. core
3. registry

Some of the key metrics  such as ;

`harbor_project_total` = Total number of public and private projects
`harbor_project_quota_usage_byte` = Total used resources of a project

See [here](https://goharbor.io/docs/2.2.0/administration/metrics/#:~:text=Harbor%20metrics%20are%20available%20at%20%3Charbor_instance%3E%3A%3Cmetrics_port%3E%2F%3Cmetrics_path%3E%20based%20on,core%20Metrics%20provided%20by%20the%20docker%20distribution%20itself) for the list of the available Harbor metrics.


In order to collect harbor metrics using [prometheus](prometheus and grafana documentation) , enable exposing the metrics in the harbor.yml file , in helm chart the same can be done using the 


Set up a prometheus server, set up the prometheus documentation for detailed instructions on installing the prometheus.

Configure promethes configuration file to scrape harbor metrics. Below is an example scrape config but you can refer the official documentation of Prometheus for more available scrape config options.

```yaml
scrape_configs:
    - job_name: 'harbor-exporter'
      scrape_interval: 20s
      static_configs:
        # Scrape metrics from the Harbor exporter component
        - targets: ['<harbor_instance>:<metrics_port>']

    - job_name: 'harbor-core'
      scrape_interval: 20s
      params:
        # Scrape metrics from the Harbor core component
        comp: ['core']
      static_configs:
        - targets: ['<harbor_instance>:<metrics_port>']

    - job_name: 'harbor-registry'
      scrape_interval: 20s
      params:
        # Scrape metrics from the Harbor registry component
        comp: ['registry']
      static_configs:
        - targets: ['<harbor_instance>:<metrics_port>']
```

Once  you have configured Prometheus server to collect Harbor metrics,  you can use Grafana to visualize the data.
I have created a simple dashboard  for visalization some key Harbor metrics, you can download the  json file  of dashboard from this github repository and build on top of it. Feel free to create a PR on this repository so that others can benefit too :)


