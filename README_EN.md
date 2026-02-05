# Perphorum - System Documentation

A project implementing a social portal for perfume enthusiasts, inspired by platforms like **Filmweb** and **Fragrantica**. The application allows users to browse a fragrance database, write reviews, create personal collections, and interact with other users.

---

## Authors
*   **Maciej Mikołajek** - [GitHub](https://github.com/maciusk8)
*   **Mateusz Wróbel** - [GitHub](https://github.com/mateuszwroobel)

---

## System Architecture

The system is built using a **Client-Server** architecture (Single Page Application + REST API). The codebase is divided into two independent parts: `frontend` (React) and `backend` (Spring Boot, Java 25).

### Technology Stack

#### Frontend (Presentation Layer)
*   **React 18**: UI library (built using Functional Components and Hooks).
*   **TypeScript**: Provides static typing and code safety.
*   **React Router**: Handles client-side routing (SPA).
*   **React Bootstrap**: UI component system styled for a "Premium" aesthetic (gold accents, serif fonts).
*   **Vite**: Build tool enabling fast development.

#### Backend (Business Logic Layer)
*   **Spring Boot 3**: Application framework (REST API, Dependency Injection).
*   **Spring Security**: Security and authentication (Stateless).
*   **JJWT (Java JSON Web Token)**: JWT standard implementation for token generation and validation.
*   **Spring Data JPA (Hibernate)**: Data access layer (ORM).
*   **H2 Database**: In-memory database (for easy demonstration and testing), configured to seed with initial data.
*   **Lombok**: Library reducing boilerplate code (automatic getters, setters, builders).
*   **Jackson**: JSON data serialization and deserialization.

---

## Feature Implementation (Requirements)

The following table shows how key functionalities required for a modern social portal about this topic were implemented.

| Module | Implementation Description |
| :--- | :--- |
| **Perfume Catalog** | Database containing detailed information: Brand, Name, Fragrance Family, Notes, Description, Image. Advanced filtering by gender, brand and ingredients. |
| **Rating System** | Users can rate perfumes (1-5 stars). System automatically calculates average rating. |
| **Reviews and Discussions** | Ability to add detailed text reviews. Other users can comment and like them. |
| **User Profiles** | Each user has a public profile displaying their activity. |
| **Virtual Shelves** | "I Own" (Collection) and "I Want" (Wishlist) functionality, visible on user profile. |
| **Community** | Friends system. Activity feed (`FriendsPage`) aggregating friends' reviews. |
| **Rankings** | Dynamically generated "Top Perfumes" list sorted by average rating. |
| **Security** | Registration, login (JWT/BCrypt), route protection (User/Admin), protection against attacks (e.g., XSS, infinite recursion in JSON). |
| **Monetization** | The service enables commercialization through sponsored articles displayed on the homepage (Hero Section). |


---

## Gallery

### Homepage
![Homepage](screenshots/home.png)

### Perfume Ranking
![Ranking](screenshots/ranking.png)

### Perfume Details
![Perfume Page](screenshots/strona%20perfum.png)

### Catalog (Men's)
![Men's Catalog](screenshots/meskie.png)

### Activity Feed
![Feed](screenshots/feed.png)

### User Profile
![Your Profile](screenshots/moj%20profil.png)

---

## Data Model and Database

The application uses a relational data model. Key entities:

*   **`AppUser`**: User, their role (USER/ADMIN), password (hash) and social relationships.
*   **`Perfume`**: Product, contains metadata (brand, family, notes) and relationship to reviews.
*   **`Review`**: User's opinion linking `AppUser` with `Perfume`.
*   **`Comment`**: Comment under a review.


**Data Seeding**:
During backend startup, the `DataLoader.java` class checks the database state. If it's empty (or nearly empty), it automatically loads several hundred perfume records from the `perfumes.json` file and creates sample users and reviews, so the system "comes alive" from the first run.

*Perfume data sourced from: [HuggingFace - Perfume Dataset](https://huggingface.co/datasets/doevent/perfume/tree/main?not-for-all-audiences=true).*

---

## Running the Project

The project consists of two independent servers that must run simultaneously:

*   **Backend (Spring Boot)**: Runs on port `8080`.
*   **Frontend (Vite/React)**: Runs on port `5175`.

---

## Test Accounts

For easy verification, pre-configured test accounts with assigned roles are available. They are also listed in a table on the login page.

| Login | Password | Role | Description |
| :--- | :--- | :--- | :--- |
| **maciej** | `maciej123` | USER | Regular user with review history. |
| **mateusz** | `mateusz123` | USER | Regular user. |
| **prowadzacy** | `admin123` | **ADMIN** | Account with content deletion permissions. |

---

## API Documentation

All backend communication is done through REST API with `/api/` prefix.

### AuthController (`/api/auth`)
*Session and user account management*

| Method | Endpoint | Parameters (Body) | Description |
| :--- | :--- | :--- | :--- |
| **POST** | `/register` | `{ username, email, password }` | Register new user. Returns JWT token on success. |
| **POST** | `/login` | `{ username, password }` | Login. Returns `AuthResponse` object containing JWT token, role and ID. |

### PerfumeController (`/api/perfumes`)
*Browsing perfume catalog*

| Method | Endpoint | Parameters (Query/Path) | Description |
| :--- | :--- | :--- | :--- |
| **GET** | `/` | - | Retrieves list of all perfumes. |
| **GET** | `/search` | `?text={query}` | Search perfumes by name (case-insensitive). |
| **GET** | `/{id}` | `id` (Path) | Detailed data for specific fragrance. |
| **GET** | `/brand/{brandName}` | `brandName` (Path) | List of perfumes from given brand. |
| **GET** | `/gender/{gender}` | `gender` (Path) | Filter by gender (`Male`, `Female`, `Unisex`). |
| **GET** | `/ingredient` | `?name={query}` | Search perfumes containing given ingredient. |

### ReviewController (`/api/reviews`)
*Review and rating system*

| Method | Endpoint | Parameters | Description |
| :--- | :--- | :--- | :--- |
| **POST** | `/` | `{ userId, perfumeId, text, rating }` | Add new review (one per product per user allowed). |
| **GET** | `/perfume/{id}` | `id` (Path) | All reviews for given perfume. |
| **GET** | `/user/{id}` | `id` (Path) | All reviews written by specific user. |
| **GET** | `/recent` | `?limit=6` | Recently added reviews (for homepage). |
| **GET** | `/feed` | `?userId={id}` | Activity feed of user's friends. |
| **POST** | `/{id}/like` | `?userId={id}` | Toggle like (like/unlike) review. |
| **DELETE** | `/{id}` | `id` (Path) | Delete review (Requires permissions). |

### CommentController (`/api/comments`)
*Commenting on reviews*

| Method | Endpoint | Parameters | Description |
| :--- | :--- | :--- | :--- |
| **POST** | `/` | `{ userId, reviewId, text }` | Add comment under review. |
| **DELETE** | `/{id}` | `id` (Path) | Delete comment. |

### AppUserController (`/api/users`)
*Social profile and lists*

| Method | Endpoint | Parameters | Description |
| :--- | :--- | :--- | :--- |
| **GET** | `/{id}` | `id` (Path) | Retrieves public profile data (name, ID). |
| **GET** | `/search` | `?query={username}` | Search users by name. |
| **GET** | `/{id}/wishlist` | `id` (Path) | Retrieves "I Want" perfume list. |
| **GET** | `/{id}/owned` | `id` (Path) | Retrieves "I Own" perfume list. |
| **POST** | `/wishlist` | `?userId=X&perfumeId=Y` | Add/Remove from wishlist (Toggle). |
| **POST** | `/owned` | `?userId=X&perfumeId=Y` | Add/Remove from collection (Toggle). |
| **GET** | `/{id}/friends` | `id` (Path) | User's friends list. |
| **POST** | `/friend` | `?myId=X&targetId=Y` | Add/Remove user from friends (Toggle). |

---

## Folder Structure

```
perphorum/
├── backend/perphorum/          # Spring Boot Application
│   ├── src/main/java/...       # Controllers, Model, Services
│   └── src/main/resources/...  # Configuration, perfumes.json
├── frontend/                   # React Application
│   ├── src/
│   │   ├── components/         # Reusable components
│   │   ├── pages/              # Views (Auth, Gender, Brand)
│   │   ├── services/           # API communication
│   │   ├── context/            # Context API (Auth)
│   │   └── [FeaturePages]/     # Page modules (Home, Profile, Product...)
│   └── package.json
├── screenshots/                # Screenshots for documentation
└── README.md                   # This document
```
