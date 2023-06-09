ansible-role-cloud-load-balancer
================================

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)
[![](https://github.com/mtulio/ansible-role-cloud-load-balancer/actions/workflows/release.yml/badge.svg)](https://github.com/mtulio/ansible-role-cloud-load-balancer/actions/workflows/release.yml)
[![](https://github.com/mtulio/ansible-role-cloud-load-balancer/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/mtulio/ansible-role-cloud-load-balancer/actions/workflows/ci.yml)
[![](https://img.shields.io/ansible/role/59599)](https://galaxy.ansible.com/mtulio/cloud_load_balancer)

Ansible Role to manage Load Balancing in Cloud Environments.

Providers
---------

- Supported providers

| Provider | Offering |
| -- | -- |
| AWS | NLB |
| DigitalOcean | Default |
| Oracle Cloud | Network |

Requirements
------------


- Ansible Collection `amazon.aws`
```
ansible-galaxy collection install amazon.aws
```

- Ansible Collection `community.aws`
```
ansible-galaxy collection install community.aws
```

- Ansible [Collection](https://docs.ansible.com/ansible/latest/collections/community/digitalocean/index.html) `community.digitalocean`

```
ansible-galaxy collection install community.digitalocean
```

Role Variables
--------------

- `cloud_loadbalancers`: list of load balancers to be created on cloud provider.
- `cloud_loadbalancer_targets`: list of load balancers targets (upstreams) to be created on cloud provider.

<!--

Dependencies
------------

> A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

-->

Example Playbook
----------------

- Create NLB on AWS and register the DNS on Route53:

```yaml
- hosts: localhost
  roles:
  - role: mtulio.cloud_load_balancer
    cloud_loadbalancer_targets:
      - name: "kube-api-external"
        provider: aws
        protocol: tcp
        port: 6443
        target_type: ip
        vpc_name: "my-vpc"
        health_check_protocol: https
        health_check_path: /readyz
        health_check_port: 6443
        successful_response_codes: "200-399"
        health_check_interval: 10
        healthy_threshold_count: 2
        unhealthy_threshold_count: 2
        state: present
        modify_targets: no
        tags:
          cluster: kube-apiserver
    cloud_loadbalancers:
      - name: "kube-external"
        openshift_id: public
        provider: aws
        type: network
        scheme: internet-facing
        state: present
        tags:
          cluster: kube-apiserver
        subnets_discovery: yes
        vpc_name: "my-vpc"
        subnets_names:
          - "my-net-public-use1-1a"
          - "my-net-public-use1-1b"
          - "my-net-public-use1-1c"
        cross_zone_load_balancing: yes
        ip_address_type: ipv4
        listeners:
          - Protocol: TCP
            Port: 6443
            DefaultActions:
              - Type: forward
                TargetGroupName: "kube-api-external"
        register_dns:
          - zone: "example.com"
            record: "api.my-cluster.example.com"
            overwrite: yes
          - zone: "my-cluster.example.com"
            record: "api.my-cluster.example.com"
            private_zone: yes
            overwrite: yes
```

- Create Load Balancer on DigitalOcean and register the DNS on Domains:

```yaml
cloud_loadbalancers:
  - name: "kube-external"
    openshift_id: public
    provider: do
    vpc_name: "my-vpc"
    project: "my-project"
    region: "nyc3"
    redirect_http_to_https: no
    size: "lb-small"
    algorithm: round_robin
    enable_backend_keepalive: no
    enable_proxy_protocol: no
    forwarding_rules:
      - entry_protocol: tcp
        entry_port: 6443
        target_protocol: tcp
        target_port: 6443
        tls_passthrough: false
    health_check:
      check_interval_seconds: 10
      healthy_threshold: 5
      path: "/healthz"
      port: 6443
      protocol: "https"
      response_timeout_seconds: 5
      unhealthy_threshold: 3
    register_resources:
      - service: dns
        domain: "{{ cluster_state.zones.cluster }}"
        records:
        - name: "lb"
          type: A
        - name: "api"
          value: "lb"
          type: CNAME
```

License
-------

Apache-2.0

Author Information
------------------

[Marco Braga (@mtulio)](https://github.com/mtulio)
