+++
draft = true
title = "Connect lambda function to EC2 instance using SSH"
date = "2016-11-21T16:31:35+01:00"
tags = ["aws", "aws-cli", "lambda", "dynamodb", "ec2"]
categories = ["AWS"]
series = []
images = []
description = ""
summary = """
Lambda functions are very useful for cronjob, but eventually you will need to
run something inside your architecture or maybe check if your service is
running... use SSH power!
"""

+++

Lambda functions can now run in your VPC, AWS documentation is not clear
about how to setup your function. I got stuck thinking is a policy problem,
but it was in fact a firewall problem.

This time there are no magic scripts that works, you have to read a create
your own.

### Security groups

{{< highlight bash >}}
# ec2 is a security-group I assign to all ec2 instances
aws ec2 create-security-group --group-name ec2 --description "ec2 servers"
# lambda is the security group for lambda function that run in the VPC
aws ec2 create-security-group --group-name lambda --description "open port 22"
{{< /highlight >}}

### EC2 Firewall

* `ec2` **in** port 22 from `lambda`
* `lambda` **out** port 22 to `ec2`

{{< highlight bash >}}
LAMBDA_GROUP_ID=$(aws ec2 describe-security-groups --group-names lambda \
  --query 'SecurityGroups[0].GroupId' --output text)
EC2_GROUP_ID=$(aws ec2 describe-security-groups --group-names "ec2" \
  --query 'SecurityGroups[0].GroupId' --output text)

tee /tmp/json <<EOF >/dev/null
[{
  "IpProtocol": "tcp",
  "FromPort": 22,
  "ToPort": 22,
  "UserIdGroupPairs": [{
    "GroupId": "${EC2_GROUP_ID}"
  }]
}]
EOF
aws ec2 authorize-security-group-ingress --group-name ${GROUP_NAME} \
  --protocol tcp --port 22 --source-group lambda

aws ec2 authorize-security-group-egress \
  --group-id ${LAMBDA_GROUP_ID} \
  --ip-permissions file:///tmp/json

{{< /highlight >}}


### Create lambda function

{{< highlight javascript >}}

var SSH = require('simple-ssh');

exports.handler = function(event, context, lambda) {
  console.log(event);

  var ssh = new SSH({
      // host: '52.57.67.55',
        host: '172.31.13.128',
      user: 'ec2-user',
      key: '-----BEGIN RSA PRIVATE KEY-----\n' +
'...\n' +
'-----END RSA PRIVATE KEY-----'
  });

  ssh
  .exec('pwd', {
    out: console.log.bind(console),
    exit: function() {
        ssh.exec('echo "new queue"', {
          out: console.log.bind(console),
        });
        return false;
    }
  })
  .on('error', function(err) {
    console.log(err);
    lambda(err);
  })
  .on('ready', console.log.bind(console))
  //.on('close', console.log.bind(console))
  .start();
};


{{< /highlight >}}


### Tips

Lamda lack of persistent storage.
So I created [dynamodb-keyvalue](https://github.com/llafuente/dynamodb-keyvalue)
to store key/value data in dynamodb (the name was very clear :smile:)
