# Getting Started with SpeQ

This guide will walk you through the standard developer workflow for writing and using `.speq` files in your software projects.

---

## 🌟 Step 1: Initialize Your Project Spec

A `.speq` file sits in the root directory of your repository. It acts as the immutable architectural contract for your system.

Create a file named `my_project.speq` and declare the system properties:

```speq
VERSION 0.3.0

PROJECT
  NAME   "todo_service"
  LANG   python
  STACK  fastapi, sqlite, pytest
  ARCH   layered
  DEPS
    RUNTIME  fastapi, pydantic
    DEV      pytest
```

---

## 🏗️ Step 2: Define Domain Entities & Vocabularies

To prevent the AI from fabricating entities or mixing up names, declare them in the `ENTITY` and `VOCABULARY` blocks:

```speq
# Declare the closed-world entities
ENTITY todo, user, session

# Establish precise naming terminology
VOCABULARY
  TodoId       # never: todo_id, task_uuid, id
  UserId       # never: user_id, uuid, uid
  SessionToken # never: token, session_id, jwt
```

---

## 🛡️ Step 3: Organize Layer Boundaries

Next, establish logical boundaries to keep your codebase clean and organized. Declare what each layer `OWNS`, what it `CALLS`, and what it is `NEVER` allowed to do:

```speq
LAYERS
  GATEWAY
    BOUNDARY  external
    EXPOSES   create_todo, get_todos, authenticate
    CALLS     SERVICE
    NEVER     access_database
    
  SERVICE
    OWNS      todo_validation, user_auth
    CALLS     DB
    
  DB
    OWNS      todo_persistence, user_persistence
    NEVER     execute_business_logic
```

---

## 🔒 Step 4: Add Invariant Contracts

Contracts ensure that your business logic remains secure and reliable across AI generations. Declare security rules and structured execution pipelines:

```speq
CLASSIFY
  user.password       credential
  session.token       credential
  user.email          pii

CONTRACTS
  # Invariants
  todo.title          ALWAYS   validated-not-empty
  user.email          ALWAYS   validated-email-format
  
  # A transactional flow
  FLOW create_todo
    1. [GATEWAY] todo.sanitize
    2. [SERVICE] todo.validate
    3. [DB]      todo.persist
    ATOMIC  true
    TIMEOUT 5s
```

---

## ⚡ Step 5: Start Your AI Coding Session

To utilize the spec, load the **[SpeQ SKILL](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/SKILL.md)** and your `.speq` file together at the start of your session:

```markdown
"I want to implement the Todo creation feature. 
Please read my_project.speq and SKILL.md. 
Maintain a closed-world assumption: do not create any entity, layer, or name not defined in the spec. 
Verify all contracts before finalizing your code."
```

By following this step-by-step workflow, you ensure that the generated codebase perfectly adheres to your target design.
