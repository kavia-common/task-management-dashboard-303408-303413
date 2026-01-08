# Task Management DB Schema and Seed Data

This database uses PostgreSQL and is initialized via one-off psql commands (no .sql migration files). Connection string is stored in db_connection.txt.

Connection:
- Use the exact line in db_connection.txt to connect with psql.
- Example: psql postgresql://appuser:dbuser123@localhost:5000/myapp

Schema:
- users
  - id SERIAL PRIMARY KEY
  - name VARCHAR(100) NOT NULL
  - email VARCHAR(255) UNIQUE NOT NULL
  - created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

- task_status (lookup)
  - id SMALLINT PRIMARY KEY
  - name VARCHAR(50) UNIQUE NOT NULL
  - Seeded values:
    - 1: "To Do"
    - 2: "In Progress"
    - 3: "Done"

- tasks
  - id SERIAL PRIMARY KEY
  - title VARCHAR(200) NOT NULL
  - description TEXT
  - status_id SMALLINT NOT NULL REFERENCES task_status(id)
  - due_date DATE
  - assignee_id INTEGER REFERENCES users(id) ON DELETE SET NULL
  - created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
  - updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

Indexes:
- CREATE INDEX idx_tasks_assignee ON tasks(assignee_id);
- CREATE INDEX idx_tasks_status ON tasks(status_id);
- CREATE INDEX idx_tasks_due_date ON tasks(due_date);

Trigger to auto-update updated_at:
- Function: set_updated_at()
- Trigger: trg_set_updated_at BEFORE UPDATE ON tasks

Seed data:
- Users: Alice Johnson (alice@example.com), Bob Smith (bob@example.com), Carol Lee (carol@example.com)
- Tasks:
  - "Design dashboard layout" (To Do) assigned to Alice, due in 7 days
  - "Implement API endpoints" (In Progress) assigned to Bob, due in 3 days
  - "Set up CI pipeline" (Done) assigned to Carol, due yesterday

Notes:
- Use individual psql -c "..." for each statement.
- If re-running seed inserts, they are safe due to ON CONFLICT where applicable and selection by email for assignee linking.
