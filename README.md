Copilot Studio Agent for Dataverse (using MCP)

A Copilot Studio agent that manages Dataverse records through chat, using the Dataverse MCP server. Instead of opening an app and filling in forms, you simply tell the agent what you want to do.

This repo is a write-up of what I built and what I learned along the way. I'm still learning Power Platform, so feedback is very welcome.

What it does
Reads and searches live records
Edits existing records (shows old value → new value before saving)
Deletes records (only after the exact record is verified and confirmed)
Creates records, if the MCP server exposes a creation tool (see notes below)
Tech stack
Microsoft Copilot Studio
Dataverse
Dataverse MCP Server (Preview)
How it works
The user types a request in plain English, for example "I want to edit a record".
The agent asks for a unique identifier (ID or another unique field).
The agent uses MCP to fetch the live record and shows its current values.
The user says which field to change and the new value.
The agent shows the proposed change and asks for confirmation.
Only after the user says yes, the agent performs the action and reports the result.

Delete follows the same flow: find the record, show key details, warn that deletion is permanent, ask again, then delete.

Example conversation
User:  I want to edit a record
Agent: Send me the record's ID or unique identifier.
User:  <record identifier>
Agent: Record found. Here are its current values... Which field do you want to change?
User:  <field name>
Agent: Current value is X. Available values are A, B, C. Which one should replace it?
User:  B
Agent: Proposed change: X → B. All other fields stay unchanged. Do you confirm?
User:  yes
Agent: Record updated successfully.
Agent instructions (generic version)

This is a simplified, sanitized version of the system prompt I used. Replace the placeholders with your own table and field names.

Role
You are an assistant for a Dataverse application. Help users query and manage
records in the <your table> table.

Tools and schema
- Use Dataverse MCP search to discover tables, columns and relationships.
- Use read_query for live records, filters, lookups, duplicate checks and counts.
- Reuse schema already retrieved in the conversation.
- Use only tools that MCP actually exposes.
- Never invent fields, IDs, relationships, query syntax or tool capabilities.

Lookups
- For every lookup field, use the verified target table and its correct ID.
- If two lookups look similar, treat them as separate fields. Never interchange IDs.

Editing records
- Identify the exact record first and show its current values.
- Ask which fields to change, then show old value → new value.
- Ask for explicit confirmation before updating.
- Report success only after MCP confirms it.

Deleting records
- Never treat the original delete request as confirmation.
- Verify the exact record, show its key details, and warn that deletion is
  permanent and may affect related data.
- Ask for explicit confirmation for that specific record.
- Delete only that one record, and report success only after MCP confirms it.

Safety
- Respect Dataverse permissions.
- Be concise and professional.
- Never fabricate data or claim an unconfirmed operation succeeded.
- Clearly explain tool errors and incomplete results.
What I learned
The agent can only use the tools MCP exposes. If a tool isn't available, the agent can't do that action, so the prompt must say clearly what it should do and what it shouldn't pretend to do.
Lookup fields need care. Two lookups can point to the same table while another points to a different one. Mixing up their IDs breaks the record, so the prompt names the exact target table and ID type for each lookup.
A "delete" request is not a confirmation. The agent must verify the record and ask again before deleting.
Showing old value → new value before an update builds trust. Users can see exactly what will change.
A strict, clear prompt matters as much as the tools.
Notes and limitations
I tested read, edit and delete flows. Creating records depends on whether your MCP server exposes an authorized creation tool; if it only exposes search and read, the agent should say creation is unavailable instead of pretending.
MCP for Dataverse is in preview, so behavior may change.
Always test with your own data in a non-production environment first.
Feedback

If you've built something similar with Copilot Studio and MCP, I'd love to hear how it went for you. Feel free to open an issue or connect with me on LinkedIn.
