# Team36 Accessible Accommodation Chat App: Project Documentation

A SwiftUI iOS app, "Chat with The Expert", that helps people with accessibility needs find suitable accommodation through a conversational interface with voice input.

## 1. Overview

### 1.1 Purpose

The app lets a user describe their accessibility requirements (for example wheelchair use, step-free access, accessible bathroom, hearing or vision impairment), their destination and length of stay. An "expert" chatbot replies in a chat thread and eventually presents matching hotels as tappable tiles. Each hotel opens into a detailed card and then a full description screen with benefits, location and a Google Maps link.

### 1.2 Key features

- Chat interface with expert bubbles, user bubbles, quick-reply chips and a typing indicator.
- Two interchangeable chat back ends: a local scripted keyword bot (no network) and a real HTTP back end.
- Voice input with live transcript, silence timeout, haptics and VoiceOver announcements.
- Hotel result tiles, hotel summary card, and hotel detail screen, with swipeable photo carousel.
- Accessibility highlights with automatic SF Symbol icon matching.
- Pagination of results with a "Show More" chip.
- Google Maps deep link per hotel.
- Strong accessibility support: labels, hints, traits, 44pt tap targets, Reduce Motion handling.
- Dark-only visual design driven by a central theme.

### 1.3 Technology

| Area | Choice |
| --- | --- |
| UI | SwiftUI |
| Concurrency | Swift async/await, `Task`, `MainActor.run` |
| Networking | `URLSession`, `JSONEncoder`/`JSONDecoder` |
| Speech | `Speech` (`SFSpeechRecognizer`), `AVFoundation` (`AVAudioEngine`, `AVAudioSession`) |
| Images | `AsyncImage` |
| Minimum iOS | iOS 15 supported (compat helpers for iOS 16/17 differences) |
| Locale for speech | `en-AU` |

## 2. Architecture

### 2.1 Pattern

The app is a small, view-state-driven SwiftUI app. There is no separate view-model class: `chatScreen` owns the conversation state with `@State` and delegates reply generation to an injected object conforming to the `chatService` protocol. Speech is a separate `ObservableObject`.

```
MyApp
  └── chatScreen  (state: messages, hotelPool, hotelsShown, input text)
        ├── chatHeader
        ├── ScrollView of bubbles
        │     ├── userText
        │     ├── expertText ── chatBubbleText + suggestionChips
        │     ├── hotelText  ── chatBubbleText + hotelPreviewTile(s) + suggestionChips
        │     │      └── sheet: hotelCard
        │     │             └── sheet: hotelDescription
        │     └── typingIndicator
        ├── messageInput (mic, text field, send)
        ├── SpeechRecognizer (StateObject)
        └── service: chatService
              ├── mockChatService
              ├── keywordChatService (local script)
              └── BackendChatService (HTTP)
```

### 2.2 Layers

1. **Model**: `chatMessage`, `chatRole`, `hotel`, `expertReply`.
2. **Service**: `chatService` protocol and its three implementations.
3. **View**: chat components, hotel components and screens.
4. **Platform helpers**: `SpeechRecognizer`, `onChangeCompat`.
5. **Design system**: `theme`.

### 2.3 Data flow for one user turn

1. User types, speaks or taps a chip, which calls `sendUserMessage(_:)`.
2. The text is trimmed. Empty text or a send while waiting is ignored.
3. Speech is cancelled so late words cannot refill the field.
4. A user bubble is appended and the input is cleared.
5. Built-in commands are handled locally: `show more` (when more hotels remain in the pool) and `new chat`.
6. Otherwise `isWaitingForReply = true` and the typing indicator appears.
7. `service.reply(to:history:)` runs asynchronously.
8. On success the hotel pool is replaced. If no hotels came back, a plain expert message is appended. If hotels did, `appendHotelPage` shows the first page.
9. On failure a generic apology message is appended.
10. The scroll view animates to the bottom whenever the message count or waiting state changes.

## 3. File reference

