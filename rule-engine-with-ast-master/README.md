# Application 1 : Rule Engine with AST

> Zeotap | Software Engineer Intern | Assignment | Application 1

### Installation

I have tried my best to make it platform agnostic, and packaged everything into a
Docker Container. You can also execute the below seperately for running them on
your machine without Docker.

- Frontend
  ```bash
  cd frontend/
  npm install
  ```
- Backend

  If `poetry` isn't previously installed, install it first.

  ```bash
  python3 -m venv $VENV_PATH
  $VENV_PATH/bin/pip install -U pip setuptools
  $VENV_PATH/bin/pip install poetry
  ```

  Then continue to create a venv, and install dependencies

  ```bash
  cd backend/
  poetry install
  ```

- Database

  After a `poetry install`, execute the following to setup raw data in your
  postgres instance. Make sure to populate the connection creds in the `.env`.

  ```bash
  docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres
  ```

### How to run

Expecting that above installation process, suceeded.

- Frontend
  ```bash
  npm run dev
  ```
- Backend
  ```bash
  poetry run python main.py --dev
  ```
- Database
  ```bash
  docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres
  ```

### Solution Overview

The solution is pretty simple, I'll jot down a few bullets about the solution -

- The UI just makes HTTP calls to the REST API and visualizes the AST using `react-d3-tree`.
- The backend parses the raw string by first tokenizing using `tokenizer()`. We have a specified
  regex to look for lexems during the tokenization process. Then it parses the stream of tokens
  using a `Parser` object. This outputs a AST node.
- We store this AST in database by serializing it to JSON, and storing the JSON. ALthough there
  could be efficient ways to store in the database, like hashing the Node object and storing it
  in a KV store, with hash : Node object storage. This way we can traverse the tree for changes, and
  save only the nodes which went through a change. We can have dynamic pointers associated with each Node
  so that we can optimize for space too. Like two trees having same condition, will be stored only once.
  But a pointer and an associated entry will be made in a table. This abstraction can save space while
  still maintaining object isolation (for data security). Something similar to how Dropbox stores files
  on its servers.

### Code Structure

This section will talk about files, and what they do.

- Backend
  file_path | description
  ----------------|-----------------------
  main.py | Entrypoint of the server
  poetry.lock | Dependencies for the server, managed by poetry
  pyproject.toml | Py Project Metadata
  rule_engine/ast_utils.py | Contains Class definitions and Utility functions around the AST like Node, etc
  rule_engine/database.py | ORM and Functions to read/write through database
  rule_engine/main.py | Entrypoint to the API, contains API contracts
  rule_engine/models.py | DB Table Schema
  rule_engine/parser_utils.py | Parser and Tokenizer definitions and utilities

- Frontend
  file_path | description
  ----------------|-----------------------
  app/ | Source Code of the App
  app/components/ASTTree.tsx | Component to visualize AST
  app/services/api.ts | Middleware/Utility to connect to REST API
  app/page.tsx | Entrypoint to the UI, renders the main pageV
