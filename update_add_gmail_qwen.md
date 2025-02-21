```markdown
<think>
```
To integrate actual Gmail access into the `MailContent` component, we need to follow a structured approach. The provided reference file outlines the necessary steps, including authentication via OAuth 2.0, setting up NextAuth.js for session management, creating an API route to interact with the Gmail API, and updating the `MailContent.tsx` component to fetch and display real email data.

Here's how I will proceed:

1. **Install Required Packages**: We need to install `next-auth` and `googleapis` to handle authentication and Gmail API interactions.
2. **Set Up Google Cloud Project**: This involves enabling the Gmail API, creating OAuth 2.0 credentials, and configuring redirect URIs.
3. **Configure NextAuth.js**: Set up NextAuth.js with the Google provider and ensure it handles the OAuth flow and persists the access token.
4. **Create API Route**: Create an API route (`/api/gmail`) that uses the Gmail API to fetch emails and returns them in a usable format.
5. **Update `MailContent.tsx`**: Modify the component to fetch email data from the API route and display it dynamically.

Let's go through each step in detail.
```markdown
</think>
```

### Step 1: Install Required Packages

First, we need to install the necessary packages:

```bash
npm install next-auth googleapis
```

### Step 2: Set Up Google Cloud Project

Follow these steps to set up your Google Cloud project:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project or select an existing one.
3. Navigate to "APIs & Services" -> "Enabled APIs & services".
4. Click "+ ENABLE APIS AND SERVICES".
5. Search for "Gmail API" and enable it.
6. Go to "APIs & Services" -> "Credentials".
7. Click "+ CREATE CREDENTIALS" -> "OAuth client ID".
8. Choose "Web application" as the application type.
9. Add `http://localhost:3000` to "Authorized JavaScript origins".
10. Add `http://localhost:3000/api/auth/callback/google` to "Authorized redirect URIs".
11. Click "Create". You'll get a client ID and client secret.

### Step 3: Configure NextAuth.js

Create the file `src/app/api/auth/[...nextauth]/route.ts`:

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import GoogleProvider from "next-auth/providers/google";

export const authOptions = {
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      authorization: {
        params: {
          prompt: "consent",
          access_type: "offline",
          response_type: "code",
          scope: "openid email profile https://www.googleapis.com/auth/gmail.readonly"
        }
      },
    })
  ],
  callbacks: {
    async jwt({ token, account }) {
      if (account) {
        token.accessToken = account.access_token;
      }
      return token;
    },
    async session({ session, token }) {
      // @ts-ignore
      session.accessToken = token.accessToken;
      return session;
    }
  },
};

const handler = NextAuth(authOptions);

export { handler as GET, handler as POST };
```

### Step 4: Create API Route

Create the file `src/app/api/gmail/route.ts`:

```typescript
// src/app/api/gmail/route.ts
import { google } from 'googleapis';
import { getServerSession } from "next-auth/next";
import { authOptions } from "../auth/[...nextauth]/route";

export async function GET(request: Request) {
  const session = await getServerSession(authOptions);

  if (!session) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 401,
    });
  }

  // @ts-ignore
  const accessToken = session?.accessToken;

  if (!accessToken) {
    return new Response(JSON.stringify({ error: 'Access token not found' }), {
      status: 401
    });
  }

  const oauth2Client = new google.auth.OAuth2();
  oauth2Client.setCredentials({ access_token: accessToken });

  const gmail = google.gmail({ version: 'v1', auth: oauth2Client });

  try {
    const listRes = await gmail.users.messages.list({
      userId: 'me',
      maxResults: 10,
      labelIds: ['INBOX'],
    });

    const messages = listRes.data.messages;

    if (!messages || messages.length === 0) {
      return new Response(JSON.stringify({ emails: [] }), {
        status: 200,
      });
    }

    const emails = await Promise.all(
      messages.map(async (message) => {
        if (!message.id) {
          return null;
        }
        const getRes = await gmail.users.messages.get({
          userId: 'me',
          id: message.id,
          format: 'full',
        });

        const subject = getRes.data.payload?.headers?.find(
          (header) => header.name === 'Subject'
        )?.value || '';

        const sender = getRes.data.payload?.headers?.find(
          (header) => header.name === 'From'
        )?.value || '';

        const date = getRes.data.payload?.headers?.find(
          (header) => header.name === 'Date'
        )?.value || '';

        const snippet = getRes.data.snippet || '';

        return {
          id: message.id,
          subject,
          sender,
          time: date,
          snippet
        };
      })
    );

    const filteredEmails = emails.filter((email): email is Exclude<typeof email, null> => email !== null);

    return new Response(JSON.stringify({ emails: filteredEmails }), {
      status: 200,
    });
  } catch (error: any) {
    console.error('Error fetching emails:', error);
    return new Response(JSON.stringify({ error: `Failed to fetch emails: ${error.message}` }), {
      status: 500,
    });
  }
}
```

### Step 5: Update `MailContent.tsx`

Modify the `MailContent.tsx` component to fetch and display real email data:

```typescript
// src/components/MailContent.tsx
import React, { useState, useEffect } from 'react';
import { FiMail } from 'react-icons/fi';
import { useSession } from 'next-auth/react';

