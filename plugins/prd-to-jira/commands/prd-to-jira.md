# PRD to Jira

Break down a PRD into Jira tickets. Invoke the `prd-to-jira` skill to decompose a product requirements document into an epic with well-structured, right-sized tickets organized by work area.

## Arguments

- $PRD_SOURCE: (optional) Path to PRD file, URL, or Jira ticket key. If omitted, expects PRD to be pasted in conversation.

## Instructions

Use the Skill tool to invoke `prd-to-jira`, then follow its workflow. If $PRD_SOURCE is provided, ingest it first. Otherwise ask the user to paste or share the PRD.
