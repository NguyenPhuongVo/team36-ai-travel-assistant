# Sprint 2: Codebase and Development Environment Preparation

## Task objectives
Prepare the YAF codebase and development environment for Sprint 2 implementation
- Review the repository areas relevant to the planned chatbot improvements.
- Set up the local development environment and required dependencies.
- Identify the configuration, API credentials, and sandbox access needed. 
- Check that the existing application can run locally and record any issues.
- Document the setup instructions and any outstanding requirements the team needs to address before moving into Sprint 2.

## Relevant Repository Areas
The following areas of the YAF repository are relevant to Sprint 2 development.

| Area to inspect | Main locations | Sprint 2 relevance |
| --- | --- | --- |
| AI request flow | `internal/ai/handler.go`, `orchestrator.go`, `router.go`, `provider.go` | Defines the chatbot’s main processing path. Sprint 2 will extend this flow so that user questions can trigger data retrieval before a response is sent to an AI provider. |
| Application wiring | `cmd/api/main.go` | Creates and connects the application’s providers, services, retrievers, authentication, and HTTP routes. Any new chatbot integration must be registered here or it will not be available at runtime. |
| Retrieval boundary | `internal/ai/retriever.go`, `internal/ai/pinecone/retriever.go` |  Defines the interface for supplying relevant information to the AI prompt. The team must decide whether to use Pinecone Assistant’s built-in retrieval or implement YAF-managed retrieval from application data. |
| Airport and flight data | `internal/airport/handler.go`, `service.go`, `model.go`, `repository_postgres.go`, `airlabs.go` | Contains airport records, accessibility services, flight-provider logic, and Airlabs integration. Sprint 2 can reuse these components to answer airport-accessibility and flight-status questions through the chatbot. |
| Airline data | `internal/airline/handler.go`, `service.go`, `model.go`, `repository_postgres.go` | Retrieves airline accessibility policies, contacts, and equipment rules. This data can be retrieved and provided to the AI when users ask about airline accessibility requirements. |
| Equipment rules | `internal/equipment/handler.go`, `service.go`, `model.go`, `repository_postgres.go` | Performs checks for wheelchair dimensions, equipment weight, battery capacity, battery type, and quantity limits. These calculations should remain consistent and predictable, while the AI explains the result in user-friendly language. |
| Hotel/booking integration | No original hotel package or provider exists | Select and integrate at least one approved hotel or booking sandbox. |
| Database structure | `pkg/database/`, SQL/schema files, `docs/erd.md` | Check what data already exists before adding new tables or persistence. Furthermore the team will need to decide whether hotel and booking data needs to be stored locally. |
| Authentication and privacy | `internal/middleware/`, `internal/user/` | Sprint 2 must ensure that personalized equipment information is retrieved only for the authorized user and is not exposed to other users or shared AI knowledge bases. |
| Configuration | `config/config.go`, `.env.example`, `cmd/api/main.go` | Configure AI providers, Airlabs, Pinecone/OpenAI credentials, timeouts, and sandbox settings. |
| Provider implementations | `internal/ai/pinecone/`, `openai/`, `xai/`, `mock/` | Provides the available AI backends, including Pinecone Assistant, OpenAI, xAI, and Mock. Pinecone Assistant is currently the main grounded knowledge-base candidate, but Sprint 2 must confirm and configure the final provider. Mock should be retained for local development and failure-path verification. |
| Tests | Existing `*_test.go` files | Preserve existing contracts and add representative chatbot question tests. |
| Documentation | `docs/` | Record setup instructions, architecture, provider decisions, limitations, and test procedures. |


## Team Branch-Based Development Workflow

The team will use `feature/team-b-travel-services-integration` to bring our work together without pushing directly to the client's `main` branch.

Create the team integration branch from the agreed version of the client's `main`. Each team member then creates their working branch from the team integration branch.

```text
feature/team-b-travel-services-integration
├── feature/hotel-provider
├── feature/ai-travel-flow
├── feature/booking-ui
└── test/travel-integration
```

This diagram shows where the working branches start; Git branches are not nested folders.

### Review and merge flow

```text
Individual branch
   ↓
Push changes to that branch
   ↓
Open a PR targeting feature/team-b-travel-services-integration
   ↓
Team review and checks
   ↓
Merge into feature/team-b-travel-services-integration
   ↓
Team tests everything together
   ↓
Open a PR from the team integration branch to the client's main
   ↓
Client / maintainer review and approval before merging
```

- Keep each working branch focused on its assigned task.
- Bring updates from the team integration branch into working branches regularly and resolve conflicts before merging.
- Check the PR's target branch before submitting it.
- Keep the team integration branch up to date with relevant changes from the client's `main`, and rerun affected tests after integrating those changes.
- Include test results and any known limitations in the final PR to the client.

