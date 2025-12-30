LAB 20 — NORTHWIND RESEARCH  
DELEGATED PERMISSIONS (USER → APPLICATION → API)

PURPOSE OF THIS LAB  
This lab focuses exclusively on delegated permissions in Microsoft Entra ID.  
It demonstrates how an interactive user signs into an application, how that application accesses a custom API on the user’s behalf, and how Microsoft Entra ID enforces authorization through delegated consent.

This lab intentionally excludes application permissions and daemon access. Those are deferred to Lab 21.

────────────────────────────────────────────────────────
NARRATIVE — NORTHWIND RESEARCH PLATFORM
────────────────────────────────────────────────────────

BACKGROUND  
Northwind Research is a mid-sized firm that manages confidential research projects for external partners.  
The company centralizes identity and access control using Microsoft Entra ID.

PHASE 1 — INTERACTIVE USER ACCESS TO A CUSTOM API  

Northwind develops a web application called Research Hub.  
Employees sign in to Research Hub using their Microsoft Entra ID user accounts.

Research Hub does not store research data locally.  
Instead, it retrieves and updates records by calling a custom internal API named Research Data API.

REQUIREMENTS  

• Users must authenticate interactively  
• The application must access the API on behalf of the signed-in user  
• The API must be able to identify which user initiated the request  
• Administrators want to prevent repeated consent prompts for employees  

IDENTITY DESIGN  

To meet these requirements:  

• Research Data API exposes a delegated permission named access_as_user  
• Research Hub requests this delegated permission  
• An administrator grants delegated consent for all users  

RESULT  

• Users sign in successfully to Research Hub  
• Research Hub obtains delegated access tokens  
• Research Data API receives requests that include user context  
• No additional consent prompts appear for users  

────────────────────────────────────────────────────────
SECTION 2 — IDENTITY MODEL (LAB 20)
────────────────────────────────────────────────────────

This section defines the actors, identities, and authorization mechanics used in Lab 20.  
No configuration or commands occur here.

CORE ACTORS  

USER  
• Employee of Northwind Research  
• Authenticates interactively using Microsoft Entra ID  
• Does not call APIs directly  

RESEARCH HUB  
• Interactive client application  
• Represents the user-facing portal  
• Requests delegated access tokens  
• Calls Research Data API on behalf of the signed-in user  

RESEARCH DATA API  
• Custom resource API  
• Does not authenticate users  
• Relies on Microsoft Entra ID to validate access tokens  
• Enforces authorization using token claims  

MICROSOFT ENTRA ID  
• Identity authority  
• Issues tokens  
• Enforces authentication and authorization decisions  
• Stores consent and permission state as directory objects  

IDENTITY OBJECTS IN ENTRA ID  

APPLICATION OBJECTS (DEFINITIONS)  
• Research Hub (client application)  
• Research Data API (resource application)  

These define capabilities, not access.

SERVICE PRINCIPALS (TENANT-LOCAL IDENTITIES)  
• One service principal per application  
• Represent the actual security identities used at runtime  
• All authorization decisions reference service principals  

AUTHORIZATION MODEL USED  

DELEGATED AUTHORIZATION  

• User signs in interactively  
• Research Hub requests delegated scope access_as_user  
• Token represents both the user and the client application  
• Token contains scp (scope) claim  
• Authorization is enforced via an Oauth2PermissionGrant object  

CONSENT VS EXECUTION (CRITICAL DISTINCTION)  

CONSENT  
• Stored in the directory  
• Created by administrators or user approval  
• Exists independently of runtime activity  

EXECUTION  
• Occurs only when an OAuth flow runs  
• Produces tokens and sign-in logs  
• Makes applications visible to users  

LAB 20 PURPOSEFULLY FOCUSES ON BOTH CONFIGURATION AND EXECUTION  
Unlike earlier labs, this lab will ensure the user actually signs in and exercises the delegated permission.

END STATE FOR LAB 20  

By the end of Lab 20:  

