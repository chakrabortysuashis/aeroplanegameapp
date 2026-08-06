# create-spec

Generate formal specification documents from brief descriptions. Use this skill whenever you need to create structured, professional specifications for features, tasks, or projects - especially when the user provides a brief description and needs a formal document.

## Overview
This skill creates structured specification documents in Markdown format. It takes a specification identifier (like SPEC-001) and a brief description, then generates a comprehensive spec document with standard sections including overview, objectives, requirements, assumptions, constraints, acceptance criteria, and open questions.

## When to Use
- When users say things like: "Create a spec for the login feature" or "I need a specification for user profile management"
- When starting new feature development that requires documentation
- When stakeholders request formal requirements documents
- Whenever a structured specification is needed from a high-level description

## Output Format
The skill generates a Markdown file with the following structure:

```markdown
# SPEC-ID: Specification Title

## Overview
[Brief description of what the feature or system is]

## Objectives
[List of key goals and objectives]

## Requirements
### Functional Requirements
- [List of functional requirements]

### Non-Functional Requirements
- [List of non-functional requirements (performance, security, usability, etc.)]

## Assumptions
[List of assumptions made during specification]

## Constraints
[List of constraints and limitations]

## Acceptance Criteria
[List of criteria that must be met for the feature to be considered complete]

## Open Questions
[List of unresolved questions or areas needing clarification]

## References
[List of relevant documents, links, or references]
```

The file will be saved as `./claude/specs/SPEC-ID.md` where SPEC-ID is the identifier provided by the user.

## Instructions
1. When the user invokes `/create-spec <spec_id> <description>`, extract the spec_id and description
2. Create the directory `./claude/specs/` if it doesn't exist
3. Generate a specification document using the format above
4. Save it as `./claude/specs/<spec_id>.md`
5. Inform the user where the specification was saved

## Example
User input: `/create-spec SPEC-001 "User login with email and password"`

Output file: `./claude/specs/SPEC-001.md`

Content:
```markdown
# SPEC-001: User login with email and password

## Overview
A secure authentication system allowing users to log in using their email address and password.

## Objectives
- Provide secure user authentication
- Enable access to protected features
- Maintain user session security

## Requirements
### Functional Requirements
- Users can enter email and password to authenticate
- System validates credentials against user database
- Successful login creates a user session
- Failed login attempts are logged and limited
- Users can reset forgotten passwords

### Non-Functional Requirements
- Authentication process completes within 2 seconds
- Passwords are stored using strong, salted hashing
- System implements rate limiting to prevent brute force attacks
- All authentication traffic is encrypted via HTTPS

## Assumptions
- Users have valid email addresses
- Password reset functionality will be implemented separately
- User database already exists with email as unique identifier

## Constraints
- Must comply with OWASP authentication guidelines
- Must be compatible with existing session management system
- Development timeline: 2 weeks

## Acceptance Criteria
- Given valid credentials, when user submits login form, then they are logged in and redirected to dashboard
- Given invalid credentials, when user submits login form, then they see an error message
- Given 5 consecutive failed attempts, when user attempts login again, then account is temporarily locked

## References
- OWASP Authentication Cheat Sheet
- Existing user database schema
- Session management documentation
```