# AI Travel Assistant for Accessible Aviation: Hotel & Booking Services Integration

**RMIT Group 36 — AI Travel Assistant Team B**

This repository is our team's shared documentation hub for the capstone project. We use it to collect, organise, and review requirements, research, meeting notes, designs, integration findings, and final deliverables.

This repository holds documentation and supporting files. Link the prototype's development repository and YAF environment here once confirmed.

## Client brief

The client wants a prototype integrated into **YAF** that helps travellers find accommodation and travel services suited to their accessibility needs. It should answer questions such as:

- “Find accessible hotels near the airport.”
- “Which hotel is more suitable for a wheelchair user?”
- “Can I book an accessible airport transfer?”
- “What accessible services are available nearby?”

The project should also be able to use flight data and aviation-related sources, including **Flightradar / Flightradar24**. Research must consider hotel APIs, booking services, other travel services, data aggregators, and chatbot/API workflows.

### Minimum expected outcome

- A working integration with **at least one hotel or booking provider**.
- A prototype of this functionality **integrated into YAF**.
- A ready-made list of relevant links and resources for research, testing, and API integration.

Documentation supports these outcomes; it does not replace the working prototype. Provider selection, booking behaviour, and the scope of flight and transfer integration still need to be agreed with the client. Label live, sandbox, and mock demonstrations clearly. Mock data alone does not demonstrate a working provider integration.


## Team

| Team member | Role |
| --- | --- |
| Johanne Gutierrez | Project Manager |
| Ethan Dalianis | Business Analyst |
| Rajvansh Singh Padda | UX Designer |
| Nguyen Phuong Vo | Developer |
| Sean Renzo Mojica | Developer |


## Team documents

The team has prepared the following files, as listed in our master document. Available files are linked below; the remaining supporting files still need to be uploaded or linked here.

| File / document | Summary |
| --- | --- |
| [UX Brief v2](docs/UX%20Brief%20v2.md) | User personas, accessibility needs, usability barriers, and proposed travel-assistant features |
| [Technical Architecture and Technology Stack](docs/Technical%20Architecture%20and%20Technology%20Stack.pdf) | Proposed architecture, AI services, external APIs, dependencies, and technical risks |
| Client Requirement Analysis Document | Initial requirements analysis, refined following client feedback |
| [YAF Technical Review – Sprint 1 Week 2](docs/YAF_technical_review.md) | Technical feasibility, user flow, constraints, risks, and external dependencies |
| User interaction flow document | Sequence diagram, example chatbot interaction, and accessibility considerations |
| [Reviewing existing chatbot implementation](docs/Reviewing%20existing%20chatbot%20implementation.pdf) | Review of YAF's chatbot, AI API, database, limitations, and potential improvements |
| Analysis of initial client requirements | Updated detailed MVP requirements for PM review before submission |
| Chatbot interaction wireframe | Initial low-fidelity chatbot interaction wireframes |
| [Sprint 2 Codebase and Development Environment Preparation](docs/sprint-2-codebase-and-development-environment-preparation.md) | Repository review, team branch workflow, local environment setup, dependencies, and development readiness |
| [User Stories and Requirements](docs/User-stories-and-requirements.md) | Prioritised user stories, acceptance criteria, MVP scope, and future improvements |
| [Sprint 2 Technical Implementation Approach](docs/Sprint%202%20Technical%20Implementation%20Approach.pdf) | Proposed technical approach for Sprint 2 development and hotel or booking API integration |
| Playback 1 Presentation — [PowerPoint](docs/36-Playback1.pptx) / [PDF](docs/36-Playback1.pdf) | Sprint 1 achievements, team contributions, blockers, and next steps |
| [Hotel/Booking Provider Preparation](docs/Hotel-provider.md) | Provider status, hotel data fields, local fixture testing, and outstanding API access requirements |
| [Hotel Response Matching and Structured Chatbot Response](docs/response-matching.md) | Matching logic, response contract, reported test results, and 12 representative requests with fields for recording responses |
| [AI Testing Requirements](docs/AI%20testing%20requirements.pdf) | Requirements for testing the YAF chatbot and evaluating its responses |
| [AI Provider Connection Verification](docs/AI%20Provider%20Connection%20Verification.pdf) | Documentation of AI provider connection verification |
| [AI Provider Connection Verification — Pinecone Version](docs/AI%20Provider%20Connection%20Verification%20-%20Pinecone%20Version.pdf) | Documentation of Pinecone provider connection verification |
| [YAF Chatbot Prompt Evaluation](docs/prompt_evaluation.md) | Comparison of prompts A–D, hotel, airport, and flight test results, response evidence, and recommended improvements |
| [YAF AI Retrieval and Response Quality Improvements](docs/ai-retrieval-response-quality.md) | Fallback handling, keyword reranking, hotel filtering and sorting, and the assistant’s remaining limitations |
| [Hotel Implementation Guidance](docs/Hotel%20Implementation%20guidance-Team36.md) | Hotel-card layout, interaction requirements, accessibility considerations, and design decisions to confirm |
| [Failure States](docs/Failure%20states-team%2036.md) | Proposed chatbot responses and accessibility considerations for missing information, errors, and unsupported requests |
| [Voice Input: Test Log and Known Limitations](docs/Voice%20Input-Test%20Log%20and%20Known%20Limitations.md) | Voice-input test checklist, accessibility checks, known limitations, and suggested refinements |
| [Sprint 2 Playback Presentation](docs/Playback%20Sprint%202.pdf) | Sprint 2 playback presentation |

## Design and demonstration files

| File / resource | Summary |
| --- | --- |
| [Team 36 Sprint Design](figma/Team-36-Sprint.fig) | Figma design file for the team prototype |
| [Updated Design](images/updated-design.png) | Updated interface design image |
| [Voice Prototype](images/vids/Voice-Prototype.mp4) | Voice prototype demonstration video |
| [Prototype Demonstration](images/vids/Prototype_working.mp4) | Prototype demonstration recording |
| App recordings, 26 September 2026: [iPhone](images/vids/app_iphone_26-09-26.mp4) / [App](images/vids/app_26-09-26.mov) | App prototype recordings |
| [Supporting Screenshots](images/) | Screenshots of local setup, backend operation, health checks, and test results |
