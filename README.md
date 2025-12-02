# Overview

The Dynatrace Remote MCP Server seamlessly connects 3rd party AI agents to the Dynatrace platform, delivering real-time production context directly into your workflows. It provides secure, governed access to Dynatrace's high-quality data, deep contextual awareness, and deterministic intelligence, available from any MCP-enabled environment. Whether in your IDE, Atlassian Rovo, Microsoft Copilot, ChatGPT, or other tools, Dynatrace empowers you to infuse reliable, real-time insights into every workflow and transform the way you work.

This repo provides a guide for setup and usage of both:
1. Dynatrace Remote MCP Server currently in Preview - [Dynatrace Hub Tile](https://www.dynatrace.com/hub/detail/dynatrace-mcp-server)
1. Community-driven OpenSource Local MCP Server - [Dynatrace Hub Tile](https://www.dynatrace.com/hub/detail/local-mcp-server-1)

# Example prompts once installed

Once either the remote or local MCP servers are configured, below are example prompts to try them out.

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

# Dynatrace Prerequisites Setup

## 1. Add Platform Token and IAM

Perform the following within [Dynatrace Account](https://myaccount.dynatrace.com/accounts) settings.

### 1.1 Create Platform Token

Below are the scopes for the Remote MCP server. Additional Scopes may be required for Local MCP server base on the use case. Refer to [Scopes for Authentication](https://github.com/dynatrace-oss/dynatrace-mcp/tree/main?tab=readme-ov-file#scopes-for-authentication) README for the latest list.

* _NOTE: The Local MCP server has an option to prompt for user credentials when the MCP Server is started. This assumes your login has the proper scopes, but it saves the step of making a platform token. So for this use case of running the MCP Local without a platform token, you can skip this step._

Create Platform Token by going to the menu option `Identify & access management --> Platform Tokens` within account management and add a new Platform token. Save the generated token to a secure place for later usage.

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

Below are the scopes for the Remote MCP server. Additional Scopes may be required for Local MCP server base on the use case. Refer to [Scopes for Authentication](https://github.com/dynatrace-oss/dynatrace-mcp/tree/main?tab=readme-ov-file#scopes-for-authentication) README for the latest list.

Within [Account Management](https://myaccount.dynatrace.com/accounts), goto this menu option `Identify & access management --> Policy Management` 

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

Within [Account Management](https://myaccount.dynatrace.com/accounts), goto this menu option `Identify & access management --> Group Management` 

You can add to an existing group or make a group with a name like `_MCP` and add the permission to the `_MCP` policy created in the previous step.

Refer to [Dynatrace Group Management](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-group-management) documentation for more details.

### 1.4. Add User making the platform token to the the policy

Within [Account Management](https://myaccount.dynatrace.com/accounts), goto this menu option `Identify & access management --> User Management` 

Edit the user and add them to the group `_MCP`

Refer to [Dynatrace User Management](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-user-management) documentation for more details.

## 2. Enable Davis Copilot

To enable Davis CoPilot on your environment, goto `Settings >  Dynatrace AI > Generative AI.` within your Dynatrace Environment.
* Turn on `Enable generative AI`
* Turn on `Enable document suggestions`
* Turn on `Enable environment-aware queries`

The `Configure data access` is not required unless you want to restrict access.

Refer to [Getting started with Davis Copilot](https://docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai/copilot/copilot-getting-started) documentation for more details.

# MCP Server Setup with in an IDE

To get familiar and to use either the local or remote Dynatrace MCP Server, a development IDE like VS Code as described below can be used. The setup is similar, but do follow the guide for either the local or remote (or both) as shown below.

## Remote MCP Server

For quick reference, here is an example from the [mcp.json](mcp.json) file included in this repo. 

Using the example below:
* Adjust `jde30943:dynatrace-remote-mcp` server name to your Dynatrace environment or preferred name.  
* Adjust `url` to your Dynatrace environment  

```
{
	"servers": {
        "jde30943:dynatrace-remote-mcp": {
            "url": "https://jde30943.apps.dynatrace.com/platform-reserved/mcp-gateway/v0.1/servers/dynatrace-mcp/mcp",
			"type": "http",
            "headers": {
                "Authorization": "Bearer ${input:bearer_token}"
            }
		}
	},
	"inputs": [
        {
            "type": "promptString",
            "password": true,
            "id": "bearer_token",
            "description": "Platform Token"
        }
    ]
}
```

When you start the MCP server, you will be prompted to the `Platform Token`

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

## Local MCP Server

There are more examples in the [Configuration Section](https://github.com/dynatrace-oss/dynatrace-mcp/tree/main?tab=readme-ov-file#configuration) of the Local MCP Repo, but for quick reference, here is an example from the [mcp.json](mcp.json) file included in this repo. 

There are two ways to setup this up, with or without a Dynatrace platform tokens.

### Use Case 1: Without platform token

In this use case of no platform token, a browser window will open and prompt with your user credentials when the MCP Server is started. This assumes your login has the proper scopes, but it saves the step of making a platform token.

Using the example below:
* Adjust `jde30943:dynatrace-local-mcp` server name to your Dynatrace environment or preferred name.  
* Adjust `DT_ENVIRONMENT` to your Dynatrace environment  

```
{
	"servers": {
		"jde30943:dynatrace-local-mcp": {
			"command": "npx",
			"cwd": "${workspaceFolder}",
			"args": ["-y", "@dynatrace-oss/dynatrace-mcp-server@latest"],
      		"env": {
              "DT_ENVIRONMENT": "https://jde30943.apps.dynatrace.com"
            }
		}
	}
}
```

When you start the MCP server, a browser window open to prompt for credentials.

### Use Case 2: With platform token as a user input

In this use case a platform token, is created and used as an input.

Using the example below:
* Adjust `jde30943:dynatrace-local-mcp` server name to your Dynatrace environment or preferred name.  

```
{
	"servers": {
		"jde30943:dynatrace-local-mcp": {
			"command": "npx",
			"cwd": "${workspaceFolder}",
			"args": ["-y", "@dynatrace-oss/dynatrace-mcp-server@latest"],
      		"envFile": "${workspaceFolder}/.env"
		},
	},
	"inputs": [
        {
            "type": "promptString",
            "password": true,
            "id": "bearer_token",
            "description": "Platform Token"
        }
    ]
}
```

When you start the MCP server, you will be prompted to the `Platform Token`


# Reference Links

* [Dynatrace MCP Remote Server Help Docs](https://docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai/dynatrace-mcp)
* [Dynatrace MCP Local Server GitHub Repo](https://github.com/dynatrace-oss/dynatrace-mcp) 
* [Visual Studio CoPilot MCP configuration Docs](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)