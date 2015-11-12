# Demeter Example Project

Sample configuration files that demonstrates how to use Demeter tool to manage security groups accross two clouds (staging, production). You can see how to reference security group ids and how to use global and env variables.


## Usage

You need to install latest version of Demeter which you can get here: [**Demeter Project**](https://github.com/coinbase/demeter)

Export AWS environment keys (AWS_ACCESS_KEY, AWS_SECRET_KEY, AWS_REGION) or create a local `.env.<environment>` file in the root folder. If you create a .env file it will be automaticly sourced once you run `demeter`.

## Demeter directory structure
```
├── configs
│   ├── engineering
│   │   ├── bitcoind.yml
│   │   └── toshi.yml
│   ├── infra
│   │   └── bastion.yml
│   ├── production.yml
│   └── staging.yml
└── variables
    ├── global.yml
    ├── production.yml
    └── staging.yml
```

`variables/global.yml` - loads keys and values in `global.` namespace
`variables/<env>.yml` - loads keys and values in `env.` namespace
`configs/*.yml` - you can define your own directory structure inside configs. Demeter recursively loads all the .yml files

Currently demeter only supports `security_group.<name>.id` that gets collected once demeter creates/describes security groups in AWS

### Demeter configuration file

Demeter uses a YAML formated configuration files. It currently supports `environments` and `security_groups`

``` yaml
environments:
- staging
- production
security_groups:
- name: ec2::infra/bastion
  vpc_id: <% env.vpc_id %>
  ingress:
  - protocol: tcp
    from_port: 22
    to_port: 22
    cidr_blocks:
    - <% global.private_office_cidr_blocks %>
  - protocol: icmp
    from_port: -1
    to_port: -1
    cidr_blocks:
    - 0.0.0.0/0
    source_security_groups:
    - sg-xxxxxx
    - <% env.ssh_security_group_id %>
    - <% security_group.ec2_infra_bastion.id %>
  egress:
  - protocol: -1
    from_port: 0
    to_port: 0
    cidr_blocks:
    - 0.0.0.0/0
```

#### environments
Environments in which to apply the config

#### security_groups > name
Suggested naming convention {ec2|redis|elb|...}::{org/repo_name}::{service_name}. 
We convert name to variable by converting all letters to lowercase and replacing all special charecters and empty spaces to underscore. 
Security group id for `ec2::infra/bastion` would be referenced by `<% security_groups.ec2_infra_bastion.id %>`

You can also check AWS documentation what are the possible options you can pass when creating/updating security groups
* [**Security Group**](http://docs.aws.amazon.com/sdkforruby/api/Aws/EC2/SecurityGroup.html)

