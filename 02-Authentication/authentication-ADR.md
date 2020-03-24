# Authentication

* Status: Accepted
* Deciders: Craig Gordon 
* Date: 2020-3-23

## Context and Problem Statement  
  
Catch The Run is a system with two primary types of users: Producers and Consumers. There are multiple client applications that each class of user will use to take part in the system, and each application needs authentication in place.
  
## Decision Drivers  
  
* Universal - It is highly preferable that there be one source of truth for authenticated states. 
* Versatility - The solution needs to support several unique authentication flows.
* Existing System - The solution should use an existing authentication solution. CTR's identity needs are complicated, and a custom solution would be error-prone and time-consuming to implement.
* Vendor Lock-in - CTR is initially only targeting Twitch as the stream platform that notifications will be geared toward. On the one hand, it would be useful to leverage Twitch's authentication system as a convenience to users, but should CTR want to expand to other stream platforms, or decouple its authentication system from Twitch's due to future complications, the decoupling process might prove challenging.
* Rate Limits - If leveraging an existing authentication system, rate limits may become a concern if CTR's userbase grows to a considerable size.

* Client applications that need authentication implemented:
	* Producer
		* LiveSplit
		* External Partner Applications
		* Website
	* Consumer
		* Twitch Extension
		* Discord
		* Website
 
## Considered Options  
  
* Option 1: Heterogeneous Authentication
	* CTR operates across different distinct domains: Twitch, Discord, CTR-originating systems, and others.
	* It may be the most convenient for users in each particular domain to authenticate in a way that makes the most sense for that domain.
		* Twitch: Twitch OAuth
		* Discord: Must be authenticated to Discord in order to use Discord at all.
		* CTR and others: Flexible, but would need a token-based system beyond Basic Authentication.
	* Problems might arise with associating user accounts in different domains owned by the same person to a single underlying user record in the database. For example, a Consumer operating in the Discord domain needs to have their Discord account associated with their Twitch account, and this is impossible without requiring them to authenticate their Twitch credentials while concurrently authenticated with Discord.
* Option 2: Auth0
	* Cutting-edge, comprehensive, and trusted authentication solution.
	* Auth0 does not currently have a Twitch integration.
	* Would cost money to use as our userbase grows.
	* Although this would make CTR's authentication system independent from Twitch's authentication system, this would be more inconvenient for, at the very least, CTR's initial wave of users.
* Option 3: Pure Twitch OAuth
	* Because Twitch is the core platform that CTR will operate around for the foreseeable future, with Producers requring an account and Consumers likely already owning an account, it makes sense to leverage its authentication system directly.
	* Both Producers and Consumers will need to authenticate on every distinct domain using a login form that either redirects to a Twitch OAuth flow or authenticates against a stored Twitch-provided token. This will reduce the complexity of the authentication solution across the entire system.
	* This will be the most convenient option for the majority of users, as no new account or credentials creation will be required if a user already owns a Twitch account. For those that don't own one, creating a Twitch account is a quick process.
	* The new Twitch API features rate-limiting. This might become a problem with a sufficiently large userbase, or a large sudden influx of users, but it could probably be mitigated in various ways. 
	* The Twitch API could also suddenly change at any time, though since Twitch recently announced major changes to their API, they are unlikely to announce significant additional changes in the near-term.

## Decision Outcome

Option 3: Pure Twitch OAuth. It will work well for the initial needs of the system, and will be convenient for most users. Transitioning to a different or more generalized authentication system in the future as system needs change will have to be bridged when the time comes.