| File | Type(s) | Responsibility |
| --- | --- | --- |
| `MyApp.swift` | `MyApp` | App entry point; chooses which service to inject |
| `chatScreen.swift` | `chatScreen` | Main screen: state, bubble routing, sending, pagination, speech wiring |
| `chatMessage.swift` | `chatRole`, `chatMessage` | Conversation message model |
| `chatService.swift` | `expertReply`, `chatService`, `mockChatService`, `BackendChatService` | Service abstraction, mock and HTTP implementation, JSON mapping |
| `keywordChatService.swift` | `keywordChatService` | Scripted offline conversation engine |
| `chatComponents.swift` | `chatBubbleText`, `suggestionChips` | Shared bubble and quick-reply chips |
| `expertText.swift` | `expertText` | Expert text bubble with chips |
| `userText.swift` | `userText` | User text bubble |
| `hotelText.swift` | `hotelText` | Expert bubble containing hotel tiles; presents `hotelCard` sheet |
| `typingIndiator.swift` | `typingIndicator` | Animated three-dot waiting bubble (file name has a typo) |
| `header.swift` | `chatHeader` | Chat screen top bar with close button |
| `messageInput.swift` | `messageInput` | Mic button, growing text field, send button, AI disclaimer |
| `speechRecognizer.swift` | `SpeechRecognizer` | Microphone capture and live speech-to-text |
| `hotel.swift` | `hotel` | Hotel model, display helpers, Maps URL, sample data |
| `hotelComponents.swift` | `hotelHeader`, `hotelSectionTitle`, `hotelPhoto`, `photoCarousel`, `StarRating`, `hotelSummary`, `hotelLocationSection`, `mapsLinkButton`, `hotelDivider`, `primaryButton`, `hotelPreviewTile` | Shared hotel UI building blocks |
| `hotelCard.swift` | `hotelCard` | Hotel summary screen (photos, description, highlights, location) |
| `hotelDescription.swift` | `hotelDescription` | Hotel detail screen (benefits, location, Book Now) |
| `theme.swift` | `theme` | Colours and the rounded font helper |
| `Viewcompat.swift` | `View.onChangeCompat` | Cross-version `onChange` wrapper |

## 4. Models

### 4.1 `chatMessage`

| Property | Type | Notes |
| --- | --- | --- |
| `id` | `UUID` | Stable identity for `ForEach` and scrolling |
| `role` | `chatRole` | `.expert` or `.user` |
| `text` | `String` | Bubble text |
| `suggestions` | `[String]` | Quick-reply chip labels |
| `hotels` | `[hotel]` | If non-empty on an expert message, it renders as `hotelText` |

`showsHotels` returns `true` when `hotels` is not empty. The type is `Identifiable` and `Equatable`.

### 4.2 `expertReply`

What a service returns: `text`, `hotels` (default empty) and `suggestions` (default empty).

### 4.3 `hotel`

| Property | Type | Purpose |
| --- | --- | --- |
| `id` | `UUID` | Generated locally, so every decode produces new IDs |
| `name` | `String` | Display name |
| `location` | `String` | Short place text, such as suburb and state |
| `distanceFromAirport` | `String` | Pre-formatted text, such as "1.2 km from airport" |
| `pricePerNight` | `Int` | `0` means unknown |
| `rating` | `Double` | `0` means no rating |
| `reviewCount` | `Int` | Number of reviews |
| `description` | `String` | Long text |
| `accessibilityHighlights` | `[String]` | Short list shown as an icon grid |
| `benefits` | `[String]` | Full list on the detail screen |
| `beds`, `showers` | `Int` | Counts shown in the summary |
| `photoURLs` | `[URL]` | Remote photos; empty shows a placeholder |
| `address` | `String` | Full street address, used for the Maps query |

**Computed helpers** make missing back-end data safe to display:

- `hasPrice`, `hasRating`
- `priceText` ("$125" or "Price unavailable")
- `ratingText` ("4.5" or "No ratings")
- `reviewText` (handles singular and plural)
- `addressLine` (address, falling back to location)
- `mapsURL`, a `https://www.google.com/maps/search/?api=1&query=<name, address>` URL. Returns `nil` if there is nothing to search for.

**Sample data**: `hotel.samples` provides four hotels (Melbourne, Melbourne, Geelong, Sydney) using Picsum placeholder photos. A commented-out helper shows how to supply your own image URL strings.

## 5. Services

### 5.1 `chatService` protocol

```swift
protocol chatService {
    func reply(to text: String, history: [chatMessage]) async throws -> expertReply
}
```

