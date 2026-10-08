# YAF Chatbot Prompt Evaluation

## Purpose

This document evaluates prompt variations used by the YAF chatbot. The aim is to identify the prompt that provides the most reliable retrieval and response behaviour for hotel, airport, and flight questions.

The retriever supplied the relevant context, while the prompt guided the generated response. Retrieval availability and prompt quality were therefore considered separately during evaluation.

The prompts are compared using representative queries and the following
criteria:
- Accurate use of supplied context
- Response relevance
- Accessibility evidence handling
- Hotel availability handling
- Answer length and clarity
- Safe handling of missing or unavailable information

The evaluation focuses on improving Pinecone-generated responses. It does not replace the existing hotel and flight retrieval logic.

## Test environment
- Endpoint: `POST /api/ai/chat`
- Primary AI provider: Pinecone Assistant
- Fallback AI provider: Mock
- Hotel provider: local synthetic fixture
- Flight provider: AirLabs or configured airport provider
- Hotel fixture: `internal/hotel/data/hotels.json`
- Prompts tested: A, B, C, and D

## Prompt versions

### Prompt A — Previous baseline
```text
You are YAF, an accessibility-focused travel assistant.
Use the supplied context when relevant. If the context does not contain the
answer, say so rather than guessing.

Accessibility statuses mean:
- confirmed: verified by a reliable source
- provider_claimed: reported by the provider but not independently verified
- unknown: no information is available

For “which hotel” questions, return only the matching available hotel name or
names, one per line. Do not include extra details unless requested.
```

Prompt A provides general safety rules and concise hotel-name responses.

### Prompt B — Concise and strict
```text
You are YAF, an accessibility-focused travel assistant.

Use only the supplied context. Never guess or invent information.

For hotel questions:
- match the requested airport and accessibility feature;
- treat only confirmed features as confirmed;
- distinguish confirmed, provider_claimed, and unknown;
- mention availability when relevant;
- answer briefly.

For airport and flight questions, use only the supplied airport or flight data.
If information is missing, say so clearly.
```

Prompt B focuses on strict evidence matching, availability, and short answers.

### Prompt C — Guided clarification
```text
You are YAF, an accessibility-focused travel assistant.

Use only the supplied context. Never guess.

When a hotel request is missing important information, ask one clear follow-up
question instead of recommending a hotel immediately.

Important information may include:
- location or airport;
- accessibility features;
- dates or availability requirements.

Once enough information is available, search the supplied hotel context and
answer concisely.
```

Prompt C focuses on asking follow-up questions when a hotel request is
incomplete. It does not provide as much detail about airport and flight output
as Prompts A and B.

### Prompt D — Guided and evidence-aware
```text
You are YAF, an accessibility-focused travel assistant.

Use only the supplied context. Never guess or invent information.

If the request is incomplete, ask one clear follow-up question. Missing
information may include the airport or location, accessibility feature, dates,
or availability requirements. Let the user answer in their own words.

Accessibility statuses mean:
- confirmed: verified in the supplied source
- provider_claimed: reported by the provider but not independently verified
- unknown: no information is available

For hotel questions, match the requested airport and accessibility feature.
Treat only confirmed features as confirmed matches. Mention availability when
relevant and keep the answer concise. For a general hotel search, list only the
hotel name, requested feature status, and availability. Do not include
addresses, prices, booking links, ratings, descriptions, or unrelated features
unless requested.

For airport and flight questions, use only the supplied airport or flight
context. If the context does not contain the answer, say so clearly.
```

Prompt D combines Prompt C's clarification behaviour with Prompt A and B's
evidence, availability, and context-use rules.


## Test results
The same queries were used when comparing prompt versions:
Ratings assess the supplied responses; shared findings are summarised under Observations.

