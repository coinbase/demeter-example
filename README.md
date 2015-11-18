# Demeter Example Project

This project contains sample configuration files that demonstrate how to use the Demeter command line tool.  In this example we show how to manage security groups accross two clouds (staging, production).

[**Demeter Source (cli)**](https://github.com/coinbase/demeter)


## Installation of the CLI

```
gem install demeter-cli
```

## Usage


### Environment configuration

For each environment you wish to use with Demeter you can create a config file for this environment with the AWS credentials for that account.

To start off create the credentials for your first environment.
```
git clone https://github.com/coinbase/demeter-example
cd demeter-example
```

Create a file called .env.staging
```
AWS_ACCESS_KEY=xxx
AWS_SECRET_KEY=xxx
AWS_REGION=us-east-1
```

See a test plan in action!
```
demeter plan -e staging
```

Output from the sample project will show the DIFF between reality and your configs.  No changes will be made yet.
```
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| ec2::infra/bastion                                                                                                                          |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"icmp", :from_port=>-1, :to_port=>-1, :cidr_block=>"0.0.0.0/0"}                                             |
| +       | ingress | {:protocol=>"tcp", :from_port=>22, :to_port=>22, :cidr_block=>"0.0.0.0/0"}                                              |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| ec2::ssh                                                                                                                                    |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"tcp", :from_port=>22, :to_port=>22, :cidr_block=>"10.0.0.0/8"}                                             |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| elb::engineering/toshi                                                                                                                      |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"tcp", :from_port=>443, :to_port=>443, :cidr_block=>"0.0.0.0/0"}                                            |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| ec2::engineering/toshi                                                                                                                      |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"tcp", :from_port=>22, :to_port=>22, :cidr_block=>"10.0.0.0/8"}                                             |
| +       | ingress | {:protocol=>"tcp", :from_port=>22, :to_port=>22, :source_security_group=>"security_group.ec2_infra_bastion.id"}         |
| +       | ingress | {:protocol=>"tcp", :from_port=>5000, :to_port=>5000, :source_security_group=>"security_group.elb_engineering_toshi.id"} |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| rds::engineering/toshi                                                                                                                      |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"tcp", :from_port=>5432, :to_port=>5432, :source_security_group=>"security_group.ec2_engineering_toshi.id"} |
| +       | ingress | {:protocol=>"tcp", :from_port=>5432, :to_port=>5432, :source_security_group=>"security_group.ec2_infra_bastion.id"}     |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| redis::engineering/toshi                                                                                                                    |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"tcp", :from_port=>6379, :to_port=>6379, :source_security_group=>"security_group.ec2_engineering_toshi.id"} |
| +       | ingress | {:protocol=>"tcp", :from_port=>6379, :to_port=>6379, :source_security_group=>"security_group.ec2_infra_bastion.id"}     |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| ec2::engineering/bitcoind                                                                                                                   |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
| +       | egress  | {:protocol=>"-1", :from_port=>0, :to_port=>0, :cidr_block=>"0.0.0.0/0"}                                                 |
| +       | ingress | {:protocol=>"tcp", :from_port=>18333, :to_port=>18333, :cidr_block=>"0.0.0.0/0"}                                        |
| +       | ingress | {:protocol=>"tcp", :from_port=>18333, :to_port=>18333, :cidr_block=>"0.0.0.0/0"}                                        |
| +       | ingress | {:protocol=>"tcp", :from_port=>18333, :to_port=>18333, :cidr_block=>"0.0.0.0/0"}                                        |
| +       | ingress | {:protocol=>"tcp", :from_port=>22, :to_port=>22, :cidr_block=>"10.0.0.0/8"}                                             |
| +       | ingress | {:protocol=>"tcp", :from_port=>22, :to_port=>22, :source_security_group=>"security_group.ec2_infra_bastion.id"}         |
| +       | ingress | {:protocol=>"tcp", :from_port=>8333, :to_port=>8333, :cidr_block=>"0.0.0.0/0"}                                          |
| +       | ingress | {:protocol=>"tcp", :from_port=>8333, :to_port=>8333, :cidr_block=>"0.0.0.0/0"}                                          |
| +       | ingress | {:protocol=>"tcp", :from_port=>8333, :to_port=>8333, :cidr_block=>"0.0.0.0/0"}                                          |
+---------+---------+-------------------------------------------------------------------------------------------------------------------------+
```

Once you're satisfied with Demeter's plan you can go ahead and run apply.  This will create or modify security groups as show in the plan.
```
demeter apply -e staging
```

## Demeter directory structure

`configs/*.yml` - you can define your own directory structure inside configs. Demeter recursively loads all the .yml files

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


## Types of Variables

### User defined variables.

`variables/global.yml` - loads keys and values in `global.` namespace
`variables/<env>.yml` - loads keys and values in `env.` namespace

Example variable references.
```
<% global.private_office_cidr_blocks %>
<% env.vpc_id %>
```

### Reference variables

Demeter uses the naming convention for referencing IDs by name. For example `<% security_group.<name>.id %>` will be substituted with the ID of this security group.  Demeter will lookup this name in AWS to determine the correct Security Group ID. 
Security Group names are converted to reference variables where the name is downcased and all special characters replaced with underscores.

Example:
`name: ec2::infra/bastion` becomes `security_group.ec2_infra_bastion.id`

### Demeter configuration file

Demeter uses YAML formated configuration files.

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

#### Naming conventions used in the example 
Suggested naming convention for security groups {ec2|redis|elb|...}::{org/repo_name}::{service_name}. 

You can also check AWS documentation what are the possible options you can pass when creating/updating security groups
* [**Security Group**](http://docs.aws.amazon.com/sdkforruby/api/Aws/EC2/SecurityGroup.html)
