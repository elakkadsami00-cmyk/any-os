# Sovereign School OS - Technical Deep Dive

This document provides a detailed technical overview of the Sovereign School OS application. It is intended for developers, architects, and anyone interested in the specific implementation details of the AI features and system design.

For a higher-level overview of the vision and features, please see [`README.md`](./README.md).

---

## Table of Contents

1.  [System Architecture & Data Flow](#-system-architecture--data-flow)
    -   [Core Architectural Principles](#core-architectural-principles)
    -   [Folder Structure](#folder-structure)
    -   [Data Flow Diagram](#data-flow-diagram)
2.  [Prompt Engineering Strategies](#-prompt-engineering-strategies)
    -   [**Professor Astra**: Persona & Interactive Widgets](#professor-astra-persona--interactive-widgets)
    -   [**Lesson Planner**: Structured JSON Generation](#lesson-planner-structured-json-generation)
    -   [**Adventure Architect**: Adaptive Narrative Generation](#adventure-architect-adaptive-narrative-generation)
    -   [**AI Coach**: Enabling Deeper Reasoning with `thinkingConfig`](#ai-coach-enabling-deeper-reasoning-with-thinkingconfig)
3.  [Core Services Breakdown (`/services`)](#-core-services-breakdown-services)
    -   [`geminiService.ts`](#geminiservicets)
    -   [`studentDataService.ts`](#studentdataservicets)
    -   [`chatHistoryService.ts`](#chathistoryservicets)
    -   [`archiveService.ts` & `lessonVersionService.ts`](#archiveservicets--lessonversionservicets)
4.  [UI/UX & Interactive Components](#-uiux--interactive-components)
    -   [The Interactive Widget Pipeline](#the-interactive-widget-pipeline)
    -   [Client-Side Media Generation](#client-side-media-generation)

---

## 🏛️ System Architecture & Data Flow

The application is designed as a **serverless, client-side-only single-page application (SPA)**. This architectural choice is central to its privacy-first philosophy.

### Core Architectural Principles

1.  **Zero Server-Side Logic**: The entire application runs in the user's browser. There is no backend server, no API for data persistence, and no central database.
2.  **`localStorage` as Database**: All application state (student profiles, incidents, generated content) is persisted directly in the browser's `localStorage`. The `studentDataService` and other services act as an ORM-like layer on top of `localStorage`, providing a clean API for data manipulation.
3.  **Separation of Concerns**:
    *   **Data Services (`/services`)**: Handle all data persistence and business logic.
    *   **AI Service (`geminiService.ts`)**: The single point of contact for all Google Gemini API calls.
    *   **React Components (`/components`)**: Handle all rendering and user interaction. They call the services but are unaware of the underlying implementation (e.g., they don't know data is stored in `localStorage`).

### Folder Structure

```
/
    components/
        administration/   # Admin Portal components
        common/           # Shared components (Button, Input, etc.)
        student/          # Student Portal components
        teacher/          # Teacher Portal components
    hooks/                # Custom React hooks (e.g., useRecorder)
    services/             # Core logic (AI, data, etc.)
    utils/                # Utility functions (parsing, etc.)
    App.tsx               # Main component, handles routing
    index.tsx             # Application entry point
    types.ts              # Global TypeScript type definitions
    index.html            # The single HTML page
```

### Data Flow Diagram

The application has two primary data flows that intentionally do not mix sensitive information.

**1. Local Data Flow (Privacy-First)**
This handles all persistent student and school data.

```
+----------------+      +-------------------------+      +-------------------+
| React UI       | ---> | `studentDataService.ts` | ---> | `localStorage`    |
| (e.g., Add IEP)|      | (CRUD operations)       |      | (Browser Storage) |
+----------------+      +-------------------------+      +-------------------+
```

**2. AI Request Flow (Anonymized Context)**
This handles interactions with the Gemini API. Note that PII is used to construct prompts but is not stored by Google.

```
+----------------+      +---------------------+      +---------------------+
| React UI       | ---> | `geminiService.ts`  | ---> |   Google Gemini API |
| (e.g., Ask AI) |      | (Builds Prompt)     |      |                     |
+----------------+      +---------------------+      +---------------------+
       ^                        |                            |
       |                        +----------------------------+
       +------------------------| (Streamed Response)
```

---

## 🧠 Prompt Engineering Strategies

The power of Sovereign School OS comes from sophisticated prompt engineering. We treat the LLM not just as a text generator, but as a structured data and logic engine.

### **Professor Astra**: Persona & Interactive Widgets

This is the most complex prompt, combining persona, adaptive teaching, and function-calling-like behavior through text parsing.

**System Instruction Breakdown:**

1.  **Persona Definition**:
    > `You are an educational AI assistant who embodies a specific character... YOUR CURRENT PERSONA: Character: ${hero.name}, Description: ${hero.description}. You MUST speak, act, and respond in the voice and personality of this character.`
    This sets the tone and makes the interaction engaging.

2.  **Teaching Style & Widget Formatting**:
    > `You MUST use interactive teaching methods... Use the following formats EXACTLY: [MULTIPLE_CHOICE:What is the capital of France?|Paris|London|Berlin]... [FILL_IN_BLANK:The answer is {correct_answer} here].`
    This is the core innovation. We define a simple, parsable markup language for the LLM. It's instructed to embed these "widgets" directly into its response to check for understanding.

3.  **Adaptive Difficulty**:
    > `Tailor your explanations to the student's... performance level. ${performanceInstruction}`
    Before the prompt is sent, we calculate the student's performance (`Struggling`, `Average`, `Advanced`). This injects a sub-prompt that guides the AI on the complexity of its explanation (e.g., for a 'Struggling' student: `Your persona should be exceptionally patient, warm, and encouraging... Use a relatable analogy...`).

### **Lesson Planner**: Structured JSON Generation

This feature demonstrates how to reliably get complex, nested JSON objects from the API.

**Key Prompting Techniques:**

1.  **Explicit JSON Requirement**:
    > `The output must be a valid JSON object.`

2.  **`responseSchema`**: We don't rely on the text instruction alone. The `config` object in the API call includes a detailed `responseSchema` that mirrors the `LessonPlan` interface in `types.ts`.
    ```typescript
    // In geminiService.ts
    config: {
      responseMimeType: 'application/json',
      responseSchema: {
        type: Type.OBJECT,
        properties: {
          title: { type: Type.STRING },
          lessonActivities: { 
            type: Type.ARRAY,
            items: { /* ... nested object schema ... */ }
          },
          // ... and so on for the entire LessonPlan structure
        }
      }
    }
    ```
    This forces the Gemini API to conform to our exact data structure, making the response as reliable as a traditional REST API.

### **Adventure Architect**: Adaptive Narrative Generation

This is a multi-step, stateful prompting process that creates a dynamic learning experience.

1.  **Initial Node Generation**: A simple prompt is sent with the topic, story idea, and number of stages. It's instructed to generate **only the first stage**.
2.  **Adaptive Next Node Generation**: This is the key. After the student answers a question, the `generateAdventureNextNode` function is called. Its prompt is far more complex:
    > `You are a master storyteller and adaptive educator. Your task is to generate the NEXT STAGE of an ongoing interactive adventure... Analyze the student's previous answers. If they answered a question incorrectly, the new interaction should be designed to REINFORCE that concept in a different, creative way.`

    It is provided with the full context:
    -   `Student Performance Notes`: "The student previously answered a question related to stage 2 incorrectly... Your new interaction should gently reteach that concept."
    -   `Story So Far`: A JSON string of all previous `AdventureNode` objects.

    This turns the LLM into a real-time, adaptive game master that adjusts the story based on the player's performance.

### **AI Coach**: Enabling Deeper Reasoning with `thinkingConfig`

For features that require more complex, pedagogical reasoning (like the AI Coach), we switch to a more powerful model and enable a special configuration.

```typescript
// In geminiService.ts
const responseStream = await ai.models.generateContentStream({
    model: 'gemini-2.5-pro', // Use the more powerful model
    contents,
    config: {
        systemInstruction,
        thinkingConfig: { thinkingBudget: 32768 } // Max budget for 2.5 Pro
    }
});
```
The `thinkingConfig` allows the model to use more internal tokens for reasoning before generating a response. This results in more thoughtful, in-depth, and well-structured answers suitable for professional development, at the cost of slightly higher latency.

---

## 🛠️ Core Services Breakdown (`/services`)

### `geminiService.ts`
This is the **single source of truth** for all Gemini API interactions.
-   **Abstraction**: It exposes a set of clear, domain-specific functions (e.g., `generateLessonPlan`, `askAITeacher`). Components call these functions without needing to know the underlying model names, prompt details, or API configuration.
-   **Model Selection**: It intelligently selects the right model for the job:
    -   `gemini-2.5-pro`: For complex reasoning (IEP Goals, Admin Copilot, AI Coach).
    -   `gemini-2.5-flash` / `gemini-flash-lite-latest`: For most text generation and chat tasks where speed is important.
    -   `imagen-4.0-generate-001`: For high-quality image generation.
    -   `veo-3.1-fast-generate-preview`: For video generation.
-   **Streaming & Schemas**: It handles both streaming responses (for chat) and schema-enforced JSON responses (for data generation).

### `studentDataService.ts`
This service acts as a **mock database layer** on top of `localStorage`.
-   **CRUD Operations**: Provides functions like `getStudentProfiles`, `addIncidentToStudent`, `updateIEPGoalForStudent`, etc.
-   **Initialization**: On first load, it checks if data exists in `localStorage`. If not, it seeds the application with a rich set of mock students, teachers, incidents, and academic data, ensuring a fully-featured demo experience out of the box.
-   **Data Integrity**: By centralizing all `localStorage` reads/writes here, we ensure data is stored and retrieved in a consistent, predictable manner.

### `chatHistoryService.ts`
Manages the persistence of conversations for all chat interfaces.
-   **Contextual Storage**: It uses a `contextId` (e.g., `student-s1-astra`, `admin-copilot`) to store and retrieve conversations, keeping student chats separate from teacher and admin chats.
-   **Lifecycle Management**: Provides functions to `saveConversation` (creating or updating), `getConversations` for a specific context, and `deleteConversation`.

### `archiveService.ts` & `lessonVersionService.ts`
These services provide **persistence and versioning for teacher-generated content**.
-   When a teacher generates a lesson plan or adventure, it's saved to a library via `archiveService`.
-   When a teacher modifies a lesson plan, `lessonVersionService` saves a snapshot, creating a version history they can revert to.
-   This demonstrates how AI-generated content can be treated as valuable, reusable assets within an educational ecosystem.

---

## 🎨 UI/UX & Interactive Components

### The Interactive Widget Pipeline

A key UX innovation is the ability for the AI to embed interactive elements directly into its chat responses. This is achieved through a three-step process:

1.  **Generation (Prompt Engineering)**: The `geminiService` prompts the AI to include special markup (e.g., `[MULTIPLE_CHOICE:...]`) in its text response.
2.  **Parsing (`interactiveParser.ts`)**: After the AI response is received, the `parseInteractiveContent` utility function is called. It uses a regular expression to scan the text, splitting it into an array of `ParsedContent` objects (e.g., `{ type: 'text', ... }`, `{ type: 'mcq', ... }`).
3.  **Rendering (`InteractiveWidgets.tsx`)**: The React component iterates through this array. When it encounters a `text` type, it renders plain text. When it encounters an `mcq` type, it renders the `<MultipleChoice />` React component, passing the parsed data as props.

This pipeline effectively turns a simple text-streaming response into a dynamic, interactive user interface.

### Client-Side Media Generation

To maintain the serverless architecture, all downloadable assets are created in the browser.

-   **PDFs**: When a user clicks "Download Slides" or "Export PDF," the application uses the `jspdf` library. It programmatically iterates through the lesson/adventure data, drawing text, shapes, and fetched images onto a virtual PDF canvas, which is then served to the user as a download.
-   **ZIPs**: The "Lesson Kit" feature uses `jszip`. It generates the Slides PDF, Exercises PDF, and Rubric PDF as in-memory blobs, adds them to a new ZIP archive, and then serves the complete ZIP file to the user.

This approach is highly performant and keeps all data, including generated assets, on the client side until the user explicitly chooses to save it.