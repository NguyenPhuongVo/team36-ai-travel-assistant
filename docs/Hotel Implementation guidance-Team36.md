Hotel Implementation guidance

1. Summary

The Mock-ups are for the proposed actions of the chat with the AI “Expert” that helps users find accessible accommodation according to their requirements. The user can type or use voice to communicate with the Expert. The Expert asks and confirms accessibility needs and returns hotel cards according to the given requirements. Users can select the chosen hotel for more information. This opens a card which includes information about the room/hotel alongside either book now or more information. 

This is for adding to current application. 

- Hierarchy: the detail card leads with identity (name, image), then location, price and rating, then Accessibility Highlights, then the primary action. Accessibility content is the priority information in the body.

- Blocking gaps: The current Design shows for use of book Now or More Information. This requirement needs to be confirmed with the client.

- Accessibility: many controls are icon-only or use small targets and low-contrast grey text. 

1. Reviewed

The PDF is mostly animation frames of one flow. Pages 1 to 4 hold the icon set, the photo gallery, the static hotel layouts and the empty chat screen; pages 5 to 30 step through the conversation.

- Empty chat: "Chat with The Expert" header, close (X), message field, microphone button, send button, AI disclaimer.

- Text and voice input: typed message bubbles; recording state (red record button, "Recording…", waveform bubble with transcript).

- Conversation: the Expert asks about accessibility needs; the user replies (wheelchair user, step-free access, accessible bathroom, wider entrances) and states a 5-night stay in Melbourne.

- Results: Expert message with hotel 1 and Hotel 2 cards (name and photo only).

- Detail card (Hotel 2): summary view with Accessibility Highlights and Book Now; Benefits view with a 12-item list and More Information.

1. Hotel detail card

Order below is top to bottom and should also be the screen-reader reading order.

| Order | Element | Status | Implementation note |
| --- | --- | --- | --- |
| 1 | Dismiss control (down arrow) | Required | Needs an accessible name and a defined dismiss behaviour (Section 4). |
| 2 | Hotel name | Required | Mark up as a heading. |
| 3 | Image carousel | Required (min. 1 image) | Show arrows and dots only when there is more than one image. Dot count must come from the data (design shows 5 dots; gallery asset has 4 photos). |
| 4 | Location and distance from airport | Required / conditional | Show distance only when it is relevant to the hotel. |
| 5 | Price per night | Required | Locale-aware currency formatting; currency and tax basis to be confirmed. |
| 6 | Rating and review count | Conditional | Hide if no reviews. Stars are decorative; expose "4.9 out of 5, 41 reviews" once. |
| 7 | Bed and bathroom counts | Optional | Icons are unlabelled in the design; add visible or accessible text ("1 bed, 1 bathroom"). |
| 8 | Description | Optional | Needs a maximum length or truncation rule. Placeholder text is lorem ipsum. |
| 9 | Accessibility Highlights (4 items) | Required; highest priority in body | Icon plus text label. Selection and ordering rule to be confirmed (Section 4). |
| 10 | Primary action (Book Now) | Required | Booking flow is not designed in Sprint 1; a hand-off point is needed. |
|  |  |  |  |
1.  Accessibility and interaction requirements

Level: Must = required for release; Should = strongly recommended. Colour-contrast and target-size items should be verified against the real design tokens, not the wireframe images.

| Area | Requirement | Level |
| --- | --- | --- |
| Labels | Every icon-only control has an accessible name: Close, start voice input, Send, Collapse hotel details, Previous image, Next image. | Must |
| Messages | Each new Expert message is announced politely via a live region without moving focus. Streamed text is not announced token by token. Messages identify speaker (Expert / User). | Must |
| Carousel | Expose position ("Image 2 of 5"). Provide button alternatives to swiping. No autoplay. Dots are not the only position indicator. | Must |
| Image alt text | Alt text describes the accessibility features visible (for example "Bathroom with roll-in shower, grab rails and fold-down seat"). The photos are evidence for the accessibility claims. | Must |
| Icons and ratings | Highlights icons are decorative because the text label carries the meaning. Stars are decorative. Price and unit are read together ("$125 per night"). | Must |
| Counts | Bed and bathroom counts have text alternatives ("1 bed, 1 bathroom"). | Must |
| Focus | Opening the detail card moves focus to the hotel name and traps focus while open. Closing returns focus to the originating card. | Must |
| Reading order | Follows the visual order in Section 3.1. | Must |
| Text scaling | Layout supports system text scaling and Dynamic Type. No fixed heights on the card, chat bubbles or Benefits list; the card scrolls and the primary action remains reachable. | Must |
| Contrast | Grey secondary text (message placeholder, disclaimer, "(41 reviews)", "Per night") and the white-on-lavender Book Now label look close to the limit. Measure against 4.5:1 for normal text. | Must |
| Target size | Carousel arrows and dots look small. Provide at least 44×44 pt (iOS) or 48×48 dp (Android) hit areas even if the visible glyph stays small. | Must |
| Not colour alone | Recording state uses a red button and the text "Recording…". Keep both. | Must |
| Voice input | The waveform bubble is decorative; the transcript text is readable by assistive technology. Recording can be stopped or cancelled without voice. | Must |
| Reduced motion | Card transitions and carousel animation respect the system reduced-motion setting. | Should |
| Layout | Description text is centred in the wireframe. Consider left alignment for multi-line text; UX to confirm. | Should |