| Query | Prompt | Accuracy | Relevance | Length | Safety | Actual response / evidence |
|---|---|---|---|---|---|---|
| `Sydney airport?` | A | Partial | Pass | Pass | Pass | "The provided context does not include information about Sydney Airport's accessibility services. You may need to contact Sydney Airport directly for accurate and up-to-date details."<br>Correctly reported that specific information was missing, but did not ask a focused follow-up question. |
| `Sydney airport?` | B | Partial | Pass | Pass | Pass | "The provided context does not include information about Sydney Airport's accessibility services. You should contact Sydney Airport directly to confirm their accessibility features."<br>Correctly avoided guessing and advised checking the airport. |
| `Sydney airport?` | C | Pass | Pass | Pass | Pass | "Could you clarify what specific information you need regarding Sydney Airport? For example, are you looking for accessibility services at the airport or accessible hotels near Sydney Airport?" `provider: "pinecone"` `synthetic: false` `warnings: []` `hotels: []`<br>Asked whether the user wanted airport services or nearby hotels. |
| `Sydney airport?` | D | Partial | Pass | Pass | Pass | HTTP 200. "The provided context does not include any information about Sydney Airport's accessibility features or services. I recommend contacting Sydney Airport directly to confirm their accessibility options. If you have specific questions or need assistance with a particular service, please let me know."<br>`provider: pinecone`; `model: gpt-4o-2024-11-20`; `latencyMs: 5018`; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Explained the source limitation and referred the user to the airport, but did not ask a focused clarification question. Captured 7 October 2026, 21:35:01 GMT, on `feature/team-b-ai-retrieval-prototype`. |
| `airport?` | A | Pass | Pass | Pass | Pass | "Please specify which airport you are asking about or provide more details so I can assist you better."<br>Asked the user to specify an airport. |
| `airport?` | B | Pass | Pass | Pass | Pass | "Please specify the airport you are asking about so I can provide relevant accessibility information."<br>Asked which airport was intended. |
| `airport?` | C | Pass | Pass | Pass | Pass | "Could you clarify which airport or location you are referring to?" `provider: "pinecone"` `synthetic: false` `warnings: []` `hotels: []`<br>Asked for an airport or location. |
| `airport?` | D | Pass | Pass | Pass | Pass | "Could you clarify which airport you are asking about and what specific information or accessibility feature you need? For example, are you looking for accessibility services, transport options, or something else?"<br>Asked for the airport and the required information or feature. |
| `hotel?` | A | Pass — clarification | Pass | Pass | Pass | HTTP 200. "Please provide a supported airport code (SYD, RMO, FNC, HKG) to search for hotels. If you have specific accessibility requirements or other preferences, feel free to include them."<br>`provider: pinecone`; `model: gpt-4o-2024-11-20`; `latencyMs: 5390`; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Asked for an airport code and invited accessibility requirements. |
| `hotel` | B | Pass — clarification | Pass | Pass | Pass | HTTP 200. "Please provide the airport code and any specific accessibility features or requirements you are looking for in a hotel. Supported airport codes are SYD, MEL, RMO, FNC, HKG, SIN, and NRT."<br>`provider: pinecone`; `model: gpt-4o-2024-11-20`; `latencyMs: 4448`; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Asked for the airport code and accessibility requirements. |
| `hotel` | C | Pass — clarification | Pass | Pass | Pass | HTTP 200. "Could you please provide the location or airport code, desired accessibility features, and travel dates or availability requirements? This information will help me find a suitable hotel for you."<br>`provider: pinecone`; `model: gpt-4o-2024-11-20`; `latencyMs: 4608`; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Asked for location, accessibility features, and dates or availability requirements. |
| `hotel` | D | Pass | Pass | Pass | Pass | "Could you please specify the airport code, accessibility features you need, and any other requirements such as dates or availability?"<br>Asked for airport, accessibility features, dates, and availability requirements. |
| Whitespace-only `{"content":" "}` | A | Partial | Partial | Pass | Pass | HTTP 200. "Hello! How can I assist you with your accessibility-focused travel needs today?"<br>Pinecone; 2624 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Returned a general greeting without a focused clarification. |
| Whitespace-only `{"content":" "}` | B | Partial | Partial | Pass | Pass | HTTP 200. "Hello! How can I assist you with your accessibility-focused travel needs today?"<br>Pinecone; 3449 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Returned a general greeting without a focused clarification. |
| Whitespace-only `{"content":" "}` | C | Partial | Partial | Pass | Pass | HTTP 200. "Could you please provide more details about your hotel request? Specifically, let me know the location or airport, any required accessibility features, and your travel dates or availability needs."<br>Pinecone; 3972 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Asked for details but assumed hotel intent from a blank request. |
| Whitespace-only `{"content":" "}` | D | Pass - clarification | Pass | Partial | Pass | HTTP 200. "Your request is incomplete. Could you please provide more details? For example:<br>- Are you asking about an airport, flight, or hotel?<br>- If it's a hotel, which airport or location should it be near?<br>- What specific accessibility features or requirements are you looking for?<br>- Do you have specific dates or availability needs?<br><br>Let me know so I can assist you better!"<br>Pinecone; 6233 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Recognised missing intent, but asked several questions instead of one concise follow-up. |
| Does FNC airport provide accessible parking? | A | Partial | Pass | Pass | Partial | HTTP 200. "Yes, Madeira Airport (FNC) provides accessible parking."<br>Pinecone; 3384 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Overstates parking accessibility; see Observations. |
| Does FNC airport provide accessible parking? | B | Partial | Pass | Pass | Partial | HTTP 200. "Yes, Madeira Airport (FNC) provides accessible parking."<br>Pinecone; 3689 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Overstates parking accessibility; see Observations. |
| Does FNC airport provide accessible parking? | C | Partial | Pass | Pass | Partial | HTTP 200. "Yes, Madeira Airport (FNC) provides accessible parking."<br>Pinecone; full answer confirmed by the tester. Latency and remaining response fields were not supplied.<br>Overstates parking accessibility; see Observations. |
| Does FNC airport provide accessible parking? | D | Partial | Pass | Pass | Partial | HTTP 200. "Yes, Madeira Airport (FNC) provides accessible parking."<br>Pinecone; 2844 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Overstates parking accessibility; see Observations. |
| Which hotel near MEL airport has a wheelchair-accessible room? | A | Pass | Pass | Pass | Pass | HTTP 200. "No available matching hotel was found."<br>Pinecone; 2746 ms; `synthetic: true`; warning: Hotel results are synthetic prototype data.<br>Room accessibility is `unknown` and the hotel is unavailable. |
| Which hotel near MEL airport has a wheelchair-accessible room? | B | Pass | Pass | Pass | Pass | HTTP 200. "No available matching hotel was found."<br>Pinecone; 3640 ms; `synthetic: true`; warning: Hotel results are synthetic prototype data.<br>Room accessibility is `unknown` and the hotel is unavailable. |
| Which hotel near MEL airport has a wheelchair-accessible room? | C | Pass | Pass | Pass | Pass | HTTP 200. "No available matching hotel was found."<br>Pinecone; 4340 ms; `synthetic: true`; warning: Hotel results are synthetic prototype data.<br>Room accessibility is `unknown` and the hotel is unavailable. |
| Which hotel near MEL airport has a wheelchair-accessible room? | D | Pass | Pass | Pass | Pass | HTTP 200. "No available matching hotel was found."<br>Pinecone; 3088 ms; `synthetic: true`; warning: Hotel results are synthetic prototype data.<br>Room accessibility is `unknown` and the hotel is unavailable. |
| What flights are departing from SYD? | A | Pending source check | Pass | Pass | Pending source check | HTTP 200. "The following flights are departing from SYD:" Three flights: JQ912 (Jetstar Airways) to TSV, QF5912 (Qantas) to TSV, and QF5719 (Qantas) to HBA. All departure times shown as 9 October 2026, 06:45 UTC.<br>Pinecone; 5321 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>These three flights are absent from the later source capture; the original retrieval context is needed to verify this earlier response. |
| What flights are departing from SYD? | B | Pass | Partial | Partial | Pass | HTTP 200. "The following flights have departed from SYD:" ZL6852 (Regional Express) to DBO, SQ6808 (Singapore Airlines) and VA808 (Virgin Australia) to MEL, NZ1704 (Air New Zealand) and CX9059 (Cathay Pacific) to BNE. All marked landed. Departure times: 9 October 2026, 06:55 UTC for ZL6852 and 07:00 UTC for the others; arrival times also included.<br>Pinecone; 5398 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Flight identifiers, airlines, destinations, statuses, and displayed time values match the later backend source capture. The added UTC label is not supported by its timezone-free timestamps. |
| What flights are departing from SYD? | C | Pass | Partial | Pass | Pass | HTTP 200. "The following flights have departed from SYD:" ZL6852 (Regional Express) to DBO, SQ6808 (Singapore Airlines) and VA808 (Virgin Australia) to MEL, NZ1704 (Air New Zealand) and CX9059 (Cathay Pacific) to BNE. All marked landed. Departure times: 9 October 2026, 06:55 UTC for ZL6852 and 07:00 UTC for the others.<br>Pinecone; 3787 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Flight identifiers, airlines, destinations, statuses, and displayed time values match the later backend source capture. The added UTC label is not supported by its timezone-free timestamps. |
| What flights are departing from SYD? | D | Pass | Partial | Partial | Pass | HTTP 200. "The following flights have departed from SYD:" ZL6852 (Regional Express) to DBO, SQ6808 (Singapore Airlines) and VA808 (Virgin Australia) to MEL, EK5504 (Emirates) and NZ1704 (Air New Zealand) to BNE. All marked landed. Departure times: 9 October 2026, 06:55 UTC for ZL6852 and 07:00 UTC for the others; arrival times also included. Used bold flight headings and multiple lines per flight.<br>Pinecone; 5232 ms; `synthetic: false`; `warnings: []`; `hotels: []`.<br>Flight identifiers, airlines, destinations, statuses, and displayed time values match the later backend source capture. The added UTC label is not supported by its timezone-free timestamps. |


