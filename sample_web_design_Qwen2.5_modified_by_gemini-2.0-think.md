```markdown

```typescript
// src/app/page.tsx
import React from 'react';
import Header from '../components/Header';
import Widget from '../components/Widget';
import ProfileContent from '../components/ProfileContent';
import MailContent from '../components/MailContent';
import DriveContent from '../components/DriveContent';
import NotesContent from '../components/NotesContent';
import Footer from '../components/Footer';
import { FiUser, FiMail, FiUploadCloud, FiFileText } from 'react-icons/fi';
import { FiMoreHorizontal } from 'react-icons/fi'; // Import more options icon

const Home: React.FC = () => {
  return (
    <div className="min-h-screen bg-gray-100"> {/* Changed background color to gray-100 to resemble iCloud gray */}
      <Header />
      <main className="container mx-auto p-4">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <Widget icon={<FiUser size={24} className="text-blue-500" />} title="Jess"> {/* Changed icon color to blue-500 */}
            <ProfileContent />
          </Widget>
          <Widget
            icon={<FiMail size={24} className="text-blue-500" />} {/* Changed icon color to blue-500 */}
            title="Mail"
            topRightIcon={<FiMoreHorizontal size={20} className="text-gray-500" />} // Using MoreHorizontal icon for edit/more options
          >
            <MailContent />
          </Widget>
          <Widget icon={<FiUploadCloud size={24} className="text-blue-500" />} title="Drive"> {/* Changed icon color to blue-500 */}
            <DriveContent />
          </Widget>
          <Widget icon={<FiFileText size={24} className="text-blue-500" />} title="Notes"> {/* Changed icon color to blue-500 */}
            <NotesContent />
          </Widget>
        </div>
      </main>
      <Footer />
    </div>
  );
};

export default Home;
```

```typescript
// src/components/MailContent.tsx
import React from 'react';
import { FiMail } from 'react-icons/fi';

const MailItem: React.FC<{ subject: string; sender: string; time: string }> = ({ subject, sender, time }) => {
  return (
    <div className="flex items-start space-x-2 py-2">
      <FiMail size={20} className="text-gray-500" />
      <div>
        <div className="flex justify-between">
          <p className="font-medium">{subject}</p>
          <p className="text-sm text-gray-500">{time}</p>
        </div>
        <p className="text-sm text-gray-500">{sender}</p>
      </div>
    </div>
  );
};

const MailContent: React.FC = () => {
  const emails = [
    { subject: 'Your Apple ID was used to sign in to iCloud', sender: 'Apple', time: '11:38 AM' },
    { subject: 'Your Apple ID was used to sign in to iCloud', sender: 'Apple', time: 'Yesterday' },
    { subject: 'Welcome to iCloud Mail.', sender: 'iCloud', time: 'Yesterday' },
  ];

  return (
    <div>
      <div className="flex justify-between items-center mb-3">
        <span className="font-medium">Inbox</span>
        <span className="text-sm text-blue-500">3 unread mes...</span> {/* Added unread messages text */}
      </div>
      <div>
        {emails.map((email, index) => (
          <MailItem key={index} subject={email.subject} sender={email.sender} time={email.time} />
        ))}
      </div>
    </div>
  );
};

export default MailContent;
```

```typescript
// src/components/ProfileContent.tsx
import React from 'react';

const ProfileContent: React.FC = () => {
  return (
    <div className="text-center"> {/* Center aligning the text content */}
      <div className="rounded-full bg-gray-200 w-16 h-16 mx-auto mb-2 flex items-center justify-center">
        <FiUser size={48} className="text-gray-500" /> {/* User Icon inside profile */}
      </div>
      <p className="text-lg font-medium">Jess</p>
      <p className="text-sm text-gray-500">jesspete888@icloud.com</p> {/* Updated email as in screenshot */}
      <p className="text-sm text-gray-500 mt-2">iCloud</p>
    </div>
  );
};

export default ProfileContent;
```

```typescript
// src/components/DriveContent.tsx
import React from 'react';
import { FiUploadCloud } from 'react-icons/fi';

