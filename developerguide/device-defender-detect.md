# Detect<a name="device-defender-detect"></a>

 Detect allows you to identify unusual behavior that may indicate a compromised device by monitoring the behavior of your devices\. Using a combination of cloud\-side metrics \(from AWS IoT\) and device\-side metrics \(from agents you install on your devices\) you can detect changes in connection patterns, devices that communicate to unauthorized or unrecognized endpoints, and changes in inbound and outbound device traffic patterns\. You create security profiles, which contain definitions of expected device behaviors, and assign them to a group of devices or to all the devices in your fleet\. Detect uses these security profiles to detect anomalies and send alerts via AWS CloudWatch metrics and AWS SNS notifications\. 

 Detect is capable of detecting a variety of security issues frequently found in connected devices: 

+ Traffic from a device to a known malicious IP address or to an unauthorized endpoint that indicates a potential malicious command and control channel\.

+ Anomalous traffic, such as a spike in outbound traffic, that indicates a device is participating in a DDoS\.

+ Devices with remote management interfaces and ports that are remotely accessible\.

+ A spike in the rate of messages sent to your account— so that a rogue device does not end up costing you in per\-message charges\.Use cases:

Measure attack surface  
You can use Detect to measure the attack surface of your devices\. For example, you can identify devices with service ports which are often the target of attack campaigns \(telnet service running on ports 23/2323, SSH service running on port 22, HTTP/S services running on ports 80/443/8080/8081\)\. While these service ports might have legitimate reasons to be used on the devices, they are also usually part of the attack surface for adversaries and carry associated risks\. After Detect alerts you to the attack surface, you can either decide to minimize it \(by eliminating unused network services\) or run additional assessments to identify security weaknesses \(for example, telnet configured with common, default or weak passwords\)\. 

Detect device behavioral anomalies with possible security root causes  
You can use Detect to alert you to unexpected device behavioral metrics \(the number of open ports, number of connections, an unexpected open port, connections to unexpected IP addresses\) which might indicate a security breach\. For example, a higher than expected number of TCP connections may indicate a device is being used for a DDoS attack\. A process listening on a port other than the one you expect may indicate a backdoor installed on a device for remote control\. You can use Detect to probe the health of your device fleets and verify your security assumptions \(for example, no device is listening on port 23 or 2323\)\. 

Detect a mis\-configured device  
A spike in the number or size of messages sent from a device to your account might indicate a mis\-configured device\. Such a device could increase your per\-message charges\. Similarly, a device with many authorization failures could require a re\-configured policy\.

## Concepts<a name="detect-concepts"></a>

metric  
 Detect uses metrics to detect anomalous behavior\. Detect compares the reported value of a metric with the expected value you provide\. These metrics can be taken from two sources: \(a\) cloud\-side metrics, and \(b\) device\-side metrics:   
