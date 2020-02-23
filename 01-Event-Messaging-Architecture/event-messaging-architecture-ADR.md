# Event Messaging Architecture

* Status: Proposed  
* Deciders: Craig Gordon 
* Date: 2020-2-20

## Context and Problem Statement  
  
The core data flow of Catch The Run occurs like so:
1. Producer (LiveSplit Plugin)
2. Processors (EC2 Instances)
3. Consumers (Discord Clients / Web Browsers)

The question is how the data should get routed from the Producer to the Processors. Currently, the LiveSplit plugin is the only planned Producer manifestation, and it is being developed internally with hard-coded endpoint(s) included. However, in the future, other externally-developed applications may want to partake in the system, in both Producer and Consumer capacities.
  
## Decision Drivers  
  
* Latency - Producer to Consumers latency should be 2-3 seconds, ideally even lower
* External Producers - Should be able to securely and easily support externally-developed Producer applications that are interested in utilizing CTR's notifications system
* External Events Consumers - Should be able to securely and easily support externally-developed Consumer applications that are interested in receiving raw Events (rather than notifications)
* Security - Bad actors should not have a trivially easy time undermining the system, particularly on the Producer side
* Extensible - Can easily support new unforeseen workflows in terms of both data and communication
  
## Considered Options  
  
* Option 1: AWS EventBridge
	* Communication unit: Event
		* Abbreviated Example:
			* {
			   "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
			   "detail-type": "speedrun.pb",
			   "source": "livesplit",
			   "time": "2020-01-01T00:01:000Z",
			   "region": "us-west-1",
			   "detail":
			   { 
				   "producer": "cyghfer",
				   "game": "Mega Man 2",
				   "category": "Any%",
				   "split": "Wily 1",
				   "pace": "-0:00:05.000",
				   "message": "it's the run",
				   "origintimestamp": "2020-01-01T00:00:000Z"
			   }
		       }
	* Can output to Kinesis Data Stream for persistent Events stream/firehose
		* This would cost more money, probably
		* Would require Event Consumer applications to use the AWS Kinesis API
	* Latency is ~500 milliseconds
	* EC2 instances can act as native Event targets (without HTTP endpoint needed?)
	* Supports Rules for filtering and routing Events properly
* Option 2: AWS Simple Notification Service (SNS)
	* Communication unit: PublishRequest
		* Abbreviated Example:
			* {
			   "MessageId": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
			   "TopicArn": "arn:aws:sns:us-east-2:123456789012:MyTopic",
			   "Subject": "speedrun.pb",
			   "Timestamp": "2020-01-01T00:00:000Z",
			   "Message": "it's the run",
			   "MessageAttributes":
			   {
			      "source": { "Type": "String", "Value": "livesplit" },
			      "producer": { "Type": "String", "Value": "cyghfer" },
			      "game": { "Type": "String", "Value": "Mega Man 2" },
			      "category": { "Type": "String", "Value": "Any%" },
			      "split": { "Type": "String", "Value": "Wily 1" },
			      "pace": { "Type": "String", "Value": "-0:00:05.000" },
			      "origintimestamp": { "Type": "String", "Value": "2020-01-01T00:00:000Z" }
			   }
			   }
	* Latency is ~30 milliseconds (!!!)
	* Output HTTP (e.g. to webhook endpoints)
		* Replaces the need for a persistent Events stream/firehose with a slightly less robust option
		* Enables this option to be more generalized and less AWS-centric
	* EC2 instances will need an HTTP endpoint to receive messages at
	* Can fan-out basic SMS and Emails if ever needed
	* Supports Topics for filtering and routing Events properly
* Option 3: Pure AWS Lambda
	* Receive HTTP requests directly
	* Would require more manual configuration
	* Would lose guarantees / functionalities of the AWS push fan-out systems
	* Lowest latency (?)

## Decision Outcome

Option 3: Pure AWS Lambda. It is possibly the lowest-latency option, and regardless of anything else, we need something at the front of the processing layer that can verify API keys. Option 2 is an acceptable plan B if Option 3 does not perform according to expectations.