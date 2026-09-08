# Your Accessible Flight Technical Review – Sprint 1 Week 2

## Proposed Requirements Review

The project aims to extend the existing Your Accessible Flight (YAF) AI assistant so that users can ask accessibility-related travel questions and receive relevant information from client provided data sources. The main requirement is to integrate at least one hotel or booking provider into the existing AI chatbot workflow, allowing users to access accommodation information, accessibility details, and relevant links where available.

## Initial User Flow

1. The user enters an accessibility-related travel query into YAF AI Assistant.
2. The query is sent to the YAF backend for processing.
3. The backend retrieves relevant information from the integrated client-provided hotel or booking data source.
4. The AI uses the retrieved information to generate a response.
5. The user receives the response together with relevant accommodation details and links where available.

## Technical Constraints

- Limited team familiarity with Go may increase development time when extending the existing YAF backend with new hotel/booking functionality.

- The current AI workflow does not include a tool/function-calling layer for travel services. Integrating hotel, booking, transfer, or other live travel APIs may therefore require additional backend logic.

- No server-side conversation history is currently stored, so each chat request is treated independently.

- Pinecone Assistant is designed for document-grounded responses rather than live transactional data. Live hotel availability, prices, bookings, or transfer information will need to come from other approved data sources.

- The current ChatResponse returns final text only and does not expose structured hotel results, booking links, or citations. Supporting these features may require changes to the API response model and frontend.

- The current database schema does not include hotel, accommodation, booking, transfer, price, or availability entities. The implementation therefore depends on whether these data are retrieved dynamically from client-provided APIs or represented through additional local data structures.

- Database schema changes are currently applied through manual SQL scripts because there is no migration runner.

## External Dependencies

- Client provided hotel or booking data source/API.
- Existing AI providers, including Pinecone and the mock provider used by the YAF backend.
- PostgreSQL database used by the existing backend.
- API credentials, sandbox access, or permissions required for client-provided external services.
- Existing YAF iOS application and its current API/chat integration

## Major Technical Risks

- The existing iOS application may require changes to support the new backend-based hotel or booking integration. Differences between the current Swift chat implementation and backend AI workflow increase integration and testing effort.

- Required API credentials or access to the client-provided hotel or booking service may not be available when development begins, which could delay integration and testing.

- The AI may generate unsupported or incorrect accessibility information if the external data source does not provide enough detail.

- External API calls combined with AI processing may increase response latency or cause incomplete responses when a provider is unavailable.

- Change to the current text only ChatResponse may require corresponding changes to existing iOS frontend and increase implementation effort.
