# Hotel Response Matching and Structured Chatbot Response

## Purpose
This document records the implementation and testing of the structured chatbot response for Sprint 2. It explains how hotel data is matched and returned, and documents the response fields for SwiftUI frontend integration.

## Implementation Summary
The backend now:
- Returns a consistent `ChatResponse` structure.
- Combines airport data from YAF with synthetic hotel data during retrieval.
- Returns structured hotel objects for frontend cards.
- Marks synthetic hotel data with synthetic and warnings.
- Distinguishes confirmed, provider-claimed, and unknown accessibility data.
- Returns concise responses for “which hotel” queries.
- Reports unavailable matching hotels instead of presenting them as bookable.
- Falls back from Pinecone to Mock when the AI provider fails.

## Response Contract and Frontend Handover

### Successful Response Structure
This illustrative example shows the response fields, not a captured test response:
```json
{
  "provider": "...",
  "model": "...",
  "content": "...",
  "latencyMs": 0,
  "synthetic": false,
  "warnings": [],
  "hotels": []
}
```
Responses containing synthetic hotel data identify it through synthetic and the relevant warnings.

### Hotel Card Information

The `hotels` field contains:
- Hotel name, airport, and location.
- Distance from the airport.
- Accessibility features and evidence status.
- Images and alt text.
- Ratings.
- Price and availability.
- Booking URL and source URL.

### Frontend Field Mapping

| Response field | Intended use in SwiftUI |
| --- | --- |
| `content` | Assistant chat message |
| `hotels` | Hotel cards |
| `warnings` | Prototype or limitation notices |
| `synthetic` | Identifies synthetic data |
| `provider`, `model`, `latencyMs` | Internal diagnostics |

This mapping documents the frontend contract; it does not by itself confirm that each field has been rendered and tested in SwiftUI.

## Matching Logic

- `requestedFeatureLabel`: Converts the user's wording into a readable accessibility label.
- `hotelMatchesQuery`: For a recognised feature, checks whether the hotel's corresponding accessibility field has a `confirmed` evidence status.

Both functions convert the query to lowercase before checking for phrases, so matching is case-insensitive.

Supported accessibility labels:

- `step-free` or `step free` → `step-free entrance`
- `accessible toilet` → `accessible toilet`
- `wheelchair-accessible room` or `wheelchair accessible room` → `wheelchair-accessible room`
- `roll-in shower` or `roll in shower` → `roll-in shower`
- `airport shuttle` or `accessible shuttle` → `accessible airport shuttle`
- `accessible parking` or `parking lot` → `accessible parking`
- No recognised phrase → `the requested accessibility feature`

For a recognised feature, `provider_claimed` or `unknown` evidence does not pass the matching check. If no recognised phrase is found, `hotelMatchesQuery` returns `true`, so this helper does not filter the hotel by accessibility. This does not confirm the hotel's accessibility or booking availability.

Each function returns on its first matching condition. Queries mentioning multiple features are therefore not checked against every feature. The check order also differs: the label function checks step-free access first, while the matching function checks accessible toilets and wheelchair-accessible rooms before step-free access. A query containing several features can therefore produce a label for a different feature from the one checked.

## Response Behaviour
- A matching hotel is available: Return the hotel name.
- A matching hotel is unavailable: Explain that the hotel matches the requested feature but is unavailable.
- No matching hotel exists: Return `No available matching hotel was found.`

Full hotel data remains in the structured `hotels` field for frontend cards.

### Example
**User query:**

> Which hotel near HKG airport has a step-free entrance?

**Chatbot response:**

> Hong Kong Harbour Access Hotel has a confirmed step-free entrance, but it is currently unavailable for booking.

This example uses fictional hotel data. “Confirmed” describes the evidence status in the test fixture, not independently verified accessibility information.

## Representative Query Tests

The table records chatbot test questions, expected behaviour, actual responses, and results. The questions and expected answers still need client review.
- **Tested by / date:** To record.
- **AI provider:** Pinecone Assistant
- **Fallback provider:** Mock
- **Hotel data:** Local synthetic fixture
- **Airport test data:** FNC
- **Hotel test codes:** SYD, FNC, RMO, HKG

### Running and Recording Tests
Use the Git Bash command below for each question in the table. For Q11, send {} as the request body. For Q13, make Pinecone unavailable and check that the chatbot switches to Mock.

