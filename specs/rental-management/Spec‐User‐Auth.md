## ADDED Requirements

### Requirement: User registration with email and password
The system SHALL allow a new user to create an account by providing an email address and a password. The password MUST be at least 8 characters and include at least one uppercase letter, one lowercase letter, and one digit. The system SHALL send a verification email before activating the account.

#### Scenario: Successful registration
- **WHEN** a visitor submits a valid email and a compliant password
- **THEN** the system creates the account in "pending verification" status and sends a verification email with a unique link

#### Scenario: Registration with existing email
- **WHEN** a visitor submits an email already associated with an existing account
- **THEN** the system displays an error message without revealing whether the email exists (security best practice)

#### Scenario: Registration with weak password
- **WHEN** a visitor submits a password that does not meet the complexity requirements
- **THEN** the system rejects the submission and displays the specific password rules that are not met

### Requirement: Email verification
The system SHALL require email verification before allowing a user to access the application. The verification link MUST expire after 24 hours.

#### Scenario: Successful email verification
- **WHEN** a user clicks the verification link within 24 hours of registration
- **THEN** the system activates the account and redirects the user to the login page

#### Scenario: Expired verification link
- **WHEN** a user clicks a verification link after 24 hours
- **THEN** the system displays an expiration message and offers to resend a new verification email

### Requirement: User login with email and password
The system SHALL allow a registered and verified user to log in with their email and password. The system SHALL create a secure session upon successful authentication.

#### Scenario: Successful login
- **WHEN** a verified user submits correct email and password
- **THEN** the system creates a session and redirects the user to the dashboard

#### Scenario: Login with incorrect credentials
- **WHEN** a user submits an incorrect email or password
- **THEN** the system displays a generic "invalid credentials" error without specifying which field is wrong

#### Scenario: Login attempt on unverified account
- **WHEN** a user with an unverified email attempts to log in
- **THEN** the system blocks login and offers to resend the verification email

### Requirement: OAuth login with Google
The system SHALL allow users to register and log in using their Google account via OAuth 2.0. If no account exists for the Google email, the system SHALL create one automatically (verified).

#### Scenario: First login with Google
- **WHEN** a visitor authenticates via Google and no account exists for that email
- **THEN** the system creates a verified account linked to the Google profile and redirects to the dashboard

#### Scenario: Returning login with Google
- **WHEN** a user with an existing Google-linked account authenticates via Google
- **THEN** the system creates a session and redirects to the dashboard

### Requirement: Password reset
The system SHALL allow a user to reset their password via a secure email link. The reset link MUST expire after 1 hour.

#### Scenario: Successful password reset request
- **WHEN** a user requests a password reset with a registered email
- **THEN** the system sends an email with a unique, time-limited reset link

#### Scenario: Successful password change
- **WHEN** a user clicks a valid reset link and submits a new compliant password
- **THEN** the system updates the password, invalidates all existing sessions, and redirects to login

### Requirement: User profile management
The system SHALL allow an authenticated user to view and update their profile information (name, email, phone number). Changing the email MUST trigger a new verification process.

#### Scenario: Update profile name
- **WHEN** an authenticated user changes their display name
- **THEN** the system saves the new name and confirms the update

#### Scenario: Update profile email
- **WHEN** an authenticated user changes their email address
- **THEN** the system sends a verification email to the new address and keeps the old email active until the new one is verified

### Requirement: Secure session management
The system SHALL manage user sessions with JWT tokens. Sessions MUST expire after 7 days of inactivity. The system SHALL support concurrent sessions across multiple devices.

#### Scenario: Session expiration
- **WHEN** a user has been inactive for more than 7 days
- **THEN** the system invalidates the session and redirects to the login page on the next request

#### Scenario: Active session refresh
- **WHEN** a user makes a request with a valid but soon-to-expire token
- **THEN** the system silently refreshes the token and continues the request
