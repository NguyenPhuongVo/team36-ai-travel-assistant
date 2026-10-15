# YAF AI Retrieval and Response Quality Improvements

## Purpose
This document records improvements made to the YAF chatbot for retrieval accuracy, response quality, fallback handling, and hotel-result presentation.

## Improvements

### Prompt D
Prompt D was integrated to reduce misleading accessibility claims and improve response reliability.

- The chatbot must use only supplied context.
- General facilities such as parking, toilets, lifts, and shuttle services must not automatically be described as accessible.
- Accessibility claims require explicit supporting evidence.
- Missing accessibility information must be described as unconfirmed.
- The chatbot must distinguish between confirmed, provider-claimed, and unknown statuses.
- Incomplete requests should receive a clarification question.

### Fallback Handling
Fallback behaviour was reviewed and improved.

Instead of returning `This is a mock AI response`, the Mock provider now returns request-specific messages for:

- Hotel searches
- Airport accessibility questions
- Flight questions
- General requests

When the Mock provider is used, hotel cards are hidden so users do not mistake fixture data for a successful AI response.

**Example request:**

```bash
curl -i -X POST http://localhost:8080/api/ai/chat \
  -H "Content-Type: application/json" \
  -d '{"content":"Find an accessible hotel near HKG airport"}'
```

**Recorded HTTP response:**

```http
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 13 Oct 2026 21:50:29 GMT
Content-Length: 286
```

```json
{
  "provider": "mock",
  "model": "mock-chat",
  "content": "I’m temporarily unable to complete the hotel search, so no hotel results are available right now. Please try again shortly or verify availability directly with the property.",
  "latencyMs": 0,
  "synthetic": false,
  "warnings": [],
  "hotels": []
}
```

### Reranking
A deterministic keyword-overlap reranker was implemented in `internal/ai/reranker.go` and enabled with a threshold of `0.2`.

The reranker:

- Compares terms in the user query with retrieved context chunks.
- Gives higher priority to chunks with more matching terms.
- Removes weak context matches when stronger matches are available.
- Sends the reordered context to Pinecone for response generation.

Accessibility evidence and hotel availability are checked separately by structured hotel filtering. Keyword overlap alone does not establish whether a feature is confirmed or a hotel is available.

The implementation is not an LLM token-logit reranker because the current Pinecone integration does not expose token probabilities.

#### Reranking Test
**Query:**

```text
Which available hotel near HKG airport has a confirmed wheelchair-accessible room?
```

The query was tested with reranking disabled and enabled. These reported results predate the availability-filtering update described below.

| Check | Without reranking | With reranking |
| --- | --- | --- |
| Selected hotel | Kowloon Step-Free Residence | Kowloon Step-Free Residence |
| Accessibility | Confirmed wheelchair-accessible room | Confirmed wheelchair-accessible room |
| Availability | Available in the fixture | Available in the fixture |
| Hotel cards | HKG-H001 and HKG-H002 | HKG-H001 and HKG-H002 |
| Response wording | "through this link" | "through the provided link" |
| Latency | 8,066 ms | 5,288 ms |
| Synthetic warning | Present | Present |

**Interpretation:**
- **Hotel recommendations:** The selected hotel and hotel-card order remained unchanged after reranking.
- **Context relevance:** The context comparison suggested that keyword-relevant hotel information was prioritised. However, the responses alone do not confirm an improvement in accuracy or reveal the internal context order. Additional evidence from context chunks and relevance scores would be needed.
- **Response latency:** Lower latency was observed, but this cannot be directly attributed to reranking based on a single comparison. Repeated testing is required to evaluate performance reliably.
- **Availability filtering:** Both responses included the unavailable hotel HKG-H001 in the hotel cards, despite only recommending HKG-H002 in the text.

### Hotel Filtering and Sorting
The following updates are recorded from the supplied implementation notes. Accessibility statuses and availability refer to the demonstration dataset.

#### Accessibility Evidence Filtering
The filter matches the requested feature and evidence status: `confirmed`, `provider_claimed`, or `unknown`. If no status is specified, it defaults to `confirmed`.

- `Which hotel near HKG airport has confirmed grab rails?` returns Hong Kong Harbour Access Hotel because its grab rails are confirmed in the fixture.
- `Which hotel near HKG airport has provider claimed grab rails?` can match a hotel with `provider_claimed` grab-rail evidence.

A match for `unknown` identifies missing evidence; it does not confirm that the feature is available.

