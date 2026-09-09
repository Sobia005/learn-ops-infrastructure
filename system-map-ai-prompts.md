# System Map AI Prompts

## 1. Describe the System

❯ Draw the system architecture of this system. Find every service and every connection between them. For each connection note the direction and type (HTTP, database query, pub/sub event, etc.), port number, and frameworks like React, Django, etc. should be labeled.

❯ give me a list of What services exist?
  What framework does each service use?
  What ports do they use?
  How do the services communicate?
  Which direction does the communication go?
  Is the connection HTTP, database, pub/sub, etc.?

❯ For every service and connection you identified in the system architecture, show me the exact file path and relevant code/configuration that proves it exists.

  Create a table with these columns:
  Service or connection
  Exact file path
  What you found there
  Evidence
  Status: confirmed, planned, external, or assumed

  Do not rely on general knowledge. Only include something as confirmed if you can point to actual code or configuration in this repository.

  Also clearly separate:
  1. Services actually running in this repository
  2. External services the application communicates with
  3. Planned services that do not exist yet
  4. Connections that you inferred or assumed


 Add diagram to your memory.


 ❯ Check your memory for the system diagram. Convert it to a Mermaid flowchart. Use graph LR.

  I have some rules please follow them and make a system daigram
  - Each service is a node with a short label
  - Each connection is a directed edge
  - Label every edge with the connection type (HTTP, DB, pub/sub, etc.) and port if known
  - Do not add anything beyond services and their connections
  - Output only the Mermaid code block.



## 2. Convert to a Mermaid Diagram



❯ put this marmaid daigram in this file system-map-ai.md 
# System Map (AI)

  1. System Diagram