• A user signs into Research Hub  
• Research Hub calls Research Data API using delegated permissions  
• Microsoft Entra ID issues delegated access tokens  
• User sign-in logs exist  
• Authorization behavior is observable and explainable  

This mirrors real enterprise usage and exam scenarios involving delegated permissions.

────────────────────────────────────────────────────────
WHAT LAB 20 DOES NOT COVER
────────────────────────────────────────────────────────

• No application permissions  
• No daemon or background services  
• No app-only tokens  
• No app role assignments  

These are intentionally excluded to keep the delegated model isolated and clear.

────────────────────────────────────────────────────────
FORECAST — LAB 21 (APPLICATION PERMISSIONS)
────────────────────────────────────────────────────────

BUSINESS EXPANSION CONTEXT  

As Northwind Research grows, a background service named Research Sync Service is introduced.  
This service synchronizes research data with external systems on a schedule.  
No user interaction is involved.

WHY LAB 21 IS NEEDED  

Delegated permissions are inappropriate because:  

• No user signs in  
• No user context should exist  
• Access must be controlled at the application level  

LAB 21 WILL INTRODUCE  

• Research Sync Service as a daemon application  
• Application permissions defined on Research Data API  
• App role assignments linking the service to the API  
• App-only access tokens  
• Service principal sign-in logs  
• Clear contrast with Lab 20 behavior  

KEY LEARNING PROGRESSION  

Lab 20 answers:  
“How does a user-driven application access a custom API?”  

Lab 21 answers:  
“How does a background service access the same API without a user?”  

Together, Labs 20 and 21 establish a complete, exam-aligned understanding of delegated versus application permissions in Microsoft Entra ID.

LAB 20 — EXECUTION SUMMARY
DELEGATED PERMISSIONS
USER → APPLICATION → CUSTOM API
(UI-ONLY)

WHAT THIS LAB EXECUTED

This lab executed a full delegated-permission flow up to the identity boundary, without requiring application code or a live API backend.

The goal was not application functionality.
The goal was identity correctness, authorization state, and observable execution evidence.

WHAT WAS CONFIGURED AND VERIFIED

A custom resource API was created.
That API exposed a delegated permission scope.
The scope existed as a directory object, not as consent.

An interactive client application was created.
The client application was configured as a Single-Page Application (SPA).
Redirect URI and token settings were configured to allow browser-based OAuth execution.

An enterprise application (service principal) was created automatically for the client.
Assignment was required.
A specific user was assigned.
Visibility to users was explicitly enabled.

Delegated permission request was configured.
The client application declared it wanted the delegated scope.
Admin consent was granted for all users.
This created an OAuth2PermissionGrant directory object.

No application code was introduced.
No API calls were required.
No database or server was involved.

WHAT EXECUTION PROVED

A real interactive user sign-in occurred.
Microsoft Entra ID authenticated the user.
Assignment and visibility gates were enforced.
Delegated consent was evaluated.
OAuth execution proceeded.
Token issuance was attempted.
The application redirect failed as expected due to no backend.

This proves that identity execution can succeed even when application runtime fails.

OBSERVABLE EVIDENCE

User sign-in logs were generated.
Multiple correlated sign-in events appeared.
One event showed primary authentication (password validation, “stay signed in”).
A subsequent event showed successful application sign-in with no authentication events triggered.

This demonstrates how Entra ID separates:
authentication
authorization
token issuance

Lack of authentication details on a given log entry does not mean authentication did not occur.
Authentication may have occurred earlier in the flow.

WHAT DID NOT OCCUR (INTENTIONALLY)

No API access occurred.
No access token was sent to the custom API.
No resource sign-in logs exist.

This is correct.
Delegated consent and execution do not require an API call to exist.

WHY THIS LAB MATTERS FOR SC-300 
This lab matters because it demonstrates how Microsoft Entra ID evaluates identity, authorization, and execution as separate concerns, even though exam questions often collapse them into a single narrative.

