# 🔄 ThonHub Data Flow

This document explains how data moves inside **ThonHub** between different layers:  
**Frontend (React)** ↔ **Backend (Flask)** ↔ **Database (MongoDB)** ↔ **Async Services (Celery + Redis)** ↔ **External APIs (AI Copilot, Notifications)**.  

---

## 🏗️ High-Level Architecture

User (Web App)
⬇
Frontend (React + Tailwind)
⬇
Backend (Flask REST APIs)
⬇
MongoDB (Data Storage)
⬇
Async Queue (Redis + Celery)
⬇
External Services (AI Copilot, Email/SMS/WhatsApp APIs, Streamlit Analytics)

---

## 📌 Dataflow by Feature

### 1. 🔍 Hackathon Search & Details
1. User opens **Hackathon Search Page** on React.  
2. React → sends request `GET /api/hackathons?filter=...` to Flask backend.  
3. Flask → queries MongoDB (`hackathons` collection).  
4. MongoDB → returns list of hackathons.  
5. Backend → sends JSON response.  
6. React → displays hackathons with filter options.  
7. User clicks one → `GET /api/hackathon/:id` → returns full details.  

---

### 2. 📅 Reminders & Notifications
1. User clicks **“Set Reminder”**.  
2. React → sends `POST /api/reminders` with hackathon ID + user ID.  
3. Flask → stores reminder in MongoDB (`reminders` collection).  
4. Flask → sends async job to **Celery (via Redis)**.  
5. Celery → triggers notification at the right time (Email/SMS/WhatsApp).  
6. User receives notification.  

---

### 3. 👥 Teammate Finder
1. User opens **Teammate Finder Page**.  
2. React → calls `GET /api/teammates?skills=python,ml`.  
3. Flask → queries MongoDB (`users` collection).  
4. Ranking algorithm runs (GitHub repos, LinkedIn data, skills match).  
5. Flask → returns ranked list of potential teammates.  
6. User can **send invite** → `POST /api/invite`.  
7. Invite stored in MongoDB (`invites` collection).  

---

### 4. 💬 Chat & AI Copilot
1. User opens **Chat Page**.  
2. React → fetches previous chats (`GET /api/chat/:teamId`).  
3. Flask → returns messages from MongoDB (`chats` collection).  
4. User sends a message → `POST /api/chat`.  
5. Flask → stores message + broadcasts via WebSocket (if enabled).  
6. For **AI Copilot**:  
   - Message forwarded to AI API (OpenAI / LangChain).  
   - AI generates response → stored in `chats` collection.  
   - React updates with AI’s response.  

---

### 5. 🏆 User Profile & Badges
1. User opens **Profile Page**.  
2. React → calls `GET /api/user/:id`.  
3. Flask → fetches user data, badges, hackathon history from MongoDB.  
4. React → displays profile (resume optional).  
5. If resume uploaded → `POST /api/resume`.  
6. Flask → stores file + sends async job to **AI resume analyzer (Celery)**.  
7. Resume analysis result saved in MongoDB (`resumes` collection).  

---

### 6. 🏢 Organization Features
1. Organization logs in → `POST /api/org/login`.  
2. Org creates hackathon → `POST /api/org/hackathons`.  
3. Flask → saves hackathon details in MongoDB (`hackathons` collection).  
4. Students register → linked to org hackathon entry.  
5. Org fetches participants → `GET /api/org/hackathon/:id/participants`.  
6. Streamlit dashboard queries MongoDB → shows participation analytics.  
7. Org can filter students by skills, history, badges (for placements).  

---

## 📊 Collections in MongoDB

- **users** → student/org profiles (skills, badges, history, resume)  
- **hackathons** → hackathon details (org, date, teams, rules)  
- **reminders** → reminders for users (linked to hackathon + notification type)  
- **invites** → team invitations & requests  
- **teams** → team details, members, projects  
- **projects** → submissions linked to hackathons & teams  
- **chats** → chat messages (team chats + AI copilot messages)  
- **resumes** → uploaded resumes + AI analysis  

---

## 🔄 Async Jobs (Celery + Redis)

- **Notifications:** schedule & send reminders via Email/WhatsApp/SMS.  
- **Resume Analysis:** run AI-based parsing & skill extraction.  
- **Analytics Jobs:** pre-compute hackathon stats for dashboards.  

---

## 🏁 Dataflow Phases

- **Phase 1:** Hackathon Search + Basic APIs + MongoDB setup  
- **Phase 2:** Teammate Finder + Profile System  
- **Phase 3:** Chat + AI Copilot  
- **Phase 4:** Notifications (Reminders via Celery/Redis)  
- **Phase 5:** Organization Tools + Analytics (Streamlit)  

---

✅ This dataflow gives contributors a clear understanding of **how data travels** and how **features interact** in ThonHub.