## Observations

### Flight responses

- B, C, and D matched the later AirLabs-backed data, but listed landed flights rather than upcoming departures. A remains unverified; its flights are absent from the later capture, and live data can change between tests.
- The chatbot added UTC to times that had no timezone label. Timezone handling needs checking.

### Clarification

- Blank input returned HTTP 200: A and B greeted the user, C assumed a hotel request, and D asked several questions. Rejecting blank input requires backend validation.
- All prompts asked for missing hotel details, but their supported airport lists differed.
- D's clarification was inconsistent: too many questions for blank input, but no focused question for "Sydney airport?"

### Accessibility and availability

- All prompts overstated FNC parking accessibility. The source confirms parking, not accessible spaces; the answer should explain this and refer the user to the airport.
- All prompts correctly reported no available MEL hotel match. The fixture's room accessibility is unknown and the hotel is unavailable, so this test does not show which prompt is better. The hotel still appears in the returned cards data.

## Recommendation

Prompt D is the current best-performing candidate overall because it provides the strongest clarification, evidence-handling, and availability guidance. However, its flight response length and inconsistent clarification behaviour should be improved and retested.

Further improvements:
- Ask one short clarification question at a time.
- Do not describe a facility as accessible unless the supplied source explicitly confirms it.
- Retest these changes using the same queries.

The router timeout was increased from 7 to 10 seconds as a separate reliability adjustment. This does not guarantee a successful Pinecone response. Mock fallback responses still need improvement.

## Limitations
- Hotel records are synthetic prototype data.
- Prices, availability, booking links, and accessibility information must be confirmed with a real provider before travel.
- Pinecone response wording can vary between requests.
- AirLabs flight coverage and availability can vary by airport and time.
- The prompt evaluation does not replace automated regression tests.

## Verification commands

```bash
gofmt -w internal/ai/orchestrator.go
go test ./internal/ai ./internal/hotel ./internal/airport ./cmd/api
```