```bash
curl -i -X POST http://localhost:8080/api/ai/chat \
  -H "Content-Type: application/json" \
  -d '{"content":"What accessibility services are available at FNC airport?"}'
```

### Results

| ID | Query or scenario | Expected behaviour | Recorded result / evidence | Status |
| --- | --- | --- | --- | --- |
| Q01 | What accessibility services are available at FNC airport? | List services supported by the FNC source data; explain missing or unverified information. | Returned ambulift, reduced-mobility assistance, immigration and customs assistance, lifts, accessible toilets, parking, and transport options. Pinecone, 5019 ms; expected response fields present. HTTP status not captured. | Partial — services returned; source verification pending |
| Q02 | Does FNC airport have accessible toilets? | Answer using FNC source evidence; explain when information is missing. | Returned “Yes, Madeira Airport (FNC) has accessible toilets.” via Pinecone in 5576 ms. Expected fields present; no source evidence included in the response. | Partial — answer returned; source verification pending |
| Q04 | Find an accessible hotel | Ask for missing location and accessibility needs, and wait for the user's response. | Asked for an airport code (SYD, MEL, RMO, FNC, or HKG). Pinecone, 3479 ms; `hotels: []`. Follow-up clarification of accessibility requirements not recorded. | Partial |
| Q05 | Which hotel near SYD airport has a confirmed accessible toilet? | Return the available matching hotel based on confirmed fixture evidence and identify synthetic data. | Returned Mascot Step-Free Hotel, whose fixture toilet status is `confirmed` and availability is `true`. Pinecone, 2447 ms; `synthetic: true` with a warning. All three SYD hotels remain in `hotels`. | Pass — answer matching and synthetic-data marking |
| Q06 | Find hotels near FNC airport with step-free access. | Return matching hotels using fixture evidence, with availability and synthetic-data limitations. | Returned Madeira Step-Free Guesthouse with `stepFreeEntrance.status: confirmed` and `offer.available: true`. Pinecone, 2813 ms; `synthetic: true` with a warning. | Pass — fixture match and structured availability |
| Q07 | Which available hotel near SYD has an accessible airport shuttle? | Return an available hotel with confirmed shuttle evidence, or explain that no match is supported. | HTTP 200. Returned Mascot Step-Free Hotel; its shuttle evidence is `confirmed` and availability is `true`. Pinecone, 4482 ms; synthetic flag and warning present. All three SYD hotels remain in `hotels`. | Pass - answer matching; frontend cards not verified |
| Q08 | Which hotel near HKG airport has a wheelchair-accessible room? | Check room accessibility evidence and availability; do not treat unknown or provider-claimed data as confirmed. | HTTP 200. Reported Hong Kong Harbour Access Hotel has a confirmed wheelchair-accessible room but is unavailable. Fixture room status is `confirmed`; availability is `false`. Pinecone, 4115 ms; synthetic flag and warning present. | Pass |
| Q09 | Which hotel near HKG airport has a step-free entrance? | Report the matching feature and explain that the matching hotel is unavailable. | HTTP 200. Reported Hong Kong Harbour Access Hotel has a confirmed step-free entrance but is unavailable. Fixture entrance status is `confirmed`; availability is `false`. Pinecone, 3632 ms; synthetic flag and warning present. | Pass |
| Q10 | Which hotel near RMO airport has confirmed accessible parking? | Return an available hotel with confirmed parking evidence; explain when no confirmed available match exists. | HTTP 200. Returned Chisinau Accessible City Hotel; parking evidence is `confirmed` and availability is `true`. Pinecone, 4252 ms; synthetic flag and warning present. | Pass |
| Q11 | Empty request `{}` | Return HTTP 400 with a validation error. | HTTP 400 Bad Request. Returned `{"error":"content required"}`. Confirmed in the supplied HTTP capture. | Pass |
| Q12 | Is Harbour Inclusive Suites hotel near SYD airport available? | Identify the hotel and use fixture availability if supported; otherwise clarify or explain the limitation. | HTTP 200. Returned "No, Harbour Inclusive Suites is not available." The named hotel (`SYD-H003`) has `offer.available: false`. Pinecone, 3164 ms; synthetic flag and warning present. All three SYD hotels remain in `hotels`. | Pass - named-hotel availability; frontend cards not verified |
| Q13 | Pinecone unavailable; query: What services are available at FNC airport? | Fall back to Mock after a Pinecone failure. | Returned `provider: mock`, `model: mock-chat`, and “This is a mock AI response.” Latency: 0 ms; expected response fields present. HTTP status and preceding Pinecone failure not captured. | Partial — Mock response confirmed; fallback trigger not verified |