Any object that implements this can power the chat. `chatScreen` receives it in its initialisers (default `mockChatService()`).

### 5.2 `mockChatService`

Waits 0.9 s and returns a fixed placeholder message. Useful for previews and UI work.

### 5.3 `keywordChatService` (offline script)

A stateless, keyword-driven bot. Every call rebuilds the user's needs from the full `history`, so no stored state is required. Configurable properties: `delay` (default 700 ms) and `hotelsToShow` (default 2).

**Conversation stages**

1. Welcome (greeting, or a help request)
2. Identify needs (wheelchair, hearing, vision)
3. Clarify (step-free, bathroom, wider doorways)
4. Travel details (city, nights)
5. Match (offers to show options, then returns hotels)
6. Direction (refuses to choose for the user)
7. Confirmation (thanks, tourist-info offer, goodbye)

**Needs profile** (`Needs`): `wheelchair`, `stepFree`, `bathroom`, `wider`, `city`, `nights`. It is rebuilt by scanning the user's messages. Short "yes" answers are interpreted using the preceding expert question.

**Order of checks in `respond`** (first match wins, so order matters):

1. Decision questions after hotels were shown ("which", "recommend", "best"...) get a refusal to decide for the user.
2. Goodbye.
3. Negative first word ("no", "nope"...), answered according to the previous expert question.
4. City or nights mentioned: asks for whichever is missing, or offers to show options when both are known.
5. A stated need or a yes to a clarifying question: summarises needs and asks the next question.
6. Wheelchair.
7. Hearing or vision.
8. Thanks.
9. Plain "yes" to "show you suitable options" (returns hotels), to the tourist offer, or to "anything else".
10. Help keywords.
11. Greeting.
12. Capability question.
13. Fallback with starter chips.

**Helpers**: `words` tokenises text, `has` checks substring matches, `nightCount` understands "5 nights", "five nights", "a week" (7) and a bare number when the expert just asked about nights, and `joined` builds natural lists ("a, b and c").

**Important coupling**: the bot decides context by searching the previous expert message for phrases such as `"do you need step-free"` and `"how many nights"`. If you change a reply's wording you must change the matching check too.

### 5.4 `BackendChatService` (HTTP)

**Request**

```
POST <endpoint>            e.g. http://localhost:8080/api/ai/chat
Content-Type: application/json

{ "content": "<user text>" }
```

Timeout is 30 seconds. Non-2xx responses throw `URLError(.badServerResponse)`.

**Response** (only the fields below are decoded; others are ignored)

```json
{
  "content": "Reply text",
  "warnings": ["..."],
  "hotels": [
    {
      "providerId": "...",
      "airportIata": "SYD",
      "name": "Hotel name",
      "address": "45 A Avenue, Sydney NSW 2000",
      "distanceKm": 3.2,
      "description": "...",
      "images": [{ "url": "https://...", "altText": "..." }],
      "rating": { "score": 4.5, "reviewCount": 41 },
      "accessibility": {
        "rollInShower": { "status": "confirmed", "source": "..." }
      },
      "offer": { "currency": "AUD", "nightly": 125, "available": true, "bookingUrl": "..." }
    }
  ]
}
```

Every hotel field is optional, so a partial hotel does not fail the whole response.

**Mapping to the `hotel` model** (`BackendHotel.model`)

| UI field | Source |
| --- | --- |
| `name` | `name`, default "Hotel" |
| `location` | Address after the first comma ("45 A Avenue, Sydney NSW 2000" becomes "Sydney NSW 2000") |
| `distanceFromAirport` | `distanceKm` and `airportIata`, formatted "3.2 km from SYD" |
| `pricePerNight` | `offer.nightly`, else 0 |
| `rating`, `reviewCount` | `rating.score`, `rating.reviewCount`, else 0 |
| `accessibilityHighlights` | Only features with status `confirmed` |
| `benefits` | `confirmed` features with " (confirmed)" and `provider_claimed` ones with " (claimed by provider)"; `unknown` omitted |
| `beds`, `showers` | Hard-coded 1 (not supplied by back end yet) |
| `photoURLs` | Image URLs that parse as valid URLs |
| `address` | Full address, used for Maps |

