# Clerk User Webhooks Payload Sync Schema Documentation

This document outlines the schema mapping between the JSON payload sent by Clerk Webhooks (specifically `user.created` and `user.updated` events) and the WorkSphere database User model defined in Prisma.

---

## 1. Webhook Event Types

WorkSphere listens for the following Clerk webhook events to keep the local database synchronized:

- `user.created`: Triggered when a new user signs up.
- `user.updated`: Triggered when a user changes their profile details (e.g., name, email, profile picture).
- `user.deleted`: Triggered when a user deletes their account.

---

## 2. Schema Mapping Reference

When a webhook is received, the payload data must be extracted and mapped to our Prisma `User` model. Below is the mapping standard for the data object.

| Clerk Payload Field                | Clerk Type       | Prisma Model Variable | Prisma Type       | Description                                   |
| :--------------------------------- | :--------------- | :-------------------- | :---------------- | :-------------------------------------------- |
| `id`                               | `String`         | `clerkId`             | `String (Unique)` | The primary identifier for the user in Clerk. |
| `email_addresses[0].email_address` | `Array[Object]`  | `email`               | `String (Unique)` | The primary email address of the user.        |
| `first_name`                       | `String \| null` | `firstName`           | `String?`         | The user's first name.                        |
| `last_name`                        | `String \| null` | `lastName`            | `String?`         | The user's last name.                         |
| `image_url`                        | `String`         | `imageUrl`            | `String?`         | URL to the user's avatar/profile picture.     |

---

## 3. Payload Extraction Example

When processing the incoming webhook, use the following pattern to extract the required variables safely, as some fields (like names) may be null depending on the user's sign-up method.

```typescript
// Inside the webhook handler
const evt = wh.verify(body, {
  "svix-id": svix_id,
  "svix-timestamp": svix_timestamp,
  "svix-signature": svix_signature,
}) as WebhookEvent;

if (evt.type === "user.created" || evt.type === "user.updated") {
  const { id, email_addresses, first_name, last_name, image_url } = evt.data;

  // Extract the primary email reliably
  const primaryEmail = email_addresses.find(
    (email) => email.id === evt.data.primary_email_address_id,
  )?.email_address;

  const prismaData = {
    clerkId: id,
    email: primaryEmail || email_addresses[0].email_address,
    firstName: first_name || "",
    lastName: last_name || "",
    imageUrl: image_url || "",
  };

  // Proceed with prisma.user.upsert(prismaData)
}
```

---

## 4. Security Requirements

- **Svix Verification:** All incoming webhooks MUST be verified using the `svix` package and the `CLERK_WEBHOOK_SECRET` environment variable to ensure the payload originated from Clerk.
- **Idempotent Operations:** Always use `prisma.user.upsert` rather than `create` to prevent unique constraint errors if a webhook is delivered multiple times or out of order.