\(a\) Abnormal behavior on the AWS IoT network is detected by using cloud\-side metrics such as the number of authorization failures, or the number or size of messages a device sends or receives via AWS IoT\.   
\(b\) Detect can also collect, aggregate, and monitor metrics data generated by AWS IoT devices, for example, the ports a device is listening on, the number of bytes or packets sent, or the device's TCP connections\.  
You can use Detect with cloud\-side metrics alone\. To use device\-side metrics you must first deploy an agent on your AWS IoT connected devices or device gateways to collect the metrics and send them to AWS IoT\. See [Sending Metrics from Devices](#DetectMetricsMessages) 

security profile  
A security profile defines anomalous behaviors for a group of devices \(a [ thing group](https://docs.aws.amazon.com/iot/latest/developerguide/thing-groups.html)\) or for all devices in your account, and specifies what actions to take when an anomaly is detected\. You can create a security profile and associate it with a group of devices using the console or with AWS IoT API commands\. Detect starts recording security\-related data and uses the behaviors defined in the security profile to detect anomalies in the behavior of the devices\. 

behavior  
A behavior tells Detect how to recognize when a device is doing something abnormal\. Each behavior consists of a name, a metric, an operator, a value, and, in some cases, a time period \(duration\)\. Any device action that does not match a defined behavior statement triggers an alert\. 

alert  
When an anomaly is detected, an alert notification can be sent via a CloudWatch metric or an SNS notification\. An alert notification is also displayed in the AWS IoT CDM console along with additional information about the alert, and a history of alerts for the device\. An alert is also sent when a monitored device stops exhibiting anomalous behavior or when it had been causing an alert but stops reporting for an extended period\.

## Metrics<a name="detect-metrics"></a>

------
#### [ aws:message\-byte\-size ]

The number of bytes in a message\.

------
#### [ more info \(1\) ]

Use this metric to specify the maximum or minimum size \(in bytes\) of each message transmitted from a device to AWS IoT\.

Source: cloud\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: bytes 

Example:

```
{
  "name": "Max Message Size",
  "metric": "aws:message-byte-size",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 1024
    }
  }
}
```

------

 

------
#### [ aws:num\-messages\-received / aws:num\-messages\-sent ]

The number of messages received or sent by a device during a given time period\.

------
#### [ more info \(2\) ]

Use this metric to specify the maximum or minimum number of messages that may be sent or received between AWS IoT and each device in a given period of time\.

Source: cloud\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: messages 

Duration: a non\-negative integer, valid values are 300, 600, 900, 1800 or 3600 seconds

Example:

```
{
  "name": "Out bound message count",
  "metric": "aws:num-messages-sent",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 50
    },
    "durationSeconds": "300"
  }
}
```

------

 

------
#### [ aws:all\-bytes\-out ]

The number of outbound bytes from a device during a given time period\.

------
#### [ more info \(3\) ]

Use this metric to specify the maximum or minimum amount of out\-bound traffic that a device should send, measured in bytes in a given period of time\.

Source: device\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: bytes 

Duration: a non\-negative integer, valid values are 300, 600, 900, 1800 or 3600 seconds

Example:

```
{
  "name": "TCP outbound traffic",
  "metric": "aws:all-bytes-out",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 4096
    },
    "durationSeconds": "300"
  }
}
```

------

 

------
#### [ aws:all\-bytes\-in ]

The number of inbound bytes to a device during a given time period\.

------
#### [ more info \(4\) ]

Use this metric to specify the maximum or minimum amount of in\-bound traffic that a device should receive, measured in bytes in a given period of time\.

Source: device\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: bytes 

Duration: a non\-negative integer, valid values are 300, 600, 900, 1800 or 3600 seconds

Example:

```
{
  "name": "TCP inbound traffic",
  "metric": "aws:all-bytes-in",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 4096
    },
    "durationSeconds": "300"
  }
}
```

------

 

------
#### [ aws:all\-packets\-out ]

The number of outbound packets from a device during a given time period\.

------
#### [ more info \(5\) ]

Use this metric to specify the maximum or minimum amount of total outbound traffic that a device should send in a given period of time\.

Source: device\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: packets 

Duration: a non\-negative integer, valid values are 300, 600, 900, 1800 or 3600 seconds

Example:

```
{
  "name": "TCP outbound traffic",
  "metric": "aws:all-packets-out",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 100
    },
    "durationSeconds": "300"
  }
}
```

------

 

------
#### [ aws:all\-packets\-in ]

The number of inbound packets to a device during a given time period\.

------
#### [ more info \(6\) ]

Use this metric to specify the maximum or minimum amount of total inbound traffic that a device should receive in a given period of time\.

Source: device\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: packets 

Duration: a non\-negative integer, valid values are 300, 600, 900, 1800 or 3600 seconds

Example:

```
{
  "name": "TCP inbound traffic",
  "metric": "aws:all-packets-in",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 100
    },
    "durationSeconds": "300"
  }
}
```

------

 

------
#### [ aws:num\-authorization\-failures ]

The number of authorization failures during a given time period\.

------
#### [ more info \(7\) ]

Use this metric to specify the maximum number of authorization failures allowed for each device in a given period of time\. An authorization failure occurs when a request from a device to AWS IoT is denied, for example, if a device attempts to publish to a topic for which it does not have sufficient permissions\. 

Source: cloud\-side

Unit: failures 

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: failures 

Duration: a non\-negative integer, valid values are 300, 600, 900, 1800 or 3600 seconds

Example:

```
{
  "name": "Authorization Failures",
  "metric": "aws:num-authorization-failures",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 5
    },
    "durationSeconds": "300"
  }
}
```

------

 

------
#### [ aws:source\-ip\-address ]

The IP address from which a device has connected to AWS IoT\.

------
#### [ more info \(8\) ]

Use this metric to specify a set of whitelisted or blacklisted CIDRs from which each device must or must not connect to AWS IoT\.

Source: cloud\-side

Operators: in\-cidr\-set | not\-in\-cidr\-set 

Values: a list of CIDRs

Units: n/a

Example:

```
{
  "name": "Blacklisted source IPs",
  "metric": "aws:source-ip-address",
  "criteria": {
    "comparisonOperator": "not-in-cidr-set",
    "value": {
      "cidrs": [ "12.8.0.0/16", "15.102.16.0/24" ]
    }
  }
}
```

------

 

------
#### [ aws:destination\-ip\-addresses ]

A set of IP destinations\.

------
#### [ more info \(9\) ]

Use this metric to specify a set of whitelisted or blacklisted CIDRs that each device must or must not communicate with\.

Source: device\-side

Operators: in\-cidr\-set | not\-in\-cidr\-set 

Values: a list of CIDRs

Units: n/a

Example:

```
{
  "name": "Blacklisted destination IPs",
  "metric": "aws:destination-ip-addresses",
  "criteria": {
    "comparisonOperator": "not-in-cidr-set",
    "value": {
      "cidrs": [ "12.8.0.0/16", "15.102.16.0/24" ]
    }
  }
}
```

------

 

------
#### [ aws:listening\-tcp\-ports / aws:listening\-udp\-ports ]

The TCP or UDP ports that the device is listening on\.

------
#### [ more info \(11\) ]

Use this metric to specify a set of whitelisted or blacklisted TCP/UDP ports that each device must or must not listen on\.

Source: device\-side

Operators: in\-port\-set | not\-in\-port\-set 

Values: a list of ports 

Units: n/a

Example:

```
{
  "name": "Listening TCP Ports",
  "metric": "aws:listening-tcp-ports",
  "criteria": {
    "comparisonOperator": "in-port-set",
    "value": {
      "ports": [ 443, 80 ]
    }
  }
}
```

------

 

------
#### [ aws:num\-listening\-tcp\-ports / aws:num\-listening\-udp\-ports ]

The number of TCP or UDP ports the device is listening on\.

------
#### [ more info \(12\) ]

Use this metric to specify the maximum or minimum number of TCP or UDP ports that each device should listen on\.

Source: device\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: ports 

Example:

```
{
  "name": "Max TCP Ports",
  "metric": "aws:num-listening-tcp-ports",
  "criteria": {
    "comparisonOperator": "less-than-equals",
    "value": {
      "count": 4
    }
  }
}
```

------

 

------
#### [ aws:num\-established\-tcp\-connections ]

The number of TCP connections for a device\.

------
#### [ more info \(13\) ]

Use this metric to specify the maximum or minimum number of active TCP connections that each device should have\. \(All TCP states\) 

Source: device\-side

Operators: less\-than | less\-than\-equals | greater\-than | greater\-than\-equals 

Value: a non\-negative integer 

Units: connections

Example:

```
{
  "name": "TCP Connection Count",
  "metric": "aws:num-established-tcp-connections",
  "criteria": {
    "comparisonOperator": "less-than",
    "value": {
      "count": 3
    }
  }
}
```

------

## How To Use Detect<a name="detect-HowTo"></a>

1. You can use Detect with just cloud\-side metrics, but if you plan to use device reported metrics, you must first deploy an agent on your AWS IoT connected devices or device gateways\. See [Sending Metrics from Devices](#DetectMetricsMessages)\. 

1. Create a set of behaviors that go in your security profile\. Behaviors contain metrics that specify normal behavior for a group of devices or for all devices in your account\. You can find more information, including examples, in [Metrics](#detect-metrics)\. After you have created a set of behaviors you can validate them with [ ValidateSecurityProfileBehaviors](DetectCommands.md#dd-api-iot-ValidateSecurityProfileBehaviors-doc)\. 

1. Use [ CreateSecurityProfile](DetectCommands.md#dd-api-iot-CreateSecurityProfile-doc) to create a security profile that includes your behaviors\. You can have alerts sent to a target \(an SNS topic\) when a device violates a behavior by using the `alertTargets` parameter\. \(If you do send alerts using SNS be aware that these will count against your account's SNS limit\. It is possible for a large burst of violations to exhaust your capacity\.\) 

1. Use [ AttachSecurityProfile](DetectCommands.md#dd-api-iot-AttachSecurityProfile-doc) to attach the security profile to a group of devices \(a thing group\) or to all the devices in your account\. Detect will begin checking for abnormal behavior and will send alerts if any behavior violations are detected\. 

   To attach a security profile to a group of devices, you must specify the ARN of the thing group which contains them\. A thing group ARN has the following format: 

   ```
   arn:aws:iot:<region>:<accountid>:thinggroup/<thing-group-name>
   ```

   To attach a security profile to all of the devices in an account, you must specify an ARN with the following format: 

   ```
   arn:aws:iot:<region>:<accountid>:all/things
   ```

1. You can also keep track of violations with [ ListActiveViolations](DetectCommands.md#dd-api-iot-ListActiveViolations-doc) which allows you to see what violations have been detected for a given security profile or target device\.

   Use [ ListViolationEvents](DetectCommands.md#dd-api-iot-ListViolationEvents-doc) to see what violations have been detected during a specified time period\. You can filter these results by a given security profile or device\. 

1. If you need to review the security profiles you have set up and the devices that are being monitored, use [ ListSecurityProfiles](DetectCommands.md#dd-api-iot-ListSecurityProfiles-doc), [ ListSecurityProfilesForTarget](DetectCommands.md#dd-api-iot-ListSecurityProfilesForTarget-doc), and [ ListTargetsForSecurityProfile](DetectCommands.md#dd-api-iot-ListTargetsForSecurityProfile-doc)\. 

   You can get more details about a particular security profile with [ DescribeSecurityProfile](DetectCommands.md#dd-api-iot-DescribeSecurityProfile-doc)\. 

1. To make changes to a security profile, use [ UpdateSecurityProfile](DetectCommands.md#dd-api-iot-UpdateSecurityProfile-doc)\. You may detach it from an account or target thing group with [ DetachSecurityProfile](DetectCommands.md#dd-api-iot-DetachSecurityProfile-doc) or delete a security profile with [ DeleteSecurityProfile](DetectCommands.md#dd-api-iot-DeleteSecurityProfile-doc)\. 

## Permissions<a name="device-defender-detect-permissions"></a>

This section contains information on how to set up the AWS IAM roles and policies required to manage Detect\. For more information on AWS IAM, see the [AWS Identity and Access Management User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)\.

### Give Detect permission to publish alerts to an SNS topic\.<a name="device-defender-detect-permissions-publish"></a>

If you use the `alertTargets` parameter in [ CreateSecurityProfile](DetectCommands.md#dd-api-iot-CreateSecurityProfile-doc), you must specify an IAM Role with two policies, a permissions policy and a trust policy\. The permissions policy grants permission to to publish notifications to your SNS topic\. The trust policy grants permission to assume the required role\.

#### permissions policy<a name="detect-account-sns-permissions-policy"></a>

```
{
  "Version":"2012-10-17",
  "Statement":[
      {
        "Effect":"Allow",
        "Action":[
          "sns:Publish"
        ],
        "Resource":[
          "arn:aws:sns:region:account-id:your-topic-name"
        ]
    }
  ]
}
```

#### trust policy<a name="detect-account-sns-trust-policy"></a>

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "iot.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

## Service Limits<a name="detect-limits"></a>

+ The maximum number of security profiles per target \(thing group or user account\) is 5\.

+ The maximum number of behaviors per security profile is 100\.

+ The maximum number of `value` elements \(counts, IP addresses, ports\) per security profile is 1000\. 

+ Device metric reporting is throttled to one metric per 5 minutes per device \(a device may not report more than one metric every 5 minutes\)\.

+  Detect violations are stored for 90 days after they have been generated\.

## Sending Metrics from Devices<a name="DetectMetricsMessages"></a>

 Detect can collect, aggregate, and monitor metrics data generated by AWS IoT devices to identify devices that are exhibiting abnormal behavior\. This section contains information on how to send metrics from a device to \.

You must securely deploy an agent on your AWS IoT connected devices or device gateways to collect device\-side metrics\. provides a sample agent to use as an example when creating your own\. Alternatively, you can provide device\-side metrics to in a pre\-determined format and frequency\. If you are unable to provide device metrics in one of these ways, you can still get limited functionality based on cloud\-side metrics\.

Please note the following:

+ A sample device metric reporting agent is currently available in Python on GitHub at [ https://github\.com/aws\-samples/aws\-iot\-device\-defender\-agent\-sdk\-python](https://github.com/aws-samples/aws-iot-device-defender-agent-sdk-python) \(specifically [ here](https://github.com/aws-samples/aws-iot-device-defender-agent-sdk-python/tree/master/samples/greengrass/greengrass_core_metrics_agent)\)\.

+ In order to report metrics, a device must be registered as a thing in AWS IoT\.

+ A device should, generally, send metrics once every 5 minutes\. Device metric reporting is throttled to one metric per minute per device \(a device may not report more than one metric per minute\)\.

+ Devices must report current metrics; historical metrics reporting is not supported\.

### Device Metrics Document Specification<a name="DetectMetricsMessagesSpec"></a>


**Overall Structure:**  

| Long Name | Short Name | Required | Type | Constraints | Notes | 
| --- | --- | --- | --- | --- | --- | 
| header | hed | Y | Object |  | Complete block required for well\-formed report\. | 
| metrics | met | Y | Object |  | Complete block required for well\-formed report\. | 


**Header Block:**  

| Long Name | Short Name | Required | Type | Constraints | Notes | 
| --- | --- | --- | --- | --- | --- | 
| report\_id | rid | Y | Integer |  | Monotonically increasing value, epoch timestamp recommended | 
| version | v | Y | String | Major\.Minor | minor increments with addition of field, major increments if metrics removed | 

**Metrics Block:**


**TCP Connections:**  

| Long Name | Short Name | Parent Element | Required | Type | Constraints | Notes | 
| --- | --- | --- | --- | --- | --- | --- | 
| tcp\_connections | tc | metrics | N | Object |  |  | 
| established\_connections | ec | tcp\_connections | N | List<Connection> |  | ESTABLISHED TCP State | 
| connections | cs | established\_connections | N | List<Object> |  |  | 
| remote\_addr | rad | connections | Y | Number | ip:port | ip can be ipv6 or ipv4 | 
| local\_port | lp | connections | N | Number | >= 0 |  | 
| local\_interface | li | connections | N | String |  | interface name | 
| total | t | established\_connections | N | Number | >= 0 | Number of established connections | 


**Listening TCP Ports:**  

| Long Name | Short Name | Parent Element | Required | Type | Constraints | Notes | 
| --- | --- | --- | --- | --- | --- | --- | 
| listening\_tcp\_ports | tp | metrics | N | Object |  |  | 
| ports | pts | listening\_tcp\_ports | N | List<Port> | > 0 |  | 
| port | pt | ports | N | Number | > 0 | ports should be numbers > 0 | 
| interface | if | ports | N | String |  | interface name | 
| total | t | listening\_tcp\_ports | N | Number | >= 0 |  | 


**Listening UDP Ports:**  

| Long Name | Short Name | Parent Element | Required | Type | Constraints | Notes | 
| --- | --- | --- | --- | --- | --- | --- | 
| listening\_udp\_ports | up | metrics | N | Object |  |  | 
| ports | pts | listening\_udp\_ports | N | List<Port> | > 0 |  | 
| port | pt | ports | N | Number | > 0 | ports should be numbers > 0 | 
| interface | if | ports | N | String |  | interface name | 
| total | t | listening\_udp\_ports | N | Number | >= 0 |  | 


**Network Stats:**  

| Long Name | Short Name | Parent Element | Required | Type | Constraints | Notes | 
| --- | --- | --- | --- | --- | --- | --- | 
| network\_stats | ns | metrics | N | Object |  |  | 
| bytes\_in | bi | network\_stats | N | Number | Delta Metric, >= 0 |  | 
| bytes\_out | bo | network\_stats | N | Number | Delta Metric, >= 0 |  | 
| packets\_in | pi | network\_stats | N | Number | Delta Metric, >= 0 |  | 
| packets\_out | po | network\_stats | N | Number | Delta Metric, >= 0 |  | 

Example JSON structure using long names:

```
{
    "header": {
        "report_id": 1530304554,
        "version": "1.0"
    },
    "metrics": {
        "listening_tcp_ports": {
            "ports": [
                {
                    "interface": "eth0",
                    "port": 24800
                },
                {
                    "interface": "eth0",
                    "port": 22
                },
                {
                    "interface": "eth0",
                    "port": 53
                }
            ],
            "total": 3
        },
        "listening_udp_ports": {
            "ports": [
                {
                    "interface": "eth0",
                    "port": 5353
                },
                {
                    "interface": "eth0",
                    "port": 67
                }
            ],
            "total": 2
        },
        "network_stats": {
            "bytes_in": 29358693495,
            "bytes_out": 26485035,
            "packets_in": 10013573555,
            "packets_out": 11382615
        },
        "tcp_connections": {
            "established_connections": {
                "connections": [
                    {
                        "local_interface": "eth0",
                        "local_port": 80,
                        "remote_addr": "192.168.0.1:8000"
                    },
                    {
                        "local_interface": "eth0",
                        "local_port": 80,
                        "remote_addr": "192.168.0.1:8000"
                    }
                ],
                "total": 2
            }
        }
    }
}
```

Example JSON structure using short names:

```
{
    "hed": {
        "rid": 1530305228,
        "v": "1.0"
    },
    "met": {
        "tp": {
            "pts": [
                {
                    "if": "eth0",
                    "pt": 24800
                },
                {
                    "if": "eth0",
                    "pt": 22
                },
                {
                    "if": "eth0",
                    "pt": 53
                }
            ],
            "t": 3
        },
        "up": {
            "pts": [
                {
                    "if": "eth0",
                    "pt": 5353
                },
                {
                    "if": "eth0",
                    "pt": 67
                }
            ],
            "t": 2
        },
        "ns": {
            "bi": 29359307173,
            "bo": 26490711,
            "pi": 10014614051,
            "po": 11387620
        },
        "tc": {
            "ec" : {
                "cs": [
                    {
                        "li": "eth0",
                        "lp": 80,
                        "rad": "192.168.0.1:8000"
                    },
                    {
                        "li": "eth0",
                        "lp": 80,
                        "rad": "192.168.0.1:8000"
                    }
                ],
                "t": 2
            }
        }
    }
}
```