The exam frequently presents scenarios where:
Permissions appear correct.
Consent has been granted.
Sign-in logs show success.
Yet the application does not function.

This lab explains why that situation is not contradictory.

Microsoft Entra ID is responsible for identity decisions only.
Once authentication, authorization, and token issuance are complete, Entra ID hands control to the application.
If the application has no backend, no redirect endpoint, or no runtime logic, the launch fails even though identity succeeded.

This is why an application can be “correctly configured” in Entra ID and still fail at runtime.
Identity configuration is not application execution.

The lab also demonstrates how sign-in logs should be interpreted.
Authentication details appear only in the event where credentials are validated.
Subsequent successful sign-ins may show no authentication events because Entra ID is reusing an existing session.
This distinction is subtle and frequently misunderstood on the exam.

Finally, the lab makes explicit that visibility, assignment, and permissions are independent controls.
An application can be permissioned but invisible.
It can be visible but not assignable.
It can be assignable but not consented.
Understanding these separations is essential for reasoning through complex access questions.

--------------------------------------------------------------------

DELEGATED PERMISSIONS — WHAT THEY ACTUALLY REQUIRE

Delegated permissions are not a single configuration step.
They are a runtime relationship that only exists when four elements are aligned.

First, there must be a user.
Delegated permissions always represent user intent.
Without a user, delegated authorization cannot exist.

Second, there must be an interactive client application.
The client does not gain authority itself.
It acts as a delegate, requesting access on behalf of the user.

Third, there must be a delegated scope defined by a resource API.
Scopes describe what actions the API is willing to allow.
They are owned and defined by the resource, not the client.

Fourth, there must be a delegated consent record.
This record is stored as an OAuth2PermissionGrant.
It links the client service principal to the resource service principal.
It represents trust, not activity.

When execution occurs, Entra ID evaluates all four:
The user authenticates.
The client requests a token.
The consent grant is checked.
If valid, a token is issued containing delegated scopes.

This is why delegated permissions cannot exist without a user and cannot be used by background services.

--------------------------------------------------------------------

CONSENT VS EXECUTION — WHY BOTH ARE REQUIRED

Consent and execution solve different problems.

Consent answers:
Is this client allowed to request this permission from this resource?

Execution answers:
Did a real sign-in occur, and was a token issued?

Consent is static.
It is stored in the directory.
It exists even if no one ever signs in.

Execution is dynamic.
It occurs only when an OAuth flow runs.
It produces tokens and sign-in logs.

In this lab, consent existed before the user clicked the application.
Execution occurred only after the user launched the app.
This separation explains why permissions can look correct even when no activity is visible.

--------------------------------------------------------------------

VISIBILITY, ASSIGNMENT, AND SIGN-IN ELIGIBILITY

Visibility controls whether a user sees an application tile.
Assignment controls whether a user is allowed to sign in.
Permissions control what the application may request after sign-in.

None of these imply the others.

The lab demonstrated this explicitly.
The application was permissioned and consented before it was visible.
The user could not access it until visibility was enabled.
Once visible and assigned, sign-in execution occurred.

This separation is intentional and frequently tested.

--------------------------------------------------------------------

CORE SC-300 UNDERSTANDING FROM THIS LAB

Microsoft Entra ID enforces identity and authorization independently of application code.
Delegated permissions represent user-driven access.
Consent records trust; execution proves activity.
Sign-in logs must be interpreted as a sequence, not a single event.
Application visibility is not authorization.

This lab provides the mechanical understanding needed to reason through SC-300 scenarios without memorizing answers.


LAB 20 FINAL STATE

User-driven delegated authorization is correctly configured.
Identity execution is observable and explainable.
The system behavior matches real enterprise tenants.
This lab establishes the delegated-permission baseline.

NEXT LAB CONTEXT

The next logical lab replaces the user with an application.
The same API will be accessed using application permissions.
The authorization model, tokens, consent objects, and logs will differ.

That contrast is the core learning objective for Lab 21.