Supported feature keys, in fixed display order: `stepFreeEntrance`, `wheelchairAccessibleRoom`, `rollInShower`, `accessibleToilet`, `grabRails`, `emergencyCallCord`, `accessibleParking`, `accessibleAirportShuttle`.

**Note**: the service only sends the latest user text. `history` is accepted but not transmitted, so the server has no conversation memory unless it keeps session state itself. `reply.suggestions` is always empty from this service.

## 6. Screens and views

### 6.1 `chatScreen`

The root screen. Two initialisers:

- `init(messages:service:)`: seeds the full history. Empty history falls back to the default opening ("Welcome back! What would you like to do?" with chips "Hotel" and "New chat").
- `init(expertMessage:suggestions:userMessage:service:)`: seeds one expert message and optionally one user message.

**State**

| State | Meaning |
| --- | --- |
| `messages` | The conversation |
| `messageText` | Text in the input field |
| `isWaitingForReply` | Shows typing indicator and blocks sending |
| `hotelPool` | All hotels returned for the current search |
| `hotelsShown` | How many pool entries have been displayed |
| `speech` | `SpeechRecognizer` (`@StateObject`) |

**Behaviours**

- **Bubble routing**: expert messages with hotels render as `hotelText`, other expert messages as `expertText`, user messages as `userText`.
- **Auto scroll**: uses `ScrollViewReader`; scrolls to the typing indicator while waiting, otherwise to the last message.
- **Voice transcript**: `speech.transcript` changes are copied into `messageText`, so the user can edit before sending.
- **Backgrounding**: if the scene phase is not active, listening stops.
- **Speech errors**: shown in an alert with an "Open Settings" button.
- **Close button**: cancels speech and dismisses.
- **Pagination**: page size 2. `appendHotelPage` builds chips from any extra chips, then the hotel names on that page, then "Show More" if more remain. Typing "Show More" (or tapping the chip) shows the next page without calling the service. When the pool is exhausted, it says so and offers "New chat".
- **New chat**: resets messages to the opening and clears the hotel pool.
- **Errors**: any thrown error becomes "Sorry, I couldn't get a response. Please try again."

### 6.2 Chat components

- `chatBubbleText`: rounded 15pt bubble, 16pt medium rounded font, max width 305.
- `suggestionChips`: horizontally scrolling capsules, each at least 44pt tall, with accessibility label and the hint "Sends this as your reply".
- `expertText`, `userText`: title label above a bubble; expert is leading aligned and user is trailing aligned and purple.
- `typingIndicator`: three dots with a staggered repeating scale animation.
- `chatHeader`: centred title ("Chat with The Expert") with a circular close button; a clear 44pt spacer balances layout.

### 6.3 `messageInput`

- **Mic button**: grey when idle, red when recording, with a pulsing ring (disabled if Reduce Motion is on). Accessibility label toggles between "Start voice input" and "Stop voice input".
- **Text field**: vertical axis, grows from 1 to 5 lines, rounded rectangle, "Recording...." placeholder while listening, Return key labelled Send.
- **Send button**: disabled and 40% opacity when the text is blank.
- **Disclaimer**: "Responses are generated by AI and may be inaccurate. Please verify important information".

### 6.4 Hotel views

- `hotelText`: expert bubble plus a column of `hotelPreviewTile`s (max width 305). Tapping a tile, or a chip whose text equals a hotel name (case-insensitive), opens `hotelCard` as a sheet. Other chips are forwarded to `onSuggestionTap`.
- `hotelPreviewTile`: name, first photo, location, price and rating on one line. Fully combined accessibility element with a descriptive label.
- `hotelCard`: header with back button, photo carousel, summary, description, accessibility highlights (two-column grid with icons), location section, and a "More Information" button that opens `hotelDescription` as a sheet.
- `hotelDescription`: same header, carousel and summary, then location, full benefits list and a "Book Now" button (`onBookNow` callback, currently a no-op default).
- `hotelSummary`: location and airport distance, price per night, star rating with review count, shower and bed counts.
- `photoCarousel`: paged `TabView`, arrow buttons (30pt visual, 44pt tap area), dot indicators, resets to the first photo when the photo list changes, wraps around at the ends.
- `hotelPhoto`: `AsyncImage` with loading spinner, failure icon and no-URL placeholder; clipped to a rounded rectangle.
- `StarRating`: five stars with full, half and empty states.
- `mapsLinkButton`: "View on Google Maps" capsule `Link`; hidden when `mapsURL` is nil.
- `primaryButton`, `hotelDivider`, `hotelSectionTitle`, `hotelHeader`: shared styling pieces.

