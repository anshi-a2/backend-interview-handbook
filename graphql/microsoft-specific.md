

## 🏢 Microsoft-Specific GraphQL Interview Questions (SDE-2)

### 1. How would you integrate GraphQL with a .NET backend?

* Use libraries like HotChocolate or GraphQL.NET.
* Map resolvers to existing service layer.
* Integrate with Azure Functions or Web API.

### 2. Microsoft services deal with massive telemetry data. How would you prevent query abuse in GraphQL?

* Use persisted queries.
* Depth/complexity limits.
* Azure API Gateway throttling.

### 3. How would you expose Teams/Outlook calendar data via GraphQL?

* Wrap REST APIs of Microsoft Graph with GraphQL resolvers.
* Handle authentication using Azure AD.

### 4. How do you handle authorization for different Azure AD roles in GraphQL?

* Inject claims/principal into GraphQL context.
* Add role-based checks in resolvers.

### 5. What telemetry would you add to monitor GraphQL APIs?

* App Insights, distributed tracing.
* Track slow queries, error rates per resolver.


