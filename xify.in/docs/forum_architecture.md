# Xify Community Forum Architecture

The **Xify Community Forum** (located at `[REPO_ROOT]/www/Home/Apps/Forum`) is a premium, secure issue-tracking and discussion platform integrated directly into the Xify ecosystem.

---

## 🏗️ Architecture Overview

The system follows a strict **Decoupled Client-Server** pattern:
- **Frontend**: High-performance Vanilla JS/CSS application.
- **Backend**: FastAPI service (`forum_main.py`) mounted at `/api/forum_dev`.
- **Database**: Integrated into the main `database.db` (SQLite) using SQLModel.

---

## 🎨 Frontend Design System

The frontend is built for speed and aesthetics, utilizing modern CSS variables and glassmorphism.

### Key Components
- **Unified Auth Strip**: Synchronized with the main portal's session management.
- **Hero Particle System**: A lightweight canvas-based background for visual flair.
- **Stats Matrix**: A dynamic `details` accordion providing a high-level view of issue distribution across categories and statuses.
- **Markdown Workflow**:
  - **EasyMDE**: Integrated Markdown editor for posts and comments.
  - **Marked.js**: High-speed Markdown-to-HTML rendering.
  - **DOMPurify**: Sanitization layer to prevent XSS.

### Interactive Features
- **File Viewer**: In-app previews for Images, PDFs, JSON, and Text files.
- **Rich Searching**: Real-time filtering by title or body with "matches in other categories" grouping.
- **Lifecycle Management**: Support for `Bug`, `Feature`, `Discussion`, and `Announcement` categories.

---

## ⚙️ Backend Logic (`forum_main.py`)

The backend is built as a robust FastAPI router with advanced security and background processing.

### 🛡️ Security & Content Validation
1. **Magic-Byte Verification**: The server inspects file headers (first 16 bytes) to verify MIME types (e.g., ensuring a `.png` actually contains PNG image data).
2. **UUID File Obfuscation**: Attachments are stored using random UUIDs to prevent path traversal and naming collisions.
3. **Admin Controls**: A hardcoded list (`ADMIN_EMAILS`) restricts critical actions like changing categories, statuses, or posting announcements.

### 📧 Communication & Notifications
The forum uses **SMTP (via GoDaddy)** for real-time engagement:
- **Status Updates**: Automatically notifies the creator when an admin resolves or closes their issue.
- **Mentions**: Custom regex detects `@user@domain.com` and triggers notification emails.
- **Background Execution**: All emails are sent via FastAPI `BackgroundTasks` to ensure zero impact on UI responsiveness.

---

## 📊 Data Model

The forum utilizes four primary tables:
| Table | Description |
| :--- | :--- |
| `ForumPost` | Primary issue data (Title, Body, Category, Status). |
| `ForumComment` | Threaded replies to posts. |
| `ForumLike` | Polymorphic table for tracking user "likes" (one per user per post). |
| `ForumAttachment` | Metadata for files linked to posts or comments. |

---

## 🚀 Deployment & Maintenance

The Forum is deployed as part of the `xify-backend` stack.

### Important Paths
- **Frontend Root**: `[REPO_ROOT]/www/Home/Apps/Forum`
- **Backend Handler**: `[REPO_ROOT]/www/Home/backend/forum_main.py`
- **Upload Storage**: `/app/data/forum_uploads` (within container volumes)

### Diagnostics
Testing the forum API logic:
```bash
# Run pytest in the forum directory
cd [REPO_ROOT]/www/Home/Apps/Forum
./venv/bin/pytest tests/
```