**Accessibility icon matching** (`hotelCard.iconName(for:)`): the first rule whose keyword is found in the lower-cased highlight wins; otherwise a checkmark. Rules are ordered so that, for example, "Large shower with seats" gets the shower icon rather than the chair. Mapped concepts include hearing, elevator, wheelchair/step-free/ramp, shower, bath, toilet, seating, braille, parking, shuttle, emergency, visual, doors, alarms, grab rails, service animals and wide doorways.

## 7. Speech recognition

`SpeechRecognizer` is a `final class ObservableObject` (works on iOS 15+) with published `transcript`, `isRecording` and `errorMessage`.

| Method | Behaviour |
| --- | --- |
| `toggle()` | Starts if idle, stops if recording |
| `start()` | Requests permissions, configures the audio session (`.record`, `.measurement`, duck others), installs the input tap, starts the engine and the recognition task, announces "Listening" to VoiceOver, gives a medium haptic, starts the silence timer |
| `stop()` | Stops audio and ends the request. Final words may still arrive and update `transcript`. Light haptic and "Stopped listening" announcement |
| `cancel()` | Immediate hard stop. Increments `generation` so late results are ignored. Deactivates the audio session |
| `finish()` | Internal clean-up after a final result or error |

**Design points**

- `generation` counter ignores callbacks from earlier recordings.
- `isStarting` prevents a second start during permission prompts.
- On-device recognition is required whenever supported, for privacy.
- Silence timeout is 3 seconds, reset by each new partial result.
- Microphone permission uses `AVAudioApplication` on iOS 17+ and `AVAudioSession` before that.

## 8. Theme and design system

`theme` is a caseless enum of static tokens: accent purple, field and card backgrounds, close button colours, bubble colours, title grey, rating gold, empty star grey, button purple, photo placeholder, and `secondaryText` (white at 65% opacity, chosen for roughly 6:1 contrast). `theme.rounded(size, weight)` returns an SF Pro Rounded system font. All screens force `.preferredColorScheme(.dark)`.

`onChangeCompat(of:perform:)` wraps `onChange` so the same code compiles for the iOS 17 two-parameter form and the older single-parameter form.

## 9. Accessibility

The app is designed for users with accessibility needs, and the code reflects it:

- Every interactive control has an accessibility label, and most have hints.
- Complex items (tiles, stats, price, rating) are merged into one element with a readable label.
- Section titles and the hotel title carry the header trait.
- Decorative icons are hidden from VoiceOver.
- Minimum 44pt tap targets for chips, arrows, buttons and the Maps link.
- Reduce Motion disables the mic pulse.
- Haptic and spoken feedback when recording starts and stops.
- Missing data is described in words ("Price unavailable", "No ratings yet") rather than hidden.
- Hotel accessibility features distinguish confirmed from provider-claimed information.

## 10. Setup and configuration

### 10.1 Choosing a chat backend

In `MyApp.swift`, exactly one of the following must be active:

```swift
// Real back end
chatScreen(service: BackendChatService(
    endpoint: URL(string: "http://localhost:8080/api/ai/chat")!))

// Offline, scripted
chatScreen(service: keywordChatService())
```

The file currently has the backend version active and the keyword version commented out.

### 10.2 Info.plist requirements

| Key | Reason |
| --- | --- |
| `NSMicrophoneUsageDescription` | Voice input |
| `NSSpeechRecognitionUsageDescription` | Speech-to-text |
| App Transport Security exception | Needed for `http://` endpoints such as localhost. Use HTTPS in production |

### 10.3 Running against a local server

- The iOS Simulator can reach `localhost` on the Mac.
- A physical device cannot. Use the Mac's LAN IP address (and allow it in ATS) or deploy the back end.

### 10.4 Swapping hotel photos

In `hotel.swift`, comment out the Picsum `photos(_:count:)` helper and uncomment the variadic version, then pass your own URL strings.

