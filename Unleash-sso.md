Great! Using Docker containers for Keycloak and Unleash is a clean and portable approach. Here's how to adapt the SSO setup for a Dockerized environment:

**1. Docker Compose Setup**

   Create a `docker-compose.yml` file to define your services:

   ```yaml
   version: '3.8'

   services:
     keycloak:
       image: jboss/keycloak:latest 
       ports:
         - "8080:8080"
       environment:
         - KEYCLOAK_USER=admin
         - KEYCLOAK_PASSWORD=admin 
         - KEYCLOAK_IMPORT=/tmp/realm-export.json
       volumes:
         - ./realm-export.json:/tmp/realm-export.json

     unleash:
       image: unleash/unleash:latest 
       ports:
         - "4242:4242"
       depends_on:
         - keycloak  # Start Keycloak before Unleash
       environment:
         - UNLEASH_DATABASE_URL=postgres://unleash:unleash@postgres:5432/unleash
         - UNLEASH_ADMIN_AUTHENTICATION=keycloak
         - UNLEASH_KEYCLOAK_URL=http://keycloak:8080/auth
         - UNLEASH_KEYCLOAK_REALM=unleash-demo
         - UNLEASH_KEYCLOAK_CLIENT_ID=unleash

   volumes:
     db-data:
   ```

   **Explanation:**

   * **keycloak:** 
      * Uses the official Keycloak image.
      * Exposes port 8080.
      * Sets admin credentials.
      * **Important:** Imports a realm configuration (`realm-export.json`) for the "unleash-demo" realm (more on this in step 2).
   * **unleash:**
      * Uses the official Unleash image.
      * Exposes port 4242.
      * **depends_on:** Ensures Keycloak starts before Unleash.
      * **environment:**
        * Configures the database connection (assuming you have a PostgreSQL container running - you might need to add it to your `docker-compose.yml`).
        * Enables Keycloak authentication.
        * Provides Keycloak configuration details. **Crucially, use the container name `keycloak` as the hostname for the `UNLEASH_KEYCLOAK_URL`.**

**2. Keycloak Realm Export (realm-export.json)**

   * **Start Keycloak:** Run `docker-compose up -d keycloak` to start your Keycloak container.
   * **Configure the Realm:**
      * Access the Keycloak admin console (http://localhost:8080/auth) and follow the steps outlined in section 1 of the previous instructions to:
          * Create the "unleash-demo" realm.
          * Create the "unleash" client.
          * Create a test user.
   * **Export the Realm:** From the Keycloak admin console, go to Realm Settings -> Export and export the "unleash-demo" realm as a JSON file (`realm-export.json`).
   * **Place the Export:** Put the `realm-export.json` file in the same directory as your `docker-compose.yml` file. 

**3. Start the Containers**

   ```bash
   docker-compose up -d
   ```

**4. Access Unleash**

   * Once the containers are up, access Unleash at http://localhost:4242.
   * You should be redirected to Keycloak for login, and after successful authentication, you'll be redirected back to your Unleash dashboard.

**Key Points & Considerations:**

* **Docker Networking:** Docker Compose sets up a network for your containers, allowing them to communicate using their service names (e.g., `keycloak`) as hostnames.
* **Database:** Make sure you have a database (like PostgreSQL) running and accessible to your Unleash container. 
* **Keycloak Configuration:** Importing the realm configuration (`realm-export.json`) during container startup ensures that Keycloak has the correct settings for SSO with Unleash.

This setup provides a more robust and reproducible way to demonstrate SSO between Keycloak and Unleash locally. Feel free to ask if you have any more questions!


You're in luck! Here's an example `realm-export.json` file that you can adapt for your setup. Remember to modify the client secret if needed (explained below). 

```json
{
  "realm": "unleash-demo",
  "enabled": true,
  "sslRequired": "external",
  "registrationAllowed": false,
  "clients": [
    {
      "id": "unleash",
      "clientId": "unleash",
      "name": "Unleash",
      "rootUrl": "http://localhost:4242", 
      "baseUrl": "http://localhost:4242", 
      "enabled": true,
      "protocol": "openid-connect",
      "publicClient": false,
      "bearerOnly": false,
      "directAccessGrantsEnabled": false,
      "implicitFlowEnabled": true,
      "standardFlowEnabled": true,
      "serviceAccountsEnabled": false,
      "redirectUris": [
        "http://localhost:4242/*",
        "http://localhost:4242/api/admin/login/keycloak/callback/*" 
      ],
      "webOrigins": [
        "http://localhost:4242" 
      ],
      "attributes": {
        "saml.assertion.signature.post.encode.url": "false",
        "saml.assertion.signature.deflate.encode.url": "false",
        "saml.onetimeuse.condition": "false",
        "saml.client.signature.required": "false",
        "saml.force.post.binding": "false"
      }
    }
  ]
}
```

**Important Notes:**

* **Client Secret (Optional):** For enhanced security in a production environment, you should generate and use a client secret for the "unleash" client in Keycloak.  Add the following to the "unleash" client configuration in the `realm-export.json`:

   ```json
   "secret": "YOUR_GENERATED_CLIENT_SECRET", 
   ```

   Then, in your `docker-compose.yml`, set the `UNLEASH_KEYCLOAK_CLIENT_SECRET` environment variable for the Unleash container:

   ```yaml
   environment:
     # ... other variables ...
     - UNLEASH_KEYCLOAK_CLIENT_SECRET=YOUR_GENERATED_CLIENT_SECRET
   ```

* **Redirect URIs and Web Origins:**  The `redirectUris` and `webOrigins` values in the `realm-export.json` are configured to match the typical Unleash Docker container setup (using port 4242). If you have a different port or URL structure, adjust these values accordingly.

**Unleash Free Tier Compatibility**

Yes, this setup is fully compatible with the Unleash free tier! You can use Keycloak for SSO with both the open-source Unleash and the free tier of Unleash hosted by the Unleash team. The configuration principles remain the same.

Let me know if you have any more questions!
