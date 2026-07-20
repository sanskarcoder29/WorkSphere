# Enterprise WebAuthn Passkey Security & Subdomain Delegation

## Overview

This specification details the WebAuthn (FIDO2) integration for WorkSphere. It covers the cryptographic challenge lifecycle, biometric security enforcement, and the Relying Party (RP) origin delegation strategy required to support passkeys seamlessly across our enterprise subdomains.

---

## 1. FIDO2 Sequence Flows

### Registration Flow (Credential Creation)

1. **Initiation:** The client requests passkey registration.
2. **Challenge Generation:** The server generates a cryptographic challenge and user identity payload, returning `PublicKeyCredentialCreationOptions`.
3. **Authenticator Interaction:** The browser prompts the user for biometric verification (FaceID/TouchID). The authenticator generates a new keypair.
4. **Attestation:** The client sends the public key, signature, and challenge back to the server.
5. **Storage:** The server verifies the attestation and securely stores the `credentialID` and `publicKey` mapped to the user.

### Authentication Flow (Login)

1. **Initiation:** The client inputs their identifier (or uses discoverable credentials) to log in.
2. **Challenge Generation:** The server generates a new challenge, returning `PublicKeyCredentialRequestOptions`.
3. **Assertion:** The user authenticates biometrically. The authenticator signs the challenge using the stored private key.
4. **Verification:** The client sends the assertion to the server. The server verifies the signature against the stored `publicKey` and grants a session token.

## 2. Challenge Creation Rules

To prevent replay attacks and ensure cryptographic integrity, all WebAuthn challenges must adhere to the following strict rules:

- **Entropy:** Challenges must contain a minimum of 32 bytes of cryptographically secure random data generated via `crypto.randomBytes(32)` (Node.js) or `crypto.getRandomValues()` (Web Crypto API).
- **Encoding:** Challenges must be converted to `base64url` format without padding before being sent to the client.
- **Lifecycle (TTL):** A challenge is valid for a maximum of 5 minutes (`300000 ms`).
- **Single-Use:** Once a challenge is validated during registration or authentication, it must be immediately invalidated/deleted from the Redis cache.

## 3. RP ID & Subdomain Delegation

To allow users to register a passkey on `app.worksphere.com` and use it to log in on `admin.worksphere.com`, we utilize a top-level Relying Party ID (RP ID).

- **Configured RP ID:** `worksphere.com`
- **Rule:** The WebAuthn specification allows authenticators scoped to a root domain (`worksphere.com`) to be utilized across any valid subdomain.

### Origin Validation Code

When the server verifies the attestation or assertion, it must strictly validate the `origin` and `rpId` against an allowed list to prevent phishing and Man-in-the-Middle (MitM) attacks.

```typescript
/**
 * Validates the origin and RP ID of a WebAuthn response.
 * @param {string} clientOrigin - The origin returned from the authenticator's client data JSON.
 * @param {string} rpId - The Relying Party ID configured on the server.
 * @returns {boolean} - True if valid, throws Error if invalid.
 */
function validateWebAuthnOrigin(clientOrigin: string, rpId: string): boolean {
  const allowedOrigins = [
    "[https://worksphere.com](https://worksphere.com)",
    "[https://app.worksphere.com](https://app.worksphere.com)",
    "[https://admin.worksphere.com](https://admin.worksphere.com)",
  ];

  // 1. Validate Origin (prevents phishing from rogue domains)
  if (!allowedOrigins.includes(clientOrigin)) {
    throw new Error(`WebAuthn Error: Untrusted origin ${clientOrigin}`);
  }

  // 2. Validate RP ID (ensures the credential belongs to our top-level domain)
  const expectedRpId = "worksphere.com";
  if (rpId !== expectedRpId) {
    throw new Error(
      `WebAuthn Error: RP ID mismatch. Expected ${expectedRpId}, got ${rpId}`,
    );
  }

  return true;
}
```