## 11. Testing and previews

Xcode previews exist for `chatHeader`, `hotelCard`, `hotelDescription`, `hotelText`, `messageInput`, `typingIndicator`, `userText`, `expertText` and `chatScreen`. For fast UI work, use `keywordChatService()` or `mockChatService()`.

**Suggested manual test script for the offline bot**

1. "Hello" → greeting with "Find accommodation" chip.
2. "I use a wheelchair" → asks about step-free access.
3. "Yes" → asks about an accessible bathroom.
4. "Yes" → asks about anything else.
5. "No" → asks for destination and nights.
6. "Melbourne for 3 nights" → offers to show options.
7. "Yes" → two hotel tiles.
8. Tap a tile → card, then More Information → description.
9. "Which is best?" → the bot declines to choose.
10. "Thanks" → tourist-info offer.

## 12. Known issues and improvement suggestions

These are observations from reading the code, in rough priority order.

1. **Conversation history is not sent to the backend.** `BackendChatService` only posts the latest message. Unless the server tracks sessions, multi-turn context (such as "yes") is lost. Consider adding a session ID or sending history.
2. **"Show More" and hotel-name chips only work client-side.** Chips named after hotels depend on exact name matches in `hotelText`.
3. **Hotel `id` is regenerated on every decode.** `var id = UUID()` means identity is not stable across responses. Consider using the back end's `providerId`.
4. **Hard-coded beds and showers.** The back end does not supply them, so every real hotel shows 1 bed and 1 shower, which could mislead users. Consider hiding these stats when unknown.
5. **"Book Now" does nothing.** `onBookNow` is never wired, and `offer.bookingUrl` is decoded by `BackendOffer` but unused. Linking to it would complete the flow.
6. **Price currency is ignored.** The UI always prefixes "$" even if `offer.currency` differs.
7. **`warnings` from the back end are decoded but not shown.**
8. **Nested sheets.** `hotelDescription` is presented over `hotelCard`, which is itself a sheet. `hotelCard`'s `onBack` closes only the top sheet; the Done flow from the description is `showDescription = false`. Check that dismissing behaves as intended on small devices.
9. **Typing indicator animation.** It starts once via `onAppear`; under Reduce Motion it still animates. Consider honouring `accessibilityReduceMotion` there too.
10. **Keyword bot fragility.** Context detection relies on matching exact phrases from earlier replies. A shared constants file would prevent drift.
11. **Error messages are generic.** Differentiate offline, timeout and server errors for clearer recovery.
12. **Naming and file hygiene.** Types use lowerCamelCase (`hotel`, `chatScreen`), which differs from Swift convention (UpperCamelCase) and creates shadowing risk, for example the `hotel` parameter name inside views of type `hotel`. File name typos exist (`typingIndiator.swift`), and several file headers refer to other file names. `Viewcompat.swift` should be `ViewCompat.swift`.
13. **Mixed naming.** `SpeechRecognizer`, `StarRating` and `BackendChatService` are capitalised while most other types are not.
14. **Disclaimer and localisation.** Strings are hard-coded in English. Consider `LocalizedStringKey` or a strings catalogue.
15. **Dark mode only.** Forced dark scheme may conflict with some users' display needs. Consider supporting light and high-contrast modes.
16. **Unit tests.** `keywordChatService` is pure logic and well suited to tests, as is `BackendHotel` mapping and `nightCount`.

## 13. Glossary

| Term | Meaning |
| --- | --- |
| Expert | The chatbot persona |
| Chip | A tappable quick-reply capsule |
| Tile | A compact hotel preview inside a chat bubble |
| Hotel pool | All hotels from the latest service reply, paged two at a time |
| Confirmed / provider-claimed | Whether an accessibility feature is verified or only asserted by the provider |
| ATS | App Transport Security, iOS's HTTPS requirement |

## 14. Quick start checklist

1. Open the project in Xcode and add the microphone and speech usage descriptions to Info.plist.
2. In `MyApp.swift`, pick `BackendChatService` or `keywordChatService`.
3. If using the back end, start it on port 8080 and confirm `POST /api/ai/chat` returns the documented JSON.
4. Run on the simulator, grant microphone and speech permissions on first mic tap.
5. Walk through the test script in section 11.
