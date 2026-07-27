# InnerVoice (CESPF-NLP)

InnerVoice is a privacy-focused mental health journaling web and mobile application. It provides users with a secure, reflective space to document their thoughts while utilizing advanced language models to offer empathetic, context-aware insights and conversational support.

---

## 🚀 Features

*   **Privacy-First Journaling:** Designed from the ground up to ensure user entries remain private and secure.
*   **Cross-Platform Experience:** Seamlessly accessible on both mobile and web platforms.
*   **AI-Assisted Reflection:** Integrates large language models (LLMs) equipped with custom system prompts to provide therapeutic insights and journaling feedback.
*   **Asynchronous AI Processing:** Non-blocking background task execution ensures a smooth and responsive user interface, even during complex LLM generations.

---

## 🛠️ Tech Stack

**Frontend**
*   **Flutter:** Framework used for building natively compiled applications for mobile and web from a single codebase.

**Backend & Infrastructure**
*   **Django:** High-level Python web framework handling the core backend API, user authentication, and database management.
*   **Celery:** Distributed task queue used to handle asynchronous operations, specifically managing the delayed processing of AI responses.
*   **Valkey:** High-performance key-value data store acting as the message broker for Celery tasks.

**AI & Third-Party APIs**
*   **OpenRouter API:** Serves as the gateway to route prompts to various large language models for generating journal responses and analyzing text.

---

## 🏗️ Architecture

1.  **Client Request:** The user submits a journal entry or prompt via the Flutter application.
2.  **API Handling:** The Django backend receives the request and creates a background task.
3.  **Task Queuing:** The task is pushed to **Valkey**, which acts as the message broker.
4.  **Asynchronous Processing:** **Celery** workers pick up the task from Valkey and initiate a call to the **OpenRouter API** using specialized mental health system prompts.
5.  **Response Delivery:** Once the LLM generates a response, the Celery worker updates the database, and the result is pushed/fetched back to the Flutter frontend.

---

## 💻 Getting Started

### Prerequisites
*   [Flutter SDK](https://docs.flutter.dev/get-started/install)
*   [Python 3.x](https://www.python.org/downloads/)
*   [Valkey](https://valkey.io/) (Ensure the Valkey server is running locally or via Docker)
*   OpenRouter API Key

### Backend Setup (Django + Celery)

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/yourusername/innervoice.git](https://github.com/yourusername/innervoice.git)
    cd innervoice/backend
    ```

2.  **Set up a virtual environment:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Environment Variables:**
    Create a `.env` file in the backend root and add your configurations:
    ```env
    OPENROUTER_API_KEY=your_api_key_here
    VALKEY_URL=valkey://localhost:6379/0
    ```

5.  **Run Migrations:**
    ```bash
    python manage.py migrate
    ```

6.  **Start the Valkey server** (if not already running).

7.  **Start the Celery worker:**
    ```bash
    celery -A your_project_name worker --loglevel=info
    ```

8.  **Run the Django development server:**
    ```bash
    python manage.py runserver
    ```

### Frontend Setup (Flutter)

1.  **Navigate to the frontend directory:**
    ```bash
    cd ../frontend
    ```

2.  **Install Flutter dependencies:**
    ```bash
    flutter pub get
    ```

3.  **Configure API Endpoints:**
    Ensure your local environment configuration points to the running Django backend (e.g., `http://127.0.0.1:8000/api/`).

4.  **Run the app:**
    ```bash
    flutter run
    ```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! 

---

## 📄 License

[Specify your license here, e.g., MIT, GPL-3.0]
