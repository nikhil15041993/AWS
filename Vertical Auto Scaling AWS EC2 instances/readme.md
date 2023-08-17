## Vertical Auto Scaling AWS EC2 instances

we will use the following AWS services:

    AWS EC2
    IAM
    AWS Lambda
    AWS SNS
    CloudWatch


### Step 1  Create IAM Policy and attach to a Role

First of all, let’s go to IAM ==> Policies ==> Create Policy ==> JSON
and fill it with this content:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:ModifyInstanceAttribute",
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceAttribute",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```


After click Review policy, give it a name “ModifyInstance” and Create Policy.

Now we have completed this step, we need to create a role and bind it to our policy “ModifyInstance”.

In IAM, select Role ==> Create Role ==> AWS Service lambda. Select the previously created policy and then name it “RoleModifyInstance” Great, we’re done with IAM.

### Step 2. Create a Lamda Function

Now let’s create a lambda function.

Click to create a function, select Node.Js 12 or another, and name your function. For example “UpperInstance”.
At the very bottom, select the role you created earlier.

Add this content to Function code and change InstanceId:

```
const AWS = require('aws-sdk');
exports.handler = (event, context, callback) => {    
  const { instanceId, instanceRegion, instanceType } = process.env;
  const ec2 = new AWS.EC2({ region: instanceRegion});
  const params = {
    Attribute: 'instanceType',
    InstanceId: 'i-02322898c187abf84'
  };
  ec2.describeInstanceAttribute(params, (err, data) => {
    console.log(instanceType)

    if (err) {
      console.log(err, err.stack); 
    } else  {
      console.log(data); 
      const instanceTypeAttr = data.InstanceType.Value
      if (instanceType === instanceTypeAttr) {
        callback(null, 'Dont need to upgrade')
    console.log(instanceTypeAttr)
      } else {
        var wait = ms => new Promise(resolve => setTimeout(resolve, ms));
        Promise.resolve()
          .then(() => ec2.stopInstances({ InstanceIds: [instanceId] }).promise())
          .then(() => ec2.waitFor('instanceStopped', { InstanceIds: [instanceId] }).promise())
          .then(() => ec2.modifyInstanceAttribute({ InstanceId : instanceId, InstanceType: { Value: instanceType } }).promise())
          .catch(error => console.log('My Catch Error', error))
          .then(() => {
              return wait(3000)
          })
          .then(() => ec2.startInstances({ InstanceIds: [instanceId] }).promise())
          .then(() => callback(null, `Successfully modified ${event.instanceId} to ${event.instanceType}`))
          .catch(err => callback(err));
        } 
    };
  });
};
```
```
python code using boto3

import boto3


def lambda_handler(event, context):
    client = boto3.client('ec2')

    # Insert your Instance ID here

    my_instance = 'i-0cd1cecsdcdodid'  # Stop the instance
    client.stop_instances(InstanceIds=[my_instance])
    waiter = client.get_waiter('instance_stopped')
    waiter.wait(InstanceIds=[my_instance])  # Change the instance type
    client.modify_instance_attribute(InstanceId=my_instance,
            Attribute='instanceType', Value='t2.medium')  # Start the instance
    client.start_instances(InstanceIds=[my_instance])
```
After that, we need to set up “Environment variables”. Сlick edit and fill in the content we need.
```
key                  value
----------------------------
instanceId   
instanceRegion    us-eeast
InstanceType     c5xlarge
```

In “Basic settings”, change the timeout to 5 minutes.
Click “Save” and “Deploy” your lambda function.


In the same way, we create the “LowerInstance” function. Do not forget to just correctly indicate the types of instances that we need to increase and decrease.
That’s where we’re done with the function.


### Step 3. set up Amazon SNS service.

Go to the Topics section and create two topics named “underthreshold_25” and “underthreshold_60”.
After that, go to our topics and select “Create subscription”.

We select AWS Lambda for the protocol, and select our desired function as the endpoint. Feel free to click Create subscription and that’s it. We have two topics for reducing and increasing the server.

### Step 4 Create Cloudwatch Alarm

Now let’s navigate and configure our Cloud Watch. Select Alarm on the left and click Create Alarm ==> Select Metric ==> EC2 ==> Per-Instance Metrics ==> and select your instance named CPUUtilization metric and select metric. Here we set up as on the screen, we do not touch anything else.
```
Threshold type = static
when = lower
than 25
period = 5min
```

Click next and in Configure actions in the “Send a notification to …” section, select our SNS topic. When the threshold is set to 60 select underthreshold_60. It’s the same with underthreshold_25.

Now it’s time to test what we configured. We can create an overload or simply publish a message on our SNS topic. If you did that, now you have m5.large instance. If we publish a message in the underthreshold_25 topic, then our function will check the server and since it will have nothing to change, it will simply end and report this in the logs. It is what we wanted.

Now let’s post the message to underthreshold_60. The function for increasing the server takes the following steps in sequence:

    Checks instance type
    Stop instance
    Waits for it to stop
    Changes instance type
    Launch instance

When you republish the message, nothing changes.

With such settings, an increase will occur when the last three checks were on average above 60% in 15 minutes, and a decrease if below 25% in 45 minutes. We were completely satisfied with it, and of course, you can make your own changes.

