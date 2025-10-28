# Sovereign School OS
### Education is Broken. This is the Fix.

For too long, education has been trapped in a crisis of time. Teachers are buried under administrative work, forced to be content creators, data analysts, and curriculum designers, leaving them with little time to actually *teach*. Students are locked into one-size-fits-all lessons, their individual sparks of curiosity dimming in the face of standardized content. And parents are left navigating a maze of disconnected, insecure digital tools.

**Sovereign School OS is the AI-native operating system built to solve this crisis.** We don't just add AI to the classroom; we rebuild the classroom *around* AI, creating a privacy-first, hyper-efficient, and deeply engaging ecosystem that gives time back to teachers and a universe of learning to students.

> Imagine a classroom where every student has a personal tutor—an AI Albert Einstein for physics, an AI Marie Curie for chemistry—that adapts to their learning style in real-time. Imagine a teacher who can generate an entire week's worth of interactive lessons, complete with slide decks, videos, and worksheets, in the time it takes to drink their morning coffee.
>
> **Stop imagining. Start exploring.**

---

## ✨ The Student Universe: Where Learning is an Adventure

We've dismantled the passive classroom and replaced it with a personalized, gamified universe where every student is the hero of their own educational journey.

### 🚀 Your Personal AI Mentor
-   **Choose Your Guide**: Your learning companion is a "Hero"—a historical or cultural icon like **Hua Mulan**, **Socrates**, or **Zumbi dos Palmares**. This AI mentor, "Professor Astra," adopts their personality to make every interaction unforgettable.
-   **Learn by Talking**: Ask any question and get instant, encouraging, grade-appropriate answers in the voice of your hero.
-   **Mastery Through Interaction**: The AI doesn't just talk; it tests. It seamlessly embeds interactive quizzes, fill-in-the-blanks, and matching games directly into the conversation to ensure you truly understand.

### 🗺️ The Adaptive Quest Engine
-   **Dynamic Storytelling**: Embark on epic, story-driven lessons where the narrative adapts to *you*. Get a question wrong, and the AI creatively rewrites the next stage of the story to reteach the concept from a new angle.
-   **Multi-Modal Worlds**: Experience lessons as interactive stories with AI-generated illustrations (`Imagen`) or as animated short films with AI-generated video clips (`Veo`).
-   **Your Story, Your Creation**: When your adventure is complete, export it as a personalized PDF "comic book"—a trophy of your learning journey.

### 🏛️ The Learning Hub: Your AI-Powered Campus
A suite of astonishingly powerful, subject-specific tools that turn "I don't know" into "I can find out."
-   **STEM Wing**: Instantly solve complex math problems, get line-by-line code explanations, and run "virtual experiments" in the Science Lab.
-   **Humanities Hall**: Debate an AI Socrates, practice Spanish with a conversational language tutor, or explore any historical era with an expert AI guide.
-   **Creative Arts Center**: Become an artist and filmmaker. Generate stunning images, edit photos, and create video clips with simple text prompts.

---

## 👩‍🏫 The Teacher Command Center: Your AI Co-Pilot

We believe a teacher's highest purpose is to inspire, not to administrate. The teacher portal is a powerful command center designed to automate the mundane and amplify creativity.

-   **🔥 The 5-Minute Lesson Plan**: From a simple text paste, dictation, or PDF upload, our AI generates a comprehensive, multi-part lesson plan, complete with objectives, activities, and differentiation strategies.
-   **📦 One-Click Content Factory**: With a single button press, generate a complete **Lesson Kit** from your plan—a `.zip` file containing a slide deck, student worksheets, and a grading rubric, all created by AI.
-   **🎭 The Adventure Architect**: Become a game designer. Write a simple story prompt (e.g., "A mission to the Amazon to learn about ecosystems") and let the AI build a dynamic, multi-stage interactive adventure for your students.
-   **🎬 The Media Hub**: Direct your own educational content. Generate bespoke illustrations (`Imagen`) and video clips (`Veo`) for your lessons without needing any artistic skill.
-   **🧠 The AI Coach**: Go beyond lesson planning. Use "Thinking Mode" to have deep, pedagogical discussions with "Professor Astra" about teaching strategies, classroom management, and professional growth.
-   **✉️ The Communications Supercharger**: Draft professional parent emails from a few bullet points. Translate messages in real-time. Get instant AI summaries of long conversation threads. Reclaim your evenings.

---

## 🏛️ The Admin Holodeck: Data-Driven Leadership

The admin portal provides the 30,000-foot view, transforming raw data into actionable insights for a healthier, more equitable school.

-   **🤖 The AI Copilot**: A conversational interface for the big questions. Ask "Generate a fairness report on all logged incidents" or "Identify students who may need support this month," and get a comprehensive, data-backed analysis in seconds.
-   **⚖️ The Equity Dashboard**: Proactively ensure fairness. The AI provides a neutral analysis of school-wide incident patterns, helping you spot and address potential biases before they become problems.
-   **❤️ The Support Hub**: See who needs help before they fall behind. The AI analyzes academic, behavioral, and attendance data to proactively recommend students who might benefit from a check-in, along with suggested actions.

---

## ⚙️ How We Built the Future (And Kept it Private)

Our revolutionary approach is built on three core principles.

-   **🔒 Absolute Privacy. No Compromises.** Sovereign School OS is **serverless**. All student and school data is stored **exclusively in the browser's `localStorage`**. No sensitive information is ever sent to a server or stored in the cloud. This isn't just a feature; it's our foundational promise.
-   **🧠 The Centralized AI Brain**: All AI capabilities are routed through a single, powerful service (`geminiService.ts`). This allows us to use the best model for every task—`gemini-2.5-pro` for complex reasoning, `imagen-4.0-generate-001` for art, `veo-3.1-fast-generate-preview` for video—while keeping the codebase clean and easy to upgrade.
-   **✅ Unprecedented Reliability**: We use Gemini's `responseSchema` feature to force the AI to return perfectly structured JSON. This makes the AI's output as predictable and reliable as a traditional API, eliminating the guesswork and ensuring a smooth user experience.

---

## 🛠️ Technology Stack

-   **Frontend**: **React 19** with **TypeScript**
-   **Styling**: **Tailwind CSS**
-   **AI Engine**:
    -   **Reasoning & Text**: `gemini-2.5-pro`, `gemini-2.5-flash`
    -   **Image Generation**: `imagen-4.0-generate-001`
    -   **Image Editing**: `gemini-2.5-flash-image`
    -   **Video Generation**: `veo-3.1-fast-generate-preview`
    -   **Audio & Speech**: `gemini-2.5-flash` (Transcription) & `gemini-2.5-flash-preview-tts` (Text-to-Speech)
-   **Client-Side Documents**: `jspdf`, `jszip`, and `pdf.js` for in-browser PDF/ZIP creation and parsing.

---

## 🚀 Getting Started

This application is designed to run in an environment where a Google Gemini API key is provided.

1.  **Launch**: Open the `index.html` file in a modern web browser.
2.  **Login**: Use the credentials provided in `credentials.md` to access the Teacher, Student, or Admin portals.
3.  **Explore**: The app is pre-populated with rich demo data, so you can start exploring all features immediately!