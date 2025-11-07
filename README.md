# Overview

Short guide for setup and usage of the Dynatrace [Remote MCP](https://docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai/dynatrace-mcp).

# Example prompts

Once configured, below are example prompts to try out.

### View available tools

Adjust name for your configuration:
```
show config and tools available in MCP Server named 'jde30943:dynatrace-remote-mcp'
```

### Dynatrace problems

Run one at a time:
```
list all problems in dynatrace from the last 24 hours

tell me more details for problem 4
```

### Write and run a DQL log querys from natural language

Run one at a time:
```
Create a DQL query to fetch the last 10 error logs

run that DQL

display those 5 of those log lines in a table
```

### Find a monitored entity from natural language

Run one at a time:
```
list all the services in dynatrace from the last 24 hours

Get all details of the entity 'BrokerService'
```

### Execute a DQL query

```
execute 
fetch logs, from: -24h | sort timestamp desc | summarize count(), by:{status}
```

### Explain a DQL query

```
What does this DQL do?
fetch logs | filter dt.source_entity == 'SERVICE-123' | summarize count(), by:{severity} | sort count() desc
```

### Get Help from Davis CoPilot

Run one at a time:
```
How can I investigate slow database queries in Dynatrace?

How can configure a tagging rule?
```

# Remote MCP Server Setup

## 1. Add Platform Token and IAM

Perform the following within [Dynatrace Account](https://myaccount.dynatrace.com/accounts) settings.

### 1.1 Create Platform Token

Within the account, goto this menu option `Identify & access management --> Platform Tokens` and add a new Platform token.   Save the generated token to a secure place for later usage.

```
davis-copilot:nl2dql:execute;
davis-copilot:dql2nl:execute;
davis-copilot:conversations:execute;
mcp-gateway:servers:invoke;
mcp-gateway:servers:read;
storage:buckets:read;
storage:logs:read;
storage:events:read;
storage:security.events:read;
storage:metrics:read;
storage:bizevents:read;
storage:spans:read;
storage:entities:read;
storage:smartscape:read;
storage:system:read;
```

Refer to [Dynatrace Platform Token](https://docs.dynatrace.com/docs/manage/identity-access-management/access-tokens-and-oauth-clients/platform-tokens) documentation for more details.

### 1.2. Add an IAM policy 

The user that creates the Platform token needs to have the same permission as the token.  So a policy must be made that the the user uses.

Within the account, goto this menu option `Identify & access management --> Policy Management` 

* Add a new policy. Suggested name `_MCP`.  
* Paste in these scopes to the policy.

```
ALLOW davis-copilot:nl2dql:execute;
ALLOW davis-copilot:dql2nl:execute;
ALLOW davis-copilot:conversations:execute;
ALLOW mcp-gateway:servers:invoke;
ALLOW mcp-gateway:servers:read;
ALLOW storage:buckets:read;
ALLOW storage:logs:read;
ALLOW storage:events:read;
ALLOW storage:security.events:read;
ALLOW storage:metrics:read;
ALLOW storage:bizevents:read;
ALLOW storage:spans:read;
ALLOW storage:entities:read;
ALLOW storage:smartscape:read;
ALLOW storage:system:read;
```

Refer to [Dynatrace IAM Policies](https://docs.dynatrace.com/docs/manage/identity-access-management/permission-management/manage-user-permissions-policies) documentation for more details.

### 1.3. Make a New Group

Within the account, goto this menu option `Identify & access management --> Group Management` 

You can add to an existing group or make a group with a name like `_MCP` and add the permission to the `_MCP` policy created in the previous step.

Refer to [Dynatrace Group Management](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-group-management) documentation for more details.

### 1.4. Add User making the platform token to the the policy

Within the account, goto this menu option `Identify & access management --> User Management` 

Edit the user and add them to the group `_MCP`

Refer to [Dynatrace User Management](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-user-management) documentation for more details.

## 2. Enable Davis Copilot

To enable Davis CoPilot on your environment, goto `Settings >  Dynatrace AI > Generative AI.`
* Turn on `Enable generative AI`
* Turn on `Enable document suggestions`
* Turn on `Enable environment-aware queries`

The `Configure data access` is not required unless you want to restrict access.

Refer to [Getting started with Davis Copilot](https://docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai/copilot/copilot-getting-started) documentation for more details.

## 3. Setup with in an IDE

In this example, I used VS Code for the `mcp.json` file. 
* Adjust `jde30943:dynatrace-remote-mcp` to your environment or preferred name.  
* Update `{PLATFORM TOKEN}` with the Platform token created above

```
{
    	"servers": {
		"jde30943:dynatrace-remote-mcp": {
			"url": "https://jde30943.apps.dynatrace.com/platform-reserved/mcp-gateway/v0.1/servers/dynatrace-mcp/mcp",
			"type": "http",
			"headers": {
					"Authorization": "Bearer {PLATFORM TOKEN}"
			},      
			"disabled": false
		}
}
```

Once added, start the MCP server and you will see the output
```
2025-11-07 15:11:27.397 [info] Starting server jde30943:dynatrace-remote-mcp
2025-11-07 15:11:27.398 [info] Connection state: Starting
2025-11-07 15:11:27.402 [info] Starting server from LocalProcess extension host
2025-11-07 15:11:27.403 [info] Connection state: Running
2025-11-07 15:11:28.027 [info] Discovered 4 tools
```

To verify, run this chat prompt to view the available tools

```
show config and tools available in MCP Server named 'jde30943:dynatrace-remote-mcp'
```


Refer to
* [Dynatrace MCP Remote](https://docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai/dynatrace-mcp) documentation for more details.
* [Visual Studio Docs](https://code.visualstudio.com/docs/copilot/customization/mcp-servers) for more details on MCP configuration.