interface Email {
  id: string;
  subject: string;
  sender: string;
  time: string;
  snippet: string;
}

const MailItem: React.FC<{ subject: string; sender: string; time: string; snippet: string }> = ({ subject, sender, time, snippet }) => {
  return (
    <div className="flex items-start space-x-2 py-2">
      <FiMail size={20} className="text-gray-500" />
      <div>
        <div className="flex justify-between">
          <p className="font-medium">{subject}</p>
          <p className="text-sm text-gray-500">{time}</p>
        </div>
        <p className="text-sm text-gray-500">{sender}</p>
        <p className="text-sm text-gray-500">{snippet}</p>
      </div>
    </div>
  );
};

const MailContent: React.FC = () => {
  const { data: session, status } = useSession();
  const [emails, setEmails] = useState<Email[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchEmails = async () => {
      if (status === 'authenticated') {
        setLoading(true);
        setError(null);
        try {
          const response = await fetch('/api/gmail');
          if (response.ok) {
            const data = await response.json();
            setEmails(data.emails);
          } else {
            setError(`Error fetching emails: ${response.status} ${response.statusText}`);
          }
        } catch (err: any) {
          setError(`Error fetching emails: ${err.message}`);
        } finally {
          setLoading(false);
        }
      }
    };

    fetchEmails();
  }, [status]);

  if (status === 'loading') {
    return <p>Loading...</p>;
  }

  return (
    <div>
      <div className="flex justify-between items-center mb-3">
        <span className="font-medium">Inbox</span>
        <span className="text-sm text-blue-500">{emails.length} unread mes...</span>
      </div>
      {loading && <p>Loading emails...</p>}
      {error && <p>{error}</p>}
      {!loading && !error && emails.length === 0 && <p>No emails found.</p>}
      {!loading && !error && emails.length > 0 && (
        <div>
          {emails.map((email) => (
            <MailItem
              key={email.id}
              subject={email.subject}
              sender={email.sender}
              time={email.time}
              snippet={email.snippet}
            />
          ))}
        </div>
      )}
    </div>
  );
};

export default MailContent;
```

### Final Steps

1. **Environment Variables**: Create a `.env.local` file in the root of your project and add your `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `NEXTAUTH_SECRET`, and `NEXTAUTH_URL`.

   ```env
   GOOGLE_CLIENT_ID=your_google_client_id
   GOOGLE_CLIENT_SECRET=your_google_client_secret
   NEXTAUTH_SECRET=your_nextauth_secret
   NEXTAUTH_URL=http://localhost:3000
   ```

2. **Run the Development Server**:

   ```bash
   npm run dev
   ```

3. **Sign In**: Go to `http://localhost:3000` in your browser. When you click on the "Mail" section, it will trigger the authentication flow. You'll be redirected to Google to sign in and grant permission to your app. After granting permission, you'll be redirected back to your app, and it should display your emails.

This implementation provides a robust and secure way to integrate Gmail into your Next.js application, addressing authentication, API interaction, UI updates, and error handling.
