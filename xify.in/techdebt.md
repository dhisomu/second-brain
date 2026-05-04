# Technical Debt & Infrastructure Roadmap: xify.in

## 🚀 Proposed Improvements

### 1. Transition from Cookie-based Sessions to JWT (JSON Web Tokens)
**Current Method**: standard server-side sessions stored in a database or cookies.
**Proposed Method**: Stateless JWT-based authentication.

#### Rationale:
*   **Stateless Architecture**: By using JWT, the server no longer needs to query the database to verify a user's session for every request. All necessary information (user ID, role, permissions) is contained within the signed token itself.
*   **Mobile-First Performance**: JWTs are easier to manage in mobile environments and across different subdomains. Since `xify.in` users are increasingly mobile, this reduces latency and eliminates "session timeout" issues caused by cookie handling in some mobile browsers.
*   **Decoupled Frontend/Backend**: JWT makes it trivial to separate the frontend from the backend. This allows for future-proofing if the project moves towards a dedicated mobile app or a more complex single-page application (SPA) architecture.
*   **Scalability**: Stateless authentication is the industry standard for modern FastAPI/Python deployments, aligning `xify.in` with the newer architecture used in the **Logit** app.

---
*Created: May 4, 2026*
