# Database

* Status: Accepted
* Deciders: Craig Gordon 
* Date: 2020-3-23

## Context and Problem Statement  
  
Catch The Run needs a database that will suit the data, architecture, and future scalability needs of the system.
  
## Decision Drivers  
  
* Data Model - CTR's data is highly related, with several many-to-many relationships between entities, and OLTP in nature.
* Cost - The database will likely be one of CTR's main operational costs. Methods to reduce database costs without significantly reducing functionality or increasing maintenance efforts should be pursued seriously.
* Cloud vs Self-Hosted - Managed cloud databases are much more convenient, and much easier to scale, but higher cost. In contrast, a self-hosted database could be the best fit for a monolithic architecture.
 
## Considered Options  
  
* Option 1: SQL
	* AWS RDS PostgreSQL initially, could transition to AWS Aurora or a different advanced cloud SQL solution later on.
	* The safest data model choice - though perhaps not as well-suited to the data as the graph model, SQL is a dependable choice in general.
	* Probably the cheapest cloud option.
	* Would require placing Subscription Included Games/Categories in JSON fields, rather than referencing the base entities, for acceptable event trigger query speed.
* Option 2: Wide Column / Document
	* This option is not currently in serious consideration, as CTR's data highly relational and not suited for non-graph NoSQL data models. It is only included because DynamoDB was used for CTR's initial development period.
* Option 3: Graph
	* Would enable Subscription Included Games/Categories to link to the base entities, instead of duplicating that data in JSON records.
	* Option 3.1: Neo4j Aura
		* Overall best cloud graph option.
		* Cheapest setting is $65/mo, which is reasonable but probably not the most cost-effective option.
	* Option 3.2: RedisGraph
		* This would potentially be both the cheapest and most performant option, but would require everything involved in self-hosting.
		* Currently does not support the entire Cypher query language - notably, does not support multiple labels per vertex, but this can be worked around with properties.

## Decision Outcome

Option 1: AWS RDS PostgreSQL. A solid initial pick which can be transitioned to a more specialized option if performance starts to lag.