#### Availability Filtering
Queries containing `available` or `availability` now exclude unavailable hotels from the structured `hotels[]` array.

For `Which available hotel near HKG airport has a confirmed wheelchair-accessible room?`, the reported updated result contains only Kowloon Step-Free Residence (HKG-H002). This later update removes HKG-H001 from the cards.

#### General Hotel Lists
When a query does not request availability, all matching hotels are returned and their availability is included in the response text.

For `Which hotels near HKG airport have confirmed accessible toilets?`, the reported result is:
- Hong Kong Harbour Access Hotel: unavailable.
- Kowloon Step-Free Residence: available.

This keeps the text consistent with the returned cards without presenting every match as available.

#### Nearest-Hotel Sorting
Queries containing `closest` or `nearest` sort matching hotels by `distanceKm`, from nearest to farthest.

#### Demonstration Data Labelling
Records from `yaf-local-fixture` are labelled as demonstration data and include a reminder to verify details with the property.

#### Before-and-After Evidence
**Query:**
```text
Which hotel near SYD airport is closest and has a confirmed step-free entrance?
```

**Captures:** 14 October 2026, 05:53 before; 10:06 after, using the GMT timestamps recorded in the HTTP Date headers.

##### Before the Changes

The response text was `Sydney Airport Accessibility Hotel`, but the first structured hotel record was Mascot Step-Free Hotel:

```json
{
  "name": "Mascot Step-Free Hotel",
  "distanceKm": 1.4,
  "accessibility": {
    "stepFreeEntrance": {
      "status": "provider_claimed"
    }
  },
  "offer": {
    "available": true
  }
}
```

This record did not meet the requested confirmed step-free requirement, and the first card did not match the named answer. JSON examples here show selected fields from the returned hotel objects.

The response returned all three hotels:
- Mascot Step-Free Hotel: 1.4 km, provider-claimed step-free entrance.
- Sydney Airport Accessibility Hotel: 2.4 km, confirmed step-free entrance.
- Harbour Inclusive Suites: 6.8 km, confirmed step-free entrance but unavailable.

##### After the Changes

The response text remained `Sydney Airport Accessibility Hotel`. The first structured hotel record now matched that answer:

```json
{
  "name": "Sydney Airport Accessibility Hotel",
  "distanceKm": 2.4,
  "accessibility": {
    "stepFreeEntrance": {
      "status": "confirmed"
    }
  },
  "offer": {
    "available": true
  }
}
```

The second returned record was:

```json
{
  "name": "Harbour Inclusive Suites",
  "distanceKm": 6.8,
  "accessibility": {
    "stepFreeEntrance": {
      "status": "confirmed"
    }
  },
  "offer": {
    "available": false
  }
}
```

Mascot Step-Free Hotel was excluded because its step-free entrance was `provider_claimed`, not `confirmed`.

##### Interpretation
The table shows corrected filtering and first-card consistency. Both lists were already ordered by distance, so it does not establish a separate sorting improvement.

Harbour Inclusive Suites remains in this captured array with `available: false`. This query asks for confirmed accessibility and distance, without explicitly requesting availability.

The captures show the final output, not the internal order of filtering and sorting.

## Limitations

- The project uses a local demonstration dataset because no live hotel API is available for this implementation.
- Prices, availability, booking links, images, and accessibility statuses should be treated as demonstration values.
- Pinecone response wording can vary between requests. 
- Keyword reranking may miss synonyms or differently worded requests.
- Flight information depends on the configured provider and retrieval time.
- Hotel-name-only queries may require additional hotel-name lookup support.
- Frontend verification is still required for card order, image loading, and fallback presentation.

## Verification

```bash
gofmt -w internal/ai/types.go
go test ./internal/ai ./internal/hotel ./cmd/api
```

**Recorded result:** The supplied output shows all three packages passing. These commands were not rerun as part of this documentation update.

Reported package results:

```text
internal/ai      ok
internal/hotel   ok
cmd/api         ok
```

The command above does not run the separate `internal/ai/mock` package. To verify the fallback tests in `internal/ai/mock/provider_test.go`, run:

```bash
go test -v ./internal/ai/mock
```
Record whether hotel, flight, airport, and general requests receive the correct messages, including when the surrounding prompt contains unrelated keywords. The supplied HTTP captures demonstrate the hotel fallback only. Pinecone-to-Mock failover also requires logs showing the Pinecone failure followed by Mock selection.