## Required Files and Components for Planned Development

| File/Component | Purpose | Status |
| --- | --- | --- |
| Local copy of the YAF repository | Allows the team to run, inspect, test, and modify the application. | Available. |
| Team integration and individual working branches | Separates Sprint 2 work from the shared `main` branch. | To be confirmed. |
| Local `.env` based on `.env.example` | Stores local settings and credentials without committing secrets. | Required. |
| Go environment and project dependencies | Supports building, running, and testing the backend. | Required. |
| PostgreSQL and existing SQL/schema files | Provides the database required by the backend. | Required. |
| AI provider configuration | Enables testing with Pinecone, OpenAI, xAI, or Mock. | Provider decision required. |
| Hotel or booking API documentation and sandbox access | Supports the required external provider integration. | Not yet provided. |
| Hotel or booking integration component | Connects YAF to the selected provider and normalizes results. | New development required. |
| Chatbot retrieval and response updates | Allows the chatbot to use relevant airport, airline, flight, equipment, hotel, or booking data. | New development required. |
| Test cases and representative questions | Evaluates expected answers, missing data, unsupported questions, and provider failures. | To be prepared. |
| Setup and testing notes | Helps the team reproduce the environment and record issues. | To be completed. |

## Development Setup Issues and Missing Dependencies

- The repository supports Pinecone, OpenAI, and xAI, but the current setup only uses the mock provider, which returns a fixed “This is a mock AI response.” Valid credentials for at least one real provider are therefore required before meaningful chatbot testing can be performed.
- The team still needs to confirm which AI provider will be used for Sprint 2 and whether retrieval will be handled internally by Pinecone Assistant or through a YAF-managed retrieval pipeline before finalising the AI architecture.
- The current Pinecone retriever is not fully implemented and cannot yet generate embeddings, query a Pinecone index, apply metadata filters, or return retrieved context. This is only a blocker if the team chooses YAF-managed retrieval.
- Airport, airline, equipment, and PostgreSQL data are not currently connected to the AI chat pipeline. 
- The original repository has no hotel or booking integration. Provider documentation and sandbox credentials are needed to develop the integration logic and a normalised data model.
- No accessible-transfer provider has been selected or integrated.
- It is not confirmed whether Airlabs satisfies the flight-data requirement or Flightradar24 is mandatory.
- Local airport seed data is limited for chatbot testing; for example, SYD may not exist.
- Provider quotas, costs, licensing, and data-usage restrictions require confirmation.
- Representative chatbot questions have not yet been agreed.

## Sprint 2 Codebase Readiness

The following preparation tasks have been completed on the branch:

`feature/team-b-travel-services-integration`

- Created and switched to the team feature branch to keep development separate from the shared `main` branch.
- Reviewed the repository areas relevant to Sprint 2 chatbot improvements.
- Reviewed the existing AI chat flow, including the handler, orchestrator, provider router, and provider implementations.
- Reviewed the airport, airline, equipment, database, authentication, configuration, and testing components.
- Installed and configured Docker Desktop for local development.
 ![PostgreSQL container running in Docker Desktop](images/docker-postgres.png)
- Started PostgreSQL using Docker and confirmed that it is available to the backend.
- Created the local `.env` configuration based on `.env.example`.
- Configured local development settings for authentication, airport data, and the Mock AI provider.
- Downloaded and verified the Go project dependencies.
- Started the YAF backend successfully using `go run ./cmd/api` and confirmed it is listening on port `8080`.
![Backend running successfully](images/backend-running.png)
*Figure 1: YAF backend running locally on port 8080.*
![YAF API Demo Explorer](images/api-demo-explorer.png)
*Figure 2: YAF API Demo Explorer running locally.*
![YAF Admin Dashboard](images/yaf-admin-dashboard.png)
*Figure 3: YAF admin dashboard running locally.*
- Confirmed that the application reports the expected local providers during startup.
- Checked the application health endpoint using `curl -i http://localhost:8080/health` and confirmed that it returned `HTTP/1.1 200 OK`.
![Health endpoint returning HTTP 200 OK](images/health-endpoint.png)
*Figure 4: Health endpoint returning HTTP 200 OK.*
- Verified that the existing chatbot endpoint accepts requests using the Mock provider.
- Recorded the remaining Sprint 2 dependencies, including real AI credentials, external hotel or booking access, retrieval integration, and client confirmation of provider requirements.

The local environment is ready for Sprint 2 development and testing with the Mock provider. Testing with real providers remains dependent on credentials, hotel or booking-provider access, retrieval integration, and confirmation of client requirements.