### Q01 Recorded Response
```json
{
  "provider": "pinecone",
  "model": "gpt-4o-2024-11-20",
  "content": "The following accessibility services are available at Madeira Airport (FNC):\n\n- Ambulift assistance\n- Barrier-free assistance for passengers with reduced mobility\n- Immigration and customs assistance\n- Lifts between floors\n- Accessible toilets\n- Accessible parking\n- Accessible transport options, including buses, taxis, or wheelchair-adapted transfers.",
  "latencyMs": 5019,
  "synthetic": false,
  "warnings": [],
  "hotels": []
}
```

### Q02 Recorded Response
```json
{
  "provider": "pinecone",
  "model": "gpt-4o-2024-11-20",
  "content": "Yes, Madeira Airport (FNC) has accessible toilets.",
  "latencyMs": 5576,
  "synthetic": false,
  "warnings": [],
  "hotels": []
}
```

### Q04 Recorded Response
```json
{
  "provider": "pinecone",
  "model": "gpt-4o-2024-11-20",
  "content": "Please provide the airport code (SYD, MEL, RMO, FNC, or HKG) to narrow down the search for accessible hotels.",
  "latencyMs": 3479,
  "synthetic": false,
  "warnings": [],
  "hotels": []
}
```

### Q05 Response Content
> Mascot Step-Free Hotel
Shows: The chatbot selected an available hotel with confirmed accessible-toilet evidence in the synthetic data.

### Q06 Response Content
> Madeira Step-Free Guesthouse
Shows: The returned hotel has confirmed step-free entrance evidence and is marked available in the synthetic data.

### Q07 Response Content
> Mascot Step-Free Hotel
Shows: The chatbot selected an available hotel with a confirmed accessible airport shuttle in the synthetic data.

### Q08 Response Content
> Hong Kong Harbour Access Hotel has a confirmed wheelchair-accessible room, but it is currently unavailable for booking.
Shows: The chatbot recognised the confirmed room feature while clearly stating that the hotel is unavailable in the synthetic data.

### Q09 Response Content
> Hong Kong Harbour Access Hotel has a confirmed step-free entrance, but it is currently unavailable for booking.
Shows: The chatbot distinguished confirmed step-free access from booking availability in the synthetic data.

### Q10 Response Content
> Chisinau Accessible City Hotel
Shows: The chatbot selected an available RMO hotel with confirmed accessible-parking evidence in the synthetic data.

### Q11 Recorded Response
```json
{"error":"content required"}
```
Shows: The backend rejected the empty request with HTTP 400 Bad Request because `content` was missing.

### Q12 Response Content
> No, Harbour Inclusive Suites is not available.
Shows: The answer matches the named hotel's `available: false` value in the synthetic data.

### Q13 Recorded Response
```json
{
  "provider": "mock",
  "model": "mock-chat",
  "content": "This is a mock AI response.",
  "latencyMs": 0,
  "synthetic": false,
  "warnings": [],
  "hotels": []
}
```

## Remaining Follow-up
- Q13: Confirm Pinecone was tried first and failed before Mock took over. Save the logs and HTTP status.
- Record any missing test results and check airport information against the YAF materials.
- Q01: Check the returned services against the YAF source material and capture the HTTP status using `curl -i`.
- Q02: Check whether the YAF source confirms accessible toilets at FNC.
- Q05, Q07, and Q12: Save the original JSON responses and HTTP statuses. Check that hotel cards clearly show which hotels match, which have unconfirmed features, and which are unavailable.
- Q04: Reply to the chatbot’s question and check that it uses the user’s location and accessibility needs.
- Review the test questions and expected answers with the client.
- Share the response format with UX/frontend and check that messages, hotel cards, and synthetic-data warnings display correctly.

## Limitations
The team is still waiting for the client to provide hotel API access. Current hotel records, including prices, availability, images, booking links, and accessibility details, are synthetic data for testing only. They must not be used for real bookings or travel decisions.