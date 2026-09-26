# Hotel/Booking Provider Preparation

## Task
Investigate and prepare a hotel or booking data provider for chatbot integration.

## Current Provider Status
The client has not yet confirmed the hotel or booking provider.

A provider-neutral hotel interface, normalized hotel model,
local mock provider, and synthetic hotel fixture have been prepared. The
fixture contains fictional records for development, UI demonstration, and
automated testing only. It is not live booking data and must not be used for
production bookings.

## Current Synthetic Hotel Data
The local hotel fixture is stored in:
`internal/hotel/data/hotels.json`

It contains fictional records for:
- SYD — Sydney
- FNC — Madeira
- RMO — Chisinau
- HKG — Hong Kong

### Record Fields
Each record includes:
- Provider and provider ID
- Airport IATA code
- Hotel name and fictional address
- Distance from the airport
- Description
- Synthetic image URL and alt text
- Synthetic rating and review count
- Accessibility features
- Accessibility evidence status
- Price and currency
- Availability
- Booking URL
- Source URL
- Retrieval timestamp

All records use:
```text
provider: yaf-local-fixture
```

### Accessibility Fields
Accessibility fields include:
- Step-free entrance
- Wheelchair-accessible room
- Roll-in shower
- Accessible toilet
- Grab rails
- Emergency call cord
- Accessible parking
- Accessible airport shuttle

## Backend Integration Structure
The normalized hotel types are defined in:
`internal/hotel/model.go`

The provider-neutral interface is defined in:
`internal/hotel/provider.go`
```go
type Provider interface {
    Search(ctx context.Context, airportIATA string, limit int) ([]Hotel, error)
}
```

## Local Integration and Testing
The local mock provider is wired into the AI retrieval flow when `HOTEL_PROVIDER=mock` is configured. Hotel data can be loaded, filtered by airport code, limited by result count, and returned as retrieval context.

Tests performed:

```text
go test ./internal/hotel
go test ./config
```

### Hotel Mock Provider Tests

![Hotel mock provider and retriever tests passing](../images/mock-test.png)
*Figure 1: Detailed hotel test results covering hotel questions, unrelated questions, airport and result-limit filtering, unknown airports, and cancelled requests.*

### Local Backend Running

![YAF backend running with mock airport, AI, and hotel providers](../images/hotel-mock-backend-running.png)

*Figure 2: The backend running on port 8080 with mock providers and development authentication. The request log shows an HTTP 200 response.*

### Related Package Tests

![Passing hotel, configuration, and AI package tests](../images/hotel-config-ai-tests.png)
*Figure 3: Hotel and configuration tests passing with cached results, alongside passing AI package tests.*

## External Provider Blockers
The following items are blocking real provider integration:
- The client has not confirmed the selected hotel or booking provider.
- No official API documentation or sandbox endpoint has been supplied.
- API credentials and authentication details are unavailable.
- Booking and availability request formats are unknown.
- Provider accessibility fields and evidence standards are not confirmed.
- A real connectivity test cannot be completed until sandbox access is provided.

## Current Mock Limitations
- The mock provider searches primarily by airport code.
- City name matching is not yet complete.
- Dates, guest counts, room selection, and live availability are not supported.
- The fixture timestamp is not a live provider retrieval timestamp.
- The booking URLs do not perform real bookings.
- The backend retrieves hotel context, but the mock AI returns a fixed response and does not include that context in its answer.
