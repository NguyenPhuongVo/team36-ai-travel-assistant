Voice Input: Test Log and Known Limitations

Test on a physical iPhone (not the simulator). Fill in the Result column as you go (Pass / Fail / Notes).

1. Representative query tests

Queries are based on the scripted flow in `keywordChatService`. Speak each one rather than typing it.

| # | Spoken input | Context | Expected result | Result |
| --- | --- | --- | --- | --- |
| 1 | "I use a wheelchair" | Start of chat | Expert asks about step-free access, Yes/No chips |  |
| 2 | "Yes" | After step-free question | "Got it. You're looking for step-free access..." |  |
| 3 | "Wider entrances" | After "anything else" question | Summary includes wider entrances, asks destination + nights |  |
| 4 | "Melbourne for five nights" | Asked for travel details | Destination and duration captured, offers to show options |  |
| 5 | "Five" | Expert just asked "how many nights" | Bare number accepted as nights |  |
| 6 | "Yes" | After "show you suitable options" | Hotel tiles appear |  |
| 7 | "Hotel" | Any time | Hotel list shown locally (built-in command) |  |
| 8 | "Show more" | After hotels shown | Next page of hotels |  |
| 9 | "Thank you" | After hotels | Offers tourist information |  |
| 10 | "Hearing impaired" | Start of chat | Asks destination and nights |  |
| 11 | "I need a wheelchair accessible room in Sydney for three nights" | Start of chat (long sentence) | Destination and nights captured in one go |  |
| 12 | "New chat" | Any time | Conversation resets |  |
| 13 | "Geelong" (check recognition) | Asked for city | City recognised (watch for mis-hearing) |  |
| 14 | "No" | After each yes/no question | Moves to next question |  |
2. Voice-specific behaviour tests

| # | Scenario | Expected | Result |
| --- | --- | --- | --- |
| 15 | First tap on mic | Permission prompts appear (speech + microphone) |  |
| 16 | Deny permission | Clear message with a way to open Settings; no crash |  |
| 17 | Tap mic, say nothing | Stops automatically after a few seconds of silence |  |
| 18 | Tap mic twice quickly | No crash, no duplicate sessions |  |
| 19 | Tap Send while still recording | Recording stops, message sends once |  |
| 20 | Background the app while recording | Recording stops cleanly |  |
| 21 | Incoming call / Siri during recording | Recording stops cleanly, mic can be restarted |  |
| 22 | Speak in a noisy room | Note accuracy |  |
| 23 | Airplane mode | Note whether recognition still works (on-device) |  |
| 24 | Edit transcribed text before sending | Edits are kept, send works |  |
| 25 | Speak a long sentence (30+ seconds) | Note whether it cuts off |  |
3. Accessibility checks

| # | Check | Result |
| --- | --- | --- |
| 26 | VoiceOver reads the mic as "Start voice input" / "Stop voice input" |  |
| 27 | VoiceOver announces "Listening" when recording starts |  |
| 28 | Recording state is not shown by colour alone (icon + placeholder text change) |  |
| 29 | Largest Dynamic Type size: bubbles, chips and input remain readable |  |
| 30 | All tappable controls are at least 44x44 pt (mic, send, chips) |  |
| 31 | Reduce Motion on: mic pulse animation is disabled |  |
| 32 | Haptic feedback on start/stop |  |
| 33 | Text contrast is readable (disclaimer text in particular) |  |
4. Known issues and limitations (edit to match your findings)

Language/accent: recognition is set to `en``-AU`. Other accents and languages are not tested and may be less accurate.

Network: Apple's recogniser may use servers when on-device recognition is unavailable, so accuracy and availability can vary with connectivity.

Duration: a single recognition session is limited by Apple (roughly one minute), so very long speech will be cut off.

Misheard words: place names (e.g. "Geelong") and accessibility terms may be mis-transcribed, and the keyword-based chatbot will then fall back to "Sorry, I didn't quite catch that".

Keyword matching: `keywordChatService` only reacts to specific phrases. Natural phrasing outside those keywords is not understood, whether typed or spoken.

Backend: `BackendChatService` sends only the latest message, not the conversation history, so spoken follow-ups like "yes" may lose context on the real backend.

No auto-send: the transcript fills the text box and the user must tap send, so there is one extra step.

VoiceOver: VoiceOver speech may be picked up by the mic if it starts speaking while recording.

Simulator: microphone and speech recognition are unreliable in the simulator; only physical-device results are valid.

Permissions: if the user denies microphone or speech access, voice input stays unavailable until they change it in Settings.

5. Suggested refinements

Auto-send option after the user stops speaking.

Custom vocabulary hints for city names and accessibility terms (`contextualStrings` on the recognition request).

Require on-device recognition where supported, for privacy and offline use.

Multi-language support.

Replace keyword matching with a language-model backend for more natural queries.


