# Data Model AI Prompts

## 1. Find the Database Connection Details

Where is the database? What host, port, database name, username, and password does the project use?


## 2. Identify the Database Type

What database engine does this application use, and where is it defined?

## 3. Map the ORM to the Database

I need to understand how this application connects Python models to the database.

First, identify the ORM this Django application uses and find where the database connection is configured. Include the ENGINE value from the settings.

Then look in LearningAPI/models/ and choose one model. Show me:
- The Python model name
- The database table name
- Each Python field/property name
- The corresponding database column name
- The database data type

Finally, open LearningAPI/views/book_view.py and find the create method. Explain what book.save() does under the hood and what SQL operation the Django ORM performs when it saves a new book.

Please include the file paths where you found the information.


## 4. Generate a Database Diagram


Generate an entity-relationship diagram (ER diagram) for the entire database used by this Django project.

Inspect all Django model files under learn-ops-api/LearningAPI/models/ and identify:
- Every database table/model
- Every field/column and its data type
- Primary keys
- Foreign keys
- All relationships between tables

Include one-to-one, one-to-many, and many-to-many relationships where they exist.

Use the actual Django model definitions to determine the relationships. Do not guess or leave out tables or fields.

Output the diagram as a Mermaid erDiagram code block that I can copy directly into a Markdown file.

Also briefly list the model files you inspected so I can verify the diagram against the project.

Do not include explanations inside the Mermaid diagram. Keep the diagram complete and accurate.



## 5. Find Relationship Examples

Find one example each of a one-to-one, one-to-many, and many-to-many relationship in the Django models for this project.

Look through the model files under `learn-ops-api/LearningAPI/models/`.

For each relationship, give me:
- The relationship type
- The two models involved
- The exact field name that defines the relationship
- The file path where the relationship is defined
- A short explanation of how the relationship works

Only report relationships that actually exist in the Django model code. Do not guess.

If a particular relationship type does not exist in the project, clearly say that it was not found.