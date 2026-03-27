# Book Rating REST API

A Rails JSON API for managing authors, books, users, and reader reviews. Reviews are polymorphic, so the same endpoint pattern and validation rules support both books and authors.

The project began as a coding exercise and retains that focused scope. The repository implements the core acceptance criteria plus filtering, sorting, description normalization, fictional-profanity validation, author reviews, and stored book averages.

## Domain model

```text
Author ── has many Books
   │                    
   └── Reviews <── User ──> Reviews ── Book
          polymorphic reviewable
```

Each review:

- belongs to a user and either a book or author
- requires an integer rating from 1 to 5
- accepts an optional description up to 300 characters
- prevents the same user from reviewing the same resource twice
- normalizes blank descriptions to `nil`
- rejects a small fictional-profanity list from the original exercise

Book averages are recalculated after a review is created. Author reviews do not update an average-rating column.

## API surface

All routes are under `/api`.

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/authors` | List authors |
| `POST` | `/authors` | Create an author |
| `GET` | `/authors/:id` | Retrieve an author |
| `PATCH` | `/authors/:id` | Update an author |
| `GET` | `/books` | List books |
| `POST` | `/books` | Create a book |
| `GET` | `/books/:id` | Retrieve a book |
| `PATCH` | `/books/:id` | Update a book |
| `GET` | `/users` | List users |
| `POST` | `/users` | Create a user |
| `GET` | `/books/:book_id/reviews` | List or filter book reviews |
| `POST` | `/books/:book_id/reviews` | Review a book |
| `GET` | `/authors/:author_id/reviews` | List or filter author reviews |
| `POST` | `/authors/:author_id/reviews` | Review an author |

## Review examples

### Create a book review

```http
POST /api/books/12/reviews
Content-Type: application/json

{
  "user_id": 7,
  "rating": 4,
  "description": "Clear and practical."
}
```

### Filter and sort reviews

```http
GET /api/books/12/reviews?rating=4&description_only=true&sort_by=created_at&order=desc
```

Allowed sort fields are `rating`, `created_at`, and `updated_at`; directions are `asc` and `desc`. The service uses a whitelist rather than interpolating arbitrary column names.

Invalid submissions return an `errors` array. For example, a rating outside the accepted range is rejected instead of being coerced.

## Project structure

```text
app/
├── controllers/api/       # Author, book, user, and review endpoints
├── models/                # Associations and review rules
├── models/concerns/       # Shared Reviewable association
└── services/              # Review filtering and sorting
spec/
├── models/
├── requests/api/
└── services/
```

## Local setup

### Prerequisites

- Ruby 2.7.3
- Bundler 2.2.x
- PostgreSQL

```bash
git clone https://github.com/alirazawatoo/book-rating-rest-api.git
cd book-rating-rest-api
bundle install
bin/rails db:setup
bin/rails server
```

## Tests

```bash
bundle exec rspec
```

The tracked suite covers models, API requests, error responses, duplicate reviews, filtering, sorting, author reviews, and the filtering service. The suite was inspected but not executed during the portfolio review.

## Design notes and boundaries

- Polymorphic reviews keep the book and author endpoints consistent.
- Sorting uses an allowlist and hash ordering to avoid query-field injection.
- User identity is accepted as `user_id`; there is no authentication or authorization layer.
- The uniqueness rule is expressed in Rails validation and should also be backed by a database constraint for concurrent requests.
- Stored book averages are updated after creation and are not shown here to handle later review edits or deletions.
- Pagination is not implemented for collection endpoints.

## Next improvements

1. Derive the reviewer from authentication rather than request parameters.
2. Add a composite database uniqueness constraint for review ownership.
3. Make aggregate ratings resilient to create, update, and delete operations.
4. Add pagination and consistent JSON serializers.
