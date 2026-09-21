# PropertyMe MCP Plugin — Submission Test Cases

---

## Positive Test Cases

### TC-1: List managed properties

- **User prompt:** "What properties do we manage in the PropertyMe MCP portfolio?"
- **Expected behavior:**
    1. Call `create_session` to establish a session for the demo account.
    2. Call `list_portfolios` to verify the "PropertyMe MCP" portfolio is the active one (switching with `set_portfolio` if it isn't).
    3. Call `list_properties` against that portfolio.
    4. Return every managed property as a summary.
- **Expected result shape:** A bulleted list of properties, each showing the fields:

    - Address
    - Rooms (bed/bath/car)
    - Manager
    - Rent
    - Tenant
    - Property type (e.g. "Residential — House")
- **Fixture data required:** The demo portfolio contains at least 1 properties with known addresses, references, and types so the reviewer can verify the list matches.

### TC-2: Property search and full details

- **User prompt:** "Show me the property details for Church Terrace in the PropertyMe MCP portfolio."
- **Expected behavior:**
    1. Call `create_session` to establish a session for the demo account.
    2. Call `list_portfolios` to verify the "PropertyMe MCP" portfolio is the active one (switching with `set_portfolio` if it isn't).
    3. Call `search_properties("Church Terrace")` to resolve the property.
    4. Call `get_property` with the returned id to fetch full details (search-then-get workflow).
- **Expected result shape:** A single property profile for the matched property, with the fields:

    - Reference
    - Type
    - Status
    - Bedrooms / Bathrooms / Car spaces
    - Inspection frequency
    - Owner
    - Tenant
    - Managing team member
    - Rent
    - Rent paid to
    - Tenancy/agreement start
    - Periodic
    - Re-let
    - Created
    - Last updated

- **Fixture data required:** The portfolio contains exactly one property matching the search term "Church Terrace" (e.g. "Church Terrace, 24B" — a residential house with an owner, tenant, managing team member, rent, agreement dates, and weekly inspection frequency) so the reviewer can verify the profile fields are populated.

### TC-3: Jobs assigned to the current user

- **User prompt:** "What maintenance jobs are assigned to me in the PropertyMe MCP portfolio?"
- **Expected behavior:**
    1. Call `create_session` to establish a session for the demo account.
    2. Call `list_portfolios` to verify the "PropertyMe MCP" portfolio is the active one (switching with `set_portfolio` if it isn't).
    3. Call `get_current_user` to resolve the reviewer's identity.
    4. Call `list_jobs`.
    5. Filter to jobs whose assigned team member matches the reviewer.
    6. Present results with human-readable values.
- **Expected result shape:** A list of the reviewer's assigned jobs, each with the fields:

    - Job Number
    - Property address
    - Job summary
    - Closed On

- **Fixture data required:** The demo account is a team member with at least 1 assigned jobs (e.g. "Leaking tap — 12 Smith Street, Medium priority").

### TC-4: Lease renewals awaiting signatures

- **User prompt:** "Which lease renewals are awaiting signatures in the PropertyMe MCP portfolio?"
- **Expected behavior:**
    1. Call `create_session` to establish a session for the demo account.
    2. Call `list_portfolios` to verify the "PropertyMe MCP" portfolio is the active one (switching with `set_portfolio` if it isn't).
    3. Call `list_lease_renewals` with status "pending".
    4. Call `get_property` for each renewal's property to fetch property details.
    5. Present only renewals in that workflow state.
    6. Translate the internal status to "awaiting signatures" in the response.
- **Expected result shape:** A list of pending renewals, each with the fields:

    - Address
    - Rent
    - Owner
    - Tenant
    - Status
    - Last updated

- **Fixture data required:** The portfolio contains 1 renewals in the pending state

### TC-5: Team directory

- **User prompt:** "Who's on the team in the PropertyMe MCP portfolio?"
- **Expected behavior:**
    1. Call `create_session` to establish a session for the demo account.
    2. Call `list_portfolios` to verify the "PropertyMe MCP" portfolio is the active one (switching with `set_portfolio` if it isn't).
    3. Call `list_team_members`.
    4. Return the portfolio's staff with their roles.
- **Expected result shape:** A list of team members, each with the fields:

    - Display name
    - Role (rendered naturally — Admin, Standard, Limited, Read-only)
    - Email
    - Phone

- **Fixture data required:** The portfolio contains at least 1 team members with known names and roles.

---

## Negative Test Cases

Negative test cases are prompts where the plugin must **not** be invoked. Each passes when ChatGPT answers without calling any PropertyMe tool.

### TC-6: Recruitment request that uses the word "job"

- **User prompt / scenario:** "Create a job posting for a junior property manager role at my agency, with a short description and key responsibilities."
- **Expected behavior:**
    1. Do not call `create_session` or any other PropertyMe tool.
    2. Draft the job advertisement from general knowledge.
- **Tools triggered:** none
- **Why the plugin shouldn't trigger:** In PropertyMe a "job" is a maintenance job raised against a managed property. This request is about hiring staff and involves no property, supplier, or record in the user's portfolio. Invoking `add_job` or `search_jobs` here would confuse a recruitment task with maintenance work and could create or surface unrelated portfolio data.

### TC-7: General tenancy-law question

- **User prompt / scenario:** "How much notice does a landlord in NSW have to give before a routine inspection, and how often can they inspect?"
- **Expected behavior:**
    1. Do not call `create_session` or any other PropertyMe tool.
    2. Answer from general knowledge of NSW residential tenancy rules.
- **Tools triggered:** none
- **Why the plugin shouldn't trigger:** The question is about legislation, not about the user's own inspections, properties, or tenants. Calling `list_inspections` or `search_inspections` would return portfolio records that cannot answer the question and would add noise to the response.

### TC-8: Public rental-listing search

- **User prompt / scenario:** "Find me 2-bedroom apartments for rent in Brisbane under $650 a week."
- **Expected behavior:**
    1. Do not call `create_session` or any other PropertyMe tool.
    2. Answer as a consumer search: explain that ChatGPT cannot browse live listings, or suggest listing sites, as it would without the plugin installed.
- **Tools triggered:** none
- **Why the plugin shouldn't trigger:** The plugin only reads properties managed by the signed-in agency; it does not search the open rental market. Calling `search_properties` would return the agency's managed portfolio, which is not what the user asked for, and could expose tenanted properties as if they were available listings.

---
