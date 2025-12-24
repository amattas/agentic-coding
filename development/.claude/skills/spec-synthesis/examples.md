# Specification Examples

## Example: User Password Reset Feature

```markdown
# Specification: Password Reset Flow

## Problem Statement

Users who forget their passwords cannot access their accounts. We need a secure self-service password reset flow to reduce support tickets and improve user experience.

## Functional Requirements

### FR-1: Request Password Reset
User can request a password reset by entering their email address on a dedicated page.

### FR-2: Email Verification
System sends a time-limited reset link to the registered email address.

### FR-3: Token Validation
Reset links expire after 1 hour and can only be used once.

### FR-4: Password Update
User can set a new password after clicking a valid reset link.

### FR-5: Confirmation
User receives confirmation email after successful password change.

## Non-Functional Requirements

### NFR-1: Performance
- Reset email sent within 30 seconds of request
- Page load times under 2 seconds

### NFR-2: Security
- Rate limit: max 3 reset requests per email per hour
- Tokens must be cryptographically secure (256-bit)
- No indication whether email exists in system (prevent enumeration)
- Password must meet existing strength requirements

### NFR-3: Usability
- Clear error messages without revealing security details
- Mobile-responsive reset pages

## Acceptance Criteria

- [ ] User can request reset with valid email
- [ ] Reset email arrives within 30 seconds
- [ ] Link expires after 1 hour
- [ ] Link works only once
- [ ] New password must meet strength requirements
- [ ] User can log in with new password
- [ ] Old password no longer works

## Out of Scope

- SMS-based reset (future enhancement)
- Admin-initiated password resets
- Password reset via security questions

## Constraints

- Must integrate with existing email service (SendGrid)
- Must use existing password strength validator
- Cannot modify user table schema

## Open Questions

- [ ] Should we invalidate existing sessions on password change?

## Dependencies

- Email service must be operational
- User authentication system
```