const DriveContent: React.FC = () => {
  return (
    <div className="flex flex-col items-center justify-center"> {/* Center aligning content */}
      <p className="text-lg font-medium text-left w-full">Recents</p> {/* Aligned Recents text to left */}
      <div className="mt-4 flex flex-col items-center justify-center space-y-2">
        <FiUploadCloud size={48} className="text-gray-500" />
        <p className="text-gray-500 text-center">Upload your first file</p> {/* Center aligned text */}
      </div>
    </div>
  );
};

export default DriveContent;
```

```typescript
// src/components/NotesContent.tsx
import React from 'react';
import { FiPlus } from 'react-icons/fi';

const NotesContent: React.FC = () => {
  return (
    <div className="flex flex-col items-center justify-center"> {/* Center aligning content */}
      <p className="text-lg font-medium text-left w-full">All iCloud</p> {/* Aligned All iCloud text to left */}
      <div className="mt-4 flex flex-col items-center justify-center space-y-2">
        <FiPlus size={48} className="text-gray-500" />
        <p className="text-gray-500 text-center">Create your first note</p> {/* Center aligned text */}
      </div>
    </div>
  );
};

export default NotesContent;
```

**Explanation of Changes:**

1.  **`src/app/page.tsx`**:
    *   Changed `bg-icloud-gray` to `bg-gray-100` as a close approximation using Tailwind CSS default gray palette. You might need to define a custom `icloud-gray` in `tailwind.config.js` for exact color match if needed.
    *   Changed icon colors to `text-blue-500` for visual similarity. Again, you might need custom iCloud blue color definition for exact match.
    *   Replaced `topRightIcon={<FiF...` in `Mail` Widget with `topRightIcon={<FiMoreHorizontal size={20} className="text-gray-500" />}` to use a more appropriate "more options" or "edit" like icon similar to the screenshot.
    *   Removed `topRightIcon` from `Notes` widget as it is not in the screenshot.

2.  **`src/components/MailContent.tsx`**:
    *   Updated `MailItem` component to include `time` prop and display time and sender in a more similar layout to the screenshot using `flex justify-between`.
    *   Updated the `emails` array to include email data that closely resembles the screenshot's content (Apple ID sign-in and Welcome email) with sender names as in the screenshot.
    *   Added "Inbox" and "3 unread mes..." text above the email list to match the Mail widget title area in the screenshot. Used `text-blue-500` for "3 unread mes..." for visual cue.

3.  **`src/components/ProfileContent.tsx`**:
    *   Added a circular gray background and `FiUser` icon to represent the profile picture placeholder.
    *   Updated email address to `jesspete888@icloud.com` to match the screenshot.
    *   Used `text-center` class to center align the profile content within the widget.

4.  **`src/components/DriveContent.tsx`**:
    *   Added `flex flex-col items-center justify-center` to the main `div` to center the content vertically and horizontally within the widget.
    *   Added `text-left w-full` to "Recents" `p` tag to align it to the left within the widget width.
    *   Added `text-center` to "Upload your first file" paragraph for center alignment.

5.  **`src/components/NotesContent.tsx`**:
    *   Applied similar centering and left-alignment adjustments as in `DriveContent.tsx` for "All iCloud" and "Create your first note" text elements.

**To further refine the look:**

*   **Custom Colors**: Define `icloud-blue` and `icloud-gray` in your `tailwind.config.js` file to match the exact colors from the screenshot.
*   **Fonts**: Ensure you are using the correct font family if "Arial, Helvetica, sans-serif" is not the exact font used in the iCloud interface. You might need to adjust the font settings in `globals.css`.
*   **Padding and Margins**: Fine-tune padding and margins in `Widget.tsx` and content components if the spacing is not exactly as in the screenshot.
*   **Shadows and Borders**: Verify if the widget shadows and borders are exactly the same. You might need to adjust the `shadow-md` class or add custom shadow styles if needed in `Widget.tsx`.

Remember to run `npm run dev` or `yarn dev` in your project directory to see the changes live in your browser after modifying the files.

```
