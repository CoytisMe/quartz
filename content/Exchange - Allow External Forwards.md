---
category: how-to
tags:
  - exchange
  - email
  - forwarding
  - security
  - outbound
---
## Policy Way

Securty / Defender Admin Portal
Email and Collaboration > Policies and rules 
Threat Policies > Anti-Spam Policies > Anti-Spam Outbound

Edit protection settings
Change forwarding rules to On - Forwarding is enabled.

To allow for Specific users, create new policy, specify who, set same rule.

Save.


## Weird Connector Way (unverified)

Create a new connector

Use of connector set to 
- Use only when I have a transport rule set up that redirects messages to this connector.
Routing set to the same

Create a rule
Apply if the sender is in organisation

Do the following
redirect the message to the follow connector (above)

Except if
The sender is (whoever you want to send externally)

