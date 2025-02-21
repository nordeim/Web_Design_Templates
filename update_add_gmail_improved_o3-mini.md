https://chatgpt.com/share/67b7cae3-21ec-8000-a543-3d0fb946399b

Below is an improved implementation that integrates Gmail access into your Next.js app without copying the reference verbatim. In this solution, we:

1. **Set up OAuth with NextAuth** – configuring the Google provider for Gmail read access.
2. **Create a secure API route** – using the Gmail API to fetch email data.
3. **Enhance the MailContent UI** – fetching live emails and handling loading and error states.

Below are the key parts of the implementation.

---

### 1. NextAuth Configuration

Create the file `src/app/api/auth/[...nextauth]/route.ts`:

```typescript
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
          scope:
            "openid email profile https://www.googleapis.com/auth/gmail.readonly",
        },
      },
    }),
  ],
  callbacks: {
    async jwt({ token, account }) {
      if (account) {
        token.accessToken = account.access_token;
      }
      return token;
    },
    async session({ session, token }) {
      session.accessToken = token.accessToken;
      return session;
    },
  },
};

const handler = NextAuth(authOptions);

export { handler as GET, handler as POST };
```

---

### 2. API Route for Gmail Access

Create the file `src/app/api/gmail/route.ts`:

```typescript
import { google } from "googleapis";
import { getServerSession } from "next-auth/next";
import { authOptions } from "../auth/[...nextauth]/route";

export async function GET() {
  const session = await getServerSession(authOptions);
  if (!session || !session.accessToken) {
    return new Response(JSON.stringify({ error: "Unauthorized" }), {
      status: 401,
    });
  }

  const oauth2Client = new google.auth.OAuth2();
  oauth2Client.setCredentials({ access_token: session.accessToken as string });

  const gmail = google.gmail({ version: "v1", auth: oauth2Client });
  try {
    const listResponse = await gmail.users.messages.list({
      userId: "me",
      maxResults: 10,
      labelIds: ["INBOX"],
    });

    const messages = listResponse.data.messages;
    if (!messages || messages.length === 0) {
      return new Response(JSON.stringify({ emails: [] }), { status: 200 });
    }

    // Retrieve detailed information for each message.
    const emails = await Promise.all(
      messages.map(async (msg) => {
        if (!msg.id) return null;
        const messageDetail = await gmail.users.messages.get({
          userId: "me",
          id: msg.id,
          format: "full",
        });

        const headers = messageDetail.data.payload?.headers || [];
        const subject = headers.find((h) => h.name === "Subject")?.value || "";
        const sender = headers.find((h) => h.name === "From")?.value || "";
        const date = headers.find((h) => h.name === "Date")?.value || "";
        const snippet = messageDetail.data.snippet || "";

        return {
          id: msg.id,
          subject,
          sender,
          time: date,
          snippet,
        };
      })
    );

    const filteredEmails = emails.filter((email): email is NonNullable<typeof email> => email !== null);
    return new Response(JSON.stringify({ emails: filteredEmails }), {
      status: 200,
    });
  } catch (error: any) {
    console.error("Error fetching emails:", error);
    return new Response(
      JSON.stringify({ error: `Failed to fetch emails: ${error.message}` }),
      { status: 500 }
    );
  }
}
```

---

### 3. Enhanced MailContent Component

Update your `src/components/MailContent.tsx` to fetch and render live email data:

```tsx
import React, { useState, useEffect } from "react";
import { FiMail } from "react-icons/fi";
import { useSession } from "next-auth/react";

interface Email {
  id: string;
  subject: string;
  sender: string;
  time: string;
  snippet: string;
}

const MailItem: React.FC<Email> = ({ subject, sender, time, snippet }) => (
  <div className="flex items-start space-x-2 py-2 border-b border-gray-200">
    <FiMail size={20} className="text-gray-500" />
    <div className="flex-grow">
      <div className="flex justify-between">
        <p className="font-medium">{subject}</p>
        <p className="text-sm text-gray-500">{time}</p>
      </div>
      <p className="text-sm text-gray-500">{sender}</p>
      <p className="text-sm text-gray-400">{snippet}</p>
    </div>
  </div>
);

const MailContent: React.FC = () => {
  const { data: session, status } = useSession();
  const [emails, setEmails] = useState<Email[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchEmails = async () => {
      if (status !== "authenticated") return;
      setLoading(true);
      setError(null);
      try {
        const response = await fetch("/api/gmail");
        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }
        const data = await response.json();
        setEmails(data.emails);
      } catch (err: any) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchEmails();
  }, [status]);

  if (status === "loading") return <p>Loading session...</p>;

  return (
    <div>
      <div className="flex justify-between items-center mb-3">
        <span className="font-medium">Inbox</span>
        <span className="text-sm text-blue-500">
          {emails.length} unread messages
        </span>
      </div>
      {loading && <p>Loading emails...</p>}
      {error && <p className="text-red-500">{error}</p>}
      {!loading && !error && emails.length === 0 && <p>No emails found.</p>}
      {!loading && !error && emails.length > 0 && (
        <div>
          {emails.map((email) => (
            <MailItem key={email.id} {...email} />
          ))}
        </div>
      )}
    </div>
  );
};

export default MailContent;
```

---

### Final Setup & Environment Variables

1. **Environment Variables:** Create a `.env.local` file at the root of your project with:

   ```env
   GOOGLE_CLIENT_ID=your_google_client_id
   GOOGLE_CLIENT_SECRET=your_google_client_secret
   NEXTAUTH_SECRET=your_nextauth_secret
   NEXTAUTH_URL=http://localhost:3000
   ```

2. **Install Dependencies:**

   ```bash
   npm install next-auth googleapis
   ```

3. **Run Your App:**

   ```bash
   npm run dev
   ```

After signing in via Google, your dashboard will fetch and display real Gmail data in the MailContent component.

---

**Final Answer:**

This improved implementation configures NextAuth with Google for Gmail read-only access, creates a secure API route to fetch emails using the Gmail API, and updates the MailContent component to dynamically render live email data while handling loading and error states.
