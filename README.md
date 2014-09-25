CloudFormation template for a self-healing [Kestrel](https://github.com/twitter/kestrel) stack.

##### Warning
This is a work-in-progress and does not currently include service discovery or persistence. But if you don't care about those things, this template should suit you fine.

## Overview

This template bootstraps a Kestrel stack.

The stack runs a single server within an auto-scaling group.  If the server is terminated, the auto-scaling group will launch a replacement.

The template creates a security group for Kestrel clients, the id for which is exposed as an output (`ClientSecurityGroup`).

Note that this template must be used with Amazon VPC. New AWS accounts automatically use VPC, but if you have an old account and are still using EC2-Classic, you'll need to modify this template or make the switch.

## Usage

### 1. Clone the repository
```bash
git clone git@github.com:thefactory/cloudformation-kestrel.git
```

### 2. Create an Admin security group
This is a VPC security group containing access rules for Kestrel administration, and should be locked down to your IP range, a bastion host, or similar. This security group will be associated with the Kestrel server.

Inbound rules are at your discretion, but you may want to include access to:
* `22 [tcp]` - SSH port
* `2222 [tcp]` - text listen port
* `2223 [tcp]` - admin HTTP port
* `2229 [tcp]` - thrift listen port
* `22133 [tcp]` - memcache listen port

### 3. Launch the stack
Launch the stack via the AWS console, a script, or [aws-cli](https://github.com/aws/aws-cli).

See `kestrel.json` for the full list of parameters, descriptions, and default values.

Example using `aws-cli`:
```bash
aws cloudformation create-stack \
    --template-body file://kestrel.json \
    --stack-name <stack> \
    --capabilities CAPABILITY_IAM \
    --parameters \
        ParameterKey=KeyName,ParameterValue=<key> \
        ParameterKey=VpcId,ParameterValue=<vpc_id> \
        ParameterKey=Subnets,ParameterValue='<subnet_id_1>\,<subnet_id_2>' \
        ParameterKey=AdminSecurityGroup,ParameterValue=<sg_id>
```

### 5. Test Kestrel
Once the stack has been provisioned, try hitting the Kestrel admin interface at `http://<host>:2223/`. You will need to do this from a location granted access by the specified `AdminSecurityGroup`.

You should see a Kestrel-generated page:
```console
$ curl localhost:2223
<html>
<head>
    ostrich
</head>
<body>

<p>
    Might be cool to have something interesting here someday. For now: Hello!
</p>

</body>
</html>
```

Once you associate `ClientSecurityGroup` with your clients, you'll be able to start using Kestrel.