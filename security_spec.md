# Security Specification — 나의 AI 프롬프트 창고

## 1. Data Invariants
- Every document in `/prompts/{promptId}` must have a valid alphanumeric/underscore/hyphen document ID (`^[a-zA-Z0-9_\-]+$`, max 128 chars).
- Public guests may read (`get`, `list`) prompts without authentication.
- Only the verified admin (`request.auth != null && request.auth.token.email == 'leejunwoo611@gmail.com' && request.auth.token.email_verified == true`) may `create`, `update`, or `delete` documents in `/prompts/{promptId}`.
- Every prompt must contain exact fields `['title', 'category', 'content', 'createdAt']` with no shadow/extra fields.
- `title` must be a non-empty string (`1 <= size <= 200`).
- `category` must be one of `['사진 합성', '스타일 변환', '캐릭터·포스터']`.
- `content` must be a non-empty string (`1 <= size <= 20000`) to support long 1,000+ character prompts.
- `createdAt` must be a Firestore timestamp equal to `request.time` on creation, and immutable on update.
- All other paths default to deny (`allow read, write: if false`).

## 2. Dirty Dozen Payloads (Expected PERMISSION_DENIED)
1. Unauthenticated guest attempting `create`, `update`, or `delete` on `/prompts/{promptId}`.
2. Non-admin authenticated user (`other@gmail.com`) attempting `create`, `update`, or `delete`.
3. Spoofed admin email with `email_verified: false` attempting `create`, `update`, or `delete`.
4. Invalid document ID containing path traversal or oversized string (`id.size() > 128`).
5. Missing required field (`title` omitted).
6. Extra ghost/shadow field (`isAdmin: true` injected).
7. Empty `title` (`""`) or oversized `title` (`> 200` chars).
8. Invalid `category` (`"무단 카테고리"`).
9. Empty `content` (`""`) or oversized `content` (`> 20000` chars).
10. Client-forged `createdAt` timestamp (`createdAt != request.time`).
11. Mutating `createdAt` during update (`incoming().createdAt != existing().createdAt`).
12. Accessing an undeclared collection (`/users/123` or `/admin/config`).
