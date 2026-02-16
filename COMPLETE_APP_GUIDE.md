# 📚 Complete Guide: Messages, Groups & Settings Apps

A comprehensive reference guide for all aspects of the Messages, Groups, and Settings micro-frontend applications.

---

## 📋 Table of Contents

1. [App Overview](#app-overview)
2. [Messages App - Deep Dive](#messages-app---deep-dive)
3. [Groups App - Deep Dive](#groups-app---deep-dive)
4. [Settings App - Deep Dive](#settings-app---deep-dive)
5. [Common Patterns & Best Practices](#common-patterns--best-practices)
6. [How to Work on Features, Bugs & Improvements](#how-to-work-on-features-bugs--improvements)

---

## App Overview

### Architecture
- **Monorepo:** All apps in one repository under `apps/` folder
- **Micro-Frontends:** Each app is standalone, loaded via Module Federation
- **Host:** Main app that orchestrates loading/unloading of remotes
- **Shared Code:** Services and utilities in `services/` and `shared/` folders

### Key Technologies
- **Angular 20** - Main framework
- **TypeScript** - Language with strict typing
- **RxJS** - Reactive programming (Observables)
- **Angular Signals** - Modern state management
- **PrimeNG** - UI components library
- **Nx** - Monorepo tooling
- **Module Federation** - Dynamic remote loading
- **Webpack** - Build bundler

### Development Workflow
```bash
# Start all apps
npm run dev:all:https

# Start individual apps
npm run dev:messages:https
npm run dev:groups:https
npm run dev:settings:https

# Build
npm run build:all
npm run build:messages

# Test
npm run test:messages

# Lint & Format
npm run lint
npm run format:fix
```

---

# MESSAGES APP - DEEP DIVE

## 📊 What Messages Does

Messages module handles all user communication:
- **Compose** - Create new messages (Email, Email Template, SMS/Text)
- **Dashboard** - Overview of messaging stats, recent sent messages, delivery stats
- **Draft** - Manage unsent/saved messages
- **Schedule** - Schedule messages to send at specific times (Premium feature)
- **Sent** - View message history with analytics and delivery details

## 🗂️ Folder Structure

```
apps/messages/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── compose/              # Email/SMS/Template composer
│   │   │   │   ├── compose.ts        # Main composer component
│   │   │   │   ├── compose.html
│   │   │   │   ├── compose.scss
│   │   │   │   ├── compose.service.ts    # API calls for compose
│   │   │   │   ├── compose_email/        # Email composer sub-component
│   │   │   │   ├── compose_text_message/ # SMS composer sub-component
│   │   │   │   └── compose_email_template/ # Template sub-component
│   │   │   │
│   │   │   ├── dashboard/           # Dashboard overview
│   │   │   │   ├── dashboard.ts      # Main dashboard logic
│   │   │   │   ├── dashboard.html
│   │   │   │   ├── dashboard.scss
│   │   │   │   └── dashboard.service.ts
│   │   │   │
│   │   │   ├── draft/
│   │   │   │   ├── draft.ts
│   │   │   │   ├── draft.html
│   │   │   │   └── draft.service.ts
│   │   │   │
│   │   │   ├── schedule/
│   │   │   │   ├── schedule.ts
│   │   │   │   ├── schedule.html
│   │   │   │   └── schedule.service.ts
│   │   │   │
│   │   │   ├── sent/
│   │   │   │   ├── sent.ts
│   │   │   │   ├── sent.html
│   │   │   │   ├── sent-details/     # Message details view
│   │   │   │   ├── message_details/
│   │   │   │   ├── message_analytics/ # Analytics & stats
│   │   │   │   └── sent.service.ts
│   │   │   │
│   │   │   ├── tablayout/
│   │   │   │   ├── tablayout.ts      # Main tab container
│   │   │   │   └── tablayout.html
│   │   │   │
│   │   │   ├── utils/
│   │   │   │   ├── compose-email-state.service.ts  # State management
│   │   │   │   ├── success-page/
│   │   │   │   └── shared components
│   │   │   │
│   │   │   └── shared/
│   │   │       ├── styles/
│   │   │       └── reusable components
│   │   │
│   │   ├── guards/
│   │   │   ├── schedule.guard.ts         # Checks if user can access schedule
│   │   │   └── unsaved-changes.guard.ts  # Warns before leaving unsaved form
│   │   │
│   │   ├── interfaces/
│   │   │   ├── compose.interface.ts      # Compose-related types
│   │   │   ├── sent.interface.ts         # Sent messages types
│   │   │   ├── draft.interface.ts        # Draft types
│   │   │   ├── schedule.interface.ts     # Schedule types
│   │   │   ├── communication-limits.interface.ts # Limits & quotas
│   │   │   └── index.ts                  # Exports all interfaces
│   │   │
│   │   ├── remote-entry/
│   │   │   └── entry.routes.ts           # Routes exposed to Host
│   │   │
│   │   ├── app.routes.ts                 # Local routing
│   │   ├── app.config.ts                 # App configuration
│   │   └── app.component.ts
│   │
│   └── main.ts, index.html, styles.scss
│
├── module-federation.config.ts
├── webpack.config.ts
├── webpack.prod.config.ts
├── vite.config.mts
├── project.json
└── tsconfig.json
```

## 🛣️ Routing Details

### Routes
```typescript
// apps/messages/src/app/remote-entry/entry.routes.ts
export const remoteRoutes: Route[] = [
  {
    path: '',
    component: TabLayoutComponent,
    canActivate: [menuAccessGuard],
    data: { menuAccess: 'messages' },
    children: [
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: Dashboard },
      {
        path: 'compose',
        canActivate: [verificationGuard],  // User must be verified
        component: Compose,
        children: [
          { path: '', redirectTo: 'email', pathMatch: 'full' },
          {
            path: 'email',
            component: ComposeEmailComponent,
            canDeactivate: [unsavedChangesGuard],  // Warn before leaving
          },
          {
            path: 'template',
            component: ComposeEmailTemplateComponent,
            canDeactivate: [unsavedChangesGuard],
          },
          {
            path: 'text',
            component: ComposeTextMessageComponent,
            canDeactivate: [unsavedChangesGuard],
          },
        ],
      },
      { path: 'draft', component: Draft },
      {
        path: 'schedule',
        component: Schedule,
        canActivate: [scheduleGuard],  // Premium users only
      },
      { path: 'sent', component: Sent },
      {
        path: 'sent/:id',
        component: SentDetails,
        children: [
          { path: '', redirectTo: 'details', pathMatch: 'full' },
          { path: 'details', component: MessageDetailsComponent },
          { path: 'analytics', component: MessageAnalyticsComponent },
        ],
      },
    ],
  },
];
```

### Default Route: `/messages/dashboard`

## 🧩 Key Components

### 1. **TabLayoutComponent** (Main Container)
- Renders tabs: Dashboard, Compose, Draft, Schedule, Sent
- Uses `<router-outlet>` to load child routes
- Manages navigation between tabs
- Responsive tab layout

### 2. **Dashboard Component**
**Location:** `apps/messages/src/app/components/dashboard/`

**What it does:**
- Shows messaging statistics
- Displays user's message quota (daily/monthly limits)
- Recent sent messages table
- Delivery stats pie chart (Delivered, Bounced, Dropped, Spam)
- Plan information (Trial, Basic, Gold, Platinum, Enterprise)

**Key Properties:**
- `progressToday` - Emails sent today vs daily limit
- `progressMonth` - Emails sent this month vs monthly limit
- `progressTextMessage` - SMS sent vs limit
- `tableData` - List of recent messages
- `deliveryStats` - Array of delivery status counts
- `chartData` - Pie chart data for delivery visualization

**Key Methods:**
- `loadUserProfileData()` - Gets user info from UserStateService
- `getMessageLimit()` - Fetches user's message limits
- `getMessageSummary()` - Gets recent messages
- `getMessageDeliveryStats()` - Gets delivery statistics
- `convertESTtoUserTZ()` - Converts dates to user timezone

### 3. **Compose Component**
**Location:** `apps/messages/src/app/components/compose/`

**What it does:**
- Main container for composing messages
- Manages three compose tabs: Email, Template, SMS
- Shows which compose methods are restricted based on plan
- Handles success page display after sending

**Key Properties:**
- `navigationComposeTabs` - Array of compose tab options
- `isProUser` - Whether user can access premium features
- `isTrialUser` - Whether user is on trial
- `hasSignups` - Whether user has signups to send to
- `isSuccessRoute` - Whether showing success page
- `successPageType` - Type of success (send, draft, scheduled, etc.)

**Sub-Components:**
- `ComposeEmailComponent` - Compose regular email
- `ComposeEmailTemplateComponent` - Compose from template
- `ComposeTextMessageComponent` - Compose SMS

**Key Methods:**
- `onTabChange()` - Handle tab switching
- `onSuccessPageDismiss()` - Close success page
- `checkSignupAccess()` - Verify user has signups

### 4. **Compose Email Component**
**Location:** `apps/messages/src/app/components/compose/compose_email/`

**Form Fields:**
- `sendToType` - Who to send to (Signups, Groups, Individuals)
- `selectedSignups` - Which signups to send to
- `selectedGroups` - Which groups to include
- `recipientEmails` - Manual email addresses
- `subject` - Email subject
- `message` - Email body
- `theme` - Email template theme (if available)

**Key Methods:**
- `validateForm()` - Validate all required fields
- `sendMessage()` - Send email immediately
- `saveDraft()` - Save as draft without sending
- `previewMessage()` - Preview before sending

### 5. **Draft Component**
**Location:** `apps/messages/src/app/components/draft/`

**What it does:**
- List all draft messages
- Resume editing a draft
- Delete draft messages
- Table view with columns: Created Date, Subject, Recipients, Actions

**Key Methods:**
- `loadDrafts()` - Fetch draft list from API
- `editDraft()` - Open draft in compose view
- `deleteDraft()` - Delete a draft
- `resumeEditing()` - Continue working on draft

### 6. **Schedule Component**
**Location:** `apps/messages/src/app/components/schedule/`

**What it does:**
- View messages scheduled for future sending
- Modify scheduled send times
- Cancel scheduled messages
- Premium feature (Gold+, Platinum, Enterprise)

**Key Properties:**
- `scheduledMessages` - Array of scheduled messages
- `selectedSchedule` - Currently selected schedule
- `newScheduleTime` - New time to reschedule to

**Guard:**
- `scheduleGuard` - Only allows Pro/Premium users (not Basic/Trial)

### 7. **Sent Component**
**Location:** `apps/messages/src/app/components/sent/`

**What it does:**
- Display history of sent messages
- Table with: Date Sent, Subject, Recipients, Actions
- Click on row to view message details
- Filter/search/sort capabilities

**Sub-Components:**
- **SentDetails** - View full message and analytics
  - **MessageDetailsComponent** - Full message content and metadata
  - **MessageAnalyticsComponent** - Opens, clicks, bounces, delivery stats

## 🔄 Key Services & Interfaces

### ComposeService
**Location:** `apps/messages/src/app/components/compose/compose.service.ts`

**Key Methods:**
- `getSignups()` - Fetch list of signups to send to
- `getGroups()` - Fetch list of groups
- `getGroupMembers()` - Get members in a group
- `getMessagePreview()` - Preview message before sending
- `createMessage()` - Send/save message
- `triggerSuccessPage()` - Show success confirmation

**Subjects (for inter-component communication):**
- `showSuccessPage$` - Emits when to show success page
- `isOptionSelected$` - Emits when option is selected

### DashboardService
**Location:** `apps/messages/src/app/components/dashboard/dashboard.service.ts`

**Key Methods:**
- `getMessageLimits()` - Fetch user's daily/monthly limits
- `getMessageSummary()` - Get recent sent messages
- `getMessageDeliveryStats()` - Get delivery breakdown

### SentService
**Location:** `apps/messages/src/app/components/sent/sent.service.ts`

**Key Methods:**
- `getSentMessages()` - Fetch sent message list
- `getMessageDetails()` - Get full message details
- `getMessageAnalytics()` - Get analytics for a message
- `deleteMessage()` - Delete a sent message

### ComposeEmailStateService
**Location:** `apps/messages/src/app/components/utils/compose-email-state.service.ts`

**What it does:**
- Manages compose form state (preventing data loss)
- Stores form values while user navigates
- Restores form when returning to compose

## 🧠 Key Interfaces (TypeScript Types)

### Message-Related Interfaces
```typescript
// From apps/messages/src/app/interfaces/

// Dashboard
interface MessageItem {
  messageid: number;
  sentdate: string;
  subject: string;
  sentTo: string;
  messagetype: 'Email' | 'Text';
  status: string;
  totalsent: number;
}

interface IMessageDeliveryStats {
  name: 'Delivered' | 'Bounced' | 'Dropped' | 'Spam';
  ct: number;  // count
  badgestyle: string;  // CSS class for badge color
}

// Compose
interface ISignUpItem {
  id: string;
  name: string;
  signup_id: number;
}

interface ICreateMessageRequest {
  subject: string;
  message: string;
  sendToType: 'signups' | 'groups' | 'individuals';
  selectedSignups?: number[];
  selectedGroups?: number[];
  recipientEmails?: string[];
  messageType: 'email' | 'template' | 'text';
  theme?: string;
  scheduledTime?: string;  // ISO format if scheduling
  saveAsDraft?: boolean;
}

interface ICreateMessageResponse extends BaseResponse {
  data: {
    messageid: number;
    message: string;
    status: 'sent' | 'scheduled' | 'saved';
  };
}

// Limits
interface MessageLimitsResponse {
  data: {
    sentemailtoday: number;
    dailylimit: number;
    sentemailforthemonth: number;
    monthlimit: number;
    senttexttoday: number;
    textmessagelimit: number;
  };
}
```

## 🔐 Guards

### menuAccessGuard
```typescript
// From @services/guards/menu-access.guard
canActivate: [menuAccessGuard]
data: { menuAccess: 'messages' }
```
- Checks if user has permission to access Messages
- Redirects to first accessible feature if denied

### verificationGuard
```typescript
// On Compose route
canActivate: [verificationGuard]
```
- Ensures user is verified before composing
- Prevents unverified users from sending messages

### scheduleGuard
```typescript
// On Schedule route
canActivate: [scheduleGuard]
```
- Only allows Pro/Premium users
- Blocks Basic/Trial users from schedule tab

### unsavedChangesGuard
```typescript
// On Compose sub-routes
canDeactivate: [unsavedChangesGuard]
```
- Warns user if leaving with unsaved form data
- Prevents accidental data loss

## 🎯 How to Work on Messages Features

### Adding a New Compose Message Type
1. Create component in `components/compose/compose_newtype/`
2. Add route in `entry.routes.ts` under compose children
3. Add tab in `navigationComposeTabs` array in Compose component
4. Import and add to compose template
5. Create service methods in `ComposeService`

### Modifying the Compose Form
1. Edit form group in `ComposeEmailComponent`/relevant sub-component
2. Add form field validation
3. Update `ICreateMessageRequest` interface
4. Update template HTML with new field
5. Update service method to include new field in API request

### Adding a New Message Status or Delivery Type
1. Update `IMessageDeliveryStats` interface
2. Update dashboard pie chart setup in `dashboard.ts`
3. Update chart colors in constants if needed
4. Update API service to handle new status

### Fixing Bug in Message Send
1. Check `ComposeService.createMessage()` method
2. Verify form validation in compose component
3. Check API response handling
4. Check error messages displayed to user
5. Write test case if not existing

---

# GROUPS APP - DEEP DIVE

## 📊 What Groups Does

Groups module manages user groups for messaging and organizing:
- **Groups List** - View all groups, create new, search, filter, paginate
- **Edit Group** - Modify group name, members, settings
- **Export Group** - Download group members as file
- **Transfer Group** - Transfer group ownership
- **Merge Groups** - Combine multiple groups
- **Delete Groups** - Remove groups

## 🗂️ Folder Structure

```
apps/groups/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── group-layout/           # Main tab container
│   │   │   │   ├── group-layout.ts
│   │   │   │   └── group-layout.html
│   │   │   │
│   │   │   ├── groups-list/            # Main list view
│   │   │   │   ├── groups-list.ts      # Main list logic
│   │   │   │   ├── groups-list.html
│   │   │   │   ├── groups-list.scss
│   │   │   │   ├── groups.service.ts   # API calls
│   │   │   │   └── groups-list.spec.ts # Tests
│   │   │   │
│   │   │   ├── groups-edit/            # Edit group
│   │   │   │   ├── groups-edit.ts
│   │   │   │   ├── groups-edit.html
│   │   │   │   └── groups-edit.service.ts
│   │   │   │
│   │   │   ├── groups-export/          # Export group
│   │   │   │   ├── groups-export.ts
│   │   │   │   ├── groups-export.html
│   │   │   │   └── groups-export.service.ts
│   │   │   │
│   │   │   ├── groups-transfer/        # Transfer group
│   │   │   │   ├── groups-transfer.ts
│   │   │   │   ├── groups-transfer.html
│   │   │   │   └── groups-transfer.service.ts
│   │   │   │
│   │   │   ├── groups-merge/           # Merge groups
│   │   │   │   ├── groups-merge.ts
│   │   │   │   ├── groups-merge.html
│   │   │   │   └── groups-merge.service.ts
│   │   │   │
│   │   │   ├── utils/
│   │   │   │   ├── add-group-dialog/
│   │   │   │   ├── delete-dialog/
│   │   │   │   ├── duplicate-group-dialog/
│   │   │   │   ├── info-dialog/
│   │   │   │   ├── premium-box/        # Shows premium features
│   │   │   │   └── send-email-messages/ # Send messages to group
│   │   │   │
│   │   │   ├── shared/
│   │   │   │   └── reusable components
│   │   │   │
│   │   │   └── shared/
│   │   │       └── utilities
│   │   │
│   │   ├── interfaces/
│   │   │   ├── group.interface.ts       # Main group types
│   │   │   ├── index.ts
│   │   │   └── other interfaces
│   │   │
│   │   ├── remote-entry/
│   │   │   └── entry.routes.ts          # Routes exposed to Host
│   │   │
│   │   ├── app.routes.ts
│   │   ├── app.config.ts
│   │   └── app.component.ts
│   │
│   └── main.ts, index.html, styles.scss
│
├── module-federation.config.ts
├── webpack.config.ts
├── webpack.prod.config.ts
├── vite.config.mts
├── project.json
└── tsconfig.json
```

## 🛣️ Routing Details

### Routes
```typescript
// apps/groups/src/app/remote-entry/entry.routes.ts
export const remoteRoutes: Route[] = [
  {
    path: '',
    component: GroupLayoutComponent,
    canActivate: [menuAccessGuard],
    data: { menuAccess: 'groups' },
    children: [
      { path: '', component: GroupsListComponent },        // List all groups
      { path: 'edit/:groupid', component: GroupsEditComponent }, // Edit specific group
      { path: 'export', component: GroupsExportComponent },     // Export functionality
      { path: 'transfer', component: GroupsTransferComponent },   // Transfer ownership
      { path: 'merge', component: GroupsMergeComponent },       // Merge groups
    ],
  },
];
```

### Default Route: `/groups` (shows list)

## 🧩 Key Components

### 1. **GroupLayoutComponent** (Main Container)
- Provides layout structure for all group features
- Manages navigation between group operations
- Similar to Messages' TabLayoutComponent

### 2. **GroupsListComponent** (Main List View)
**Location:** `apps/groups/src/app/components/groups-list/`

**What it does:**
- Display paginated list of all user's groups
- Search groups by name
- Filter groups
- Sort by: Member Count, Group Name, Signup Count
- Pagination (configurable items per page)
- Action menu: Edit, Export, Transfer, Merge, Duplicate, Delete

**Key Properties:**
- `tableData` - List of groups to display (Signal)
- `isLoading` - Loading state (Signal)
- `sortBy` - Current sort column
- `sortOrder` - ASC or DESC
- `pagination` - Pagination config and state
- `searchControl` - FormControl for search input

**Key Methods:**
- `loadGroups()` - Fetch groups from API
- `onSearch()` - Search groups
- `onSort()` - Sort table by column
- `onPageChange()` - Handle pagination
- `onEdit()` - Navigate to edit group
- `onDelete()` - Delete a group
- `onDuplicate()` - Create copy of group
- `onExport()` - Export group members
- `onTransfer()` - Transfer group ownership
- `onMerge()` - Merge multiple groups

**Table Columns:**
- Group Name (sortable, filterable)
- Member Count (sortable)
- Signup Count (sortable)
- Date Created
- Last Modified
- Actions (Edit, Export, Transfer, Merge, Duplicate, Delete)

**Signals & Reactive State:**
```typescript
tableData = signal<IGroup[]>([]);              // Groups to display
isLoading = signal(false);                     // Loading state
isDuplicating = signal(false);                 // Duplicating in progress
isCreatingGroup = signal(false);               // Creating new group
sortBy = signal<'title' | 'membercount' | 'signupcount'>('membercount');
sortOrder = signal<'ASC' | 'DESC'>('DESC');
```

### 3. **GroupsEditComponent**
**Location:** `apps/groups/src/app/components/groups-edit/`

**What it does:**
- Edit group name
- Add/remove members from group
- Manage group settings
- Save changes

**Key Features:**
- Two-way binding for group name
- Dynamic member list management
- Add members (search and select)
- Remove members (with confirmation)
- Save/Cancel actions

### 4. **GroupsExportComponent**
**Location:** `apps/groups/src/app/components/groups-export/`

**What it does:**
- Export group members as CSV/Excel file
- Select format (CSV, Excel, PDF)
- Filter which members to export
- Download file

**Key Methods:**
- `selectFormat()` - Choose export format
- `selectMembers()` - Choose which members to include
- `generateExport()` - Create export file
- `downloadExport()` - Trigger download

### 5. **GroupsTransferComponent**
**Location:** `apps/groups/src/app/components/groups-transfer/`

**What it does:**
- Transfer group ownership to another user
- Select recipient user
- Confirmation before transfer

**Key Methods:**
- `selectRecipient()` - Choose new owner
- `confirmTransfer()` - Execute transfer
- `cancelTransfer()` - Cancel operation

### 6. **GroupsMergeComponent**
**Location:** `apps/groups/src/app/components/groups-merge/`

**What it does:**
- Select multiple groups to merge
- Merge into single group
- Handle duplicate members

**Key Methods:**
- `selectSourceGroups()` - Choose groups to merge from
- `selectTargetGroup()` - Choose destination group
- `executeMerge()` - Perform merge operation
- `handleDuplicateMembers()` - Decide how to handle duplicates

## 🔄 Key Services & Interfaces

### GroupsService
**Location:** `apps/groups/src/app/components/groups-list/groups.service.ts`

**Key Methods:**
```typescript
// Fetch all groups with pagination
getAllGroupsPaginated(
  page: number,
  limit: number,
  sortby: string,
  sort: string,
  search: string
): Observable<IGroupListApiResponse>

// Create new group
createGroup(payload: CreateGroupPayload): Observable<IAddGroupResponse>

// Delete group
deleteGroup(groupId: number): Observable<IDeleteResponse>

// Duplicate group
duplicateGroup(groupId: number): Observable<IDuplicateGroupResponse>

// Get group details
getGroupDetails(groupId: number): Observable<IGroupDetailsResponse>

// Update group
updateGroup(groupId: number, payload: any): Observable<IGroupUpdateResponse>

// Transfer group
transferGroup(groupId: number, payload: ITransferGroupPayload): Observable<ITransferGroupResponse>

// Merge groups
mergeGroups(payload: MergeGroupsPayload): Observable<MergeGroupsResponse>

// Export group
exportGroup(groupId: number, format: string): Observable<DownloadGroupsResponse>
```

### Key Interfaces
```typescript
// Main group interface
export interface IGroup {
  groupid: number;
  title: string;
  membercount: number;
  signupcount: number;
  datecreated: string;
  datemodified: string;
  description?: string;
  isadmin?: boolean;
}

// API response for groups list
export interface IGroupListApiResponse extends BaseResponse {
  data: {
    data: IGroup[];
    total: number;
    page: number;
    limit: number;
  };
}

// Add group response
export interface IAddGroupResponse extends BaseResponse {
  data: {
    groupid: number;
    title: string;
    message: string;
  };
}

// Delete response
export interface IDeleteResponse extends BaseResponse {
  data: {
    message: string;
    success: boolean;
  };
}

// Duplicate response
export interface IDuplicateGroupResponse extends BaseResponse {
  data: {
    groupid: number;
    title: string;
    message: string;
  };
}

// Transfer payload
export interface ITransferGroupPayload {
  groupid: number;
  newownerid: number;
  message?: string;
}

// Merge payload
export interface MergeGroupsPayload {
  sourceGroupIds: number[];
  targetGroupId: number;
  handleDuplicates: 'keep' | 'remove' | 'merge';
}

// Error message structure
export interface Message {
  message: string;
  field?: string;
  type?: 'error' | 'warning' | 'info';
}
```

## 🎯 How to Work on Groups Features

### Adding a New Group Action
1. Add route in `entry.routes.ts`
2. Create new component in `components/`
3. Create service file for API calls
4. Add action button in `GroupsListComponent`
5. Handle navigation/dialog
6. Create service methods in `GroupsService`

### Adding a New Column to Group List
1. Update `tableColumns` array in `GroupsListComponent`
2. Add column definition with:
   - `field` - Property name from IGroup
   - `header` - Display name
   - `sortable` - Boolean
   - `width` - Percentage or pixels
3. Update API response type if needed
4. Update sort logic if sortable

### Fixing Bug in Group Export
1. Check `GroupsExportComponent` logic
2. Verify `GroupsService.exportGroup()` method
3. Check file format generation
4. Test with different file formats
5. Check error handling

### Adding Search/Filter
1. Groups already have search
2. Update search logic in component
3. Call API with search parameter
4. Update `groupsService.getAllGroupsPaginated()` if needed

---

# SETTINGS APP - DEEP DIVE

## 📊 What Settings Does

Settings module manages user account and preferences:
- **Profile** - Edit personal and organizational info
- **Password** - Change account password
- **Notifications** - Email notification preferences
- **Signups** - Default settings for new signups
- **Date/Time** - Date format, timezone, language
- **Referral Program** - View referral code and earnings

## 🗂️ Folder Structure

```
apps/settings/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── tablayout/              # Main tab container
│   │   │   │   ├── tablayout.ts
│   │   │   │   └── tablayout.html
│   │   │   │
│   │   │   ├── profile/                # Profile editor
│   │   │   │   ├── profile.ts          # Main profile logic
│   │   │   │   ├── profile.html
│   │   │   │   ├── profile.scss
│   │   │   │   └── profile.service.ts
│   │   │   │
│   │   │   ├── password/               # Password change
│   │   │   │   ├── password.ts
│   │   │   │   ├── password.html
│   │   │   │   ├── password.scss
│   │   │   │   └── password.service.ts
│   │   │   │
│   │   │   ├── notification/           # Notification preferences
│   │   │   │   ├── notification.ts
│   │   │   │   ├── notification.html
│   │   │   │   └── notification.service.ts
│   │   │   │
│   │   │   ├── signups/                # Signup defaults
│   │   │   │   ├── signups.ts
│   │   │   │   ├── signups.html
│   │   │   │   └── signups.service.ts
│   │   │   │
│   │   │   ├── datetime/               # Date/Time preferences
│   │   │   │   ├── datetime.ts
│   │   │   │   ├── datetime.html
│   │   │   │   └── datetime.service.ts
│   │   │   │
│   │   │   ├── referralprogram/        # Referral info
│   │   │   │   ├── referralprogram.ts
│   │   │   │   ├── referralprogram.html
│   │   │   │   └── referralprogram.service.ts
│   │   │   │
│   │   │   ├── utils/
│   │   │   │   ├── common-dialog/      # Reusable dialog
│   │   │   │   └── other utilities
│   │   │   │
│   │   │   └── shared/
│   │   │       └── reusable components
│   │   │
│   │   ├── interfaces/
│   │   │   └── index.ts
│   │   │
│   │   ├── remote-entry/
│   │   │   └── entry.routes.ts         # Routes exposed to Host
│   │   │
│   │   ├── app.routes.ts
│   │   ├── app.config.ts
│   │   └── app.component.ts
│   │
│   └── main.ts, index.html, styles.scss
│
├── module-federation.config.ts
├── webpack.config.ts
├── webpack.prod.config.ts
├── vite.config.mts
├── project.json
└── tsconfig.json
```

## 🛣️ Routing Details

### Routes
```typescript
// apps/settings/src/app/remote-entry/entry.routes.ts
export const remoteRoutes: Route[] = [
  {
    path: '',
    component: SettingsTabLayoutComponent,
    canActivate: [menuAccessGuard],
    data: { menuAccess: 'settings' },
    children: [
      { path: '', redirectTo: 'profile', pathMatch: 'full' },
      { path: 'profile', component: ProfileComponent },
      { path: 'password', component: PasswordComponent },
      { path: 'notification', component: NotificationComponent },
      { path: 'signups', component: SignupsComponent },
      { path: 'datetime', component: DatetimeComponent },
      { path: 'referralprogram', component: ReferralProgramComponent },
    ],
  },
];
```

### Default Route: `/settings/profile`

## 🧩 Key Components

### 1. **SettingsTabLayoutComponent** (Main Container)
- Provides tabbed interface for all settings
- Tabs: Profile, Password, Notification, Signups, Date/Time, Referral Program

### 2. **ProfileComponent**
**Location:** `apps/settings/src/app/components/profile/`

**What it does:**
- Display and edit user profile information
- Upload profile picture
- Edit personal information
- Edit organization information
- Edit address information

**Form Fields:**
```typescript
profileForm = fb.group({
  firstName: [''],
  lastName: [''],
  email: [''],
  mobile: [''],
  organizationName: [''],
  organizationRole: [''],
  address1: [''],
  address2: [''],
  city: [''],
  state: [''],
  zip: [''],
  country: [''],
  // ... more fields
});
```

**Key Methods:**
- `loadProfile()` - Load user data
- `saveProfile()` - Save changes
- `uploadProfilePicture()` - Handle file upload
- `removeProfilePicture()` - Remove picture
- `resetForm()` - Clear form
- `onCancel()` - Cancel editing

**Key Features:**
- Uses Reactive Forms (FormBuilder, FormGroup)
- Profile picture upload dialog
- Initials display if no picture
- Form validation
- Success/error messages

### 3. **PasswordComponent**
**Location:** `apps/settings/src/app/components/password/`

**What it does:**
- Change user account password
- Validate password strength
- Confirm password match

**Form Fields:**
```typescript
currentPassword = '';                    // Current password for verification
newPassword = signal('');                // New password
confirmPassword = signal('');            // Confirmation of new password
```

**Password Requirements:**
- Minimum 8 characters
- At least one uppercase letter (A-Z)
- At least one lowercase letter (a-z)
- At least one number (0-9)

**Validation Logic:**
```typescript
isNewPasswordValid = computed(() => {
  const v = this.newPassword();
  if (!v) return false;
  
  return (
    v.length >= 8 &&
    /[A-Z]/.test(v) &&
    /[a-z]/.test(v) &&
    /\d/.test(v)
  );
});
```

**Key Methods:**
- `validatePassword()` - Check strength and requirements
- `changePassword()` - Submit password change
- `togglePasswordVisibility()` - Show/hide password
- `handleForgotPassword()` - Open forgot password dialog

**Key Features:**
- Password strength indicator
- Show/hide password toggle
- Forgot password link
- Success notification after change
- Error handling for weak passwords

### 4. **NotificationComponent**
**Location:** `apps/settings/src/app/components/notification/`

**What it does:**
- Configure email notification preferences
- Choose notification types to receive
- Set notification frequency
- Enable/disable specific notification categories

**Notification Types:**
- Newsletter (occasional updates)
- Event Reminders (reminder messages)
- System Alerts (important notifications)
- Digest (daily/weekly digest)
- Marketing (promotional offers)

**Key Methods:**
- `loadNotificationPreferences()` - Get current settings
- `updateNotificationSetting()` - Save preference
- `toggleNotificationType()` - Enable/disable notification
- `saveAll()` - Save all changes

### 5. **SignupsComponent**
**Location:** `apps/settings/src/app/components/signups/`

**What it does:**
- Set default settings for new signups
- Default message/template
- Default location
- Default availability settings

**Key Methods:**
- `loadSignupDefaults()` - Get default settings
- `updateSignupDefault()` - Update a specific default
- `saveSignupDefaults()` - Save all changes
- `resetToDefaults()` - Reset to system defaults

### 6. **DatetimeComponent**
**Location:** `apps/settings/src/app/components/datetime/`

**What it does:**
- Set date format preference
- Set timezone
- Set time format (12-hour or 24-hour)
- Set language/locale

**Options Available:**
- Date Formats:
  - MM/DD/YYYY (US)
  - DD/MM/YYYY (Europe)
  - YYYY-MM-DD (ISO)
  - MM-DD-YYYY
  - etc.
- Timezones: All standard timezones (EST, PST, UTC, etc.)
- Time Format: 12-hour (12:30 PM) or 24-hour (14:30)
- Language: English, Spanish, French, etc.

**Key Methods:**
- `loadDatetimeSettings()` - Get current settings
- `setDateFormat()` - Update date format
- `setTimezone()` - Update timezone
- `setTimeFormat()` - Update time format
- `setLanguage()` - Update language
- `saveAll()` - Persist all changes

**Usage in App:**
- `UserStateService.convertESTtoUserTZ()` uses timezone setting
- Dashboard and other components format dates using this preference

### 7. **ReferralProgramComponent**
**Location:** `apps/settings/src/app/components/referralprogram/`

**What it does:**
- Display user's referral code/link
- Show referral statistics
- Copy referral link to clipboard
- View referral earnings/credits

**Key Data:**
- Referral code (unique identifier)
- Referral link (shareable URL)
- Number of referrals made
- Credits earned
- Referral status (active, pending, etc.)

**Key Methods:**
- `loadReferralData()` - Fetch referral info
- `copyReferralLink()` - Copy to clipboard
- `shareReferralLink()` - Share via email/social
- `viewReferralHistory()` - See past referrals

## 🔐 Guards

### menuAccessGuard
- Checks if user has permission to access Settings
- Applied at route level: `data: { menuAccess: 'settings' }`

## 🎯 How to Work on Settings Features

### Adding a New Settings Tab
1. Create new component in `components/new-feature/`
2. Add route in `entry.routes.ts`
3. Create service file for API calls
4. Add tab button in `SettingsTabLayoutComponent` template
5. Create service methods for data fetching/updating

### Adding a New Profile Field
1. Add field to `profileForm` FormGroup in ProfileComponent
2. Add form control to template with validation
3. Update form submission to include new field
4. Update API request payload
5. Update ProfileService method if needed

### Changing Password Validation Rules
1. Edit `isNewPasswordValid` computed in PasswordComponent
2. Update regex patterns as needed
3. Update validation message in template
4. Test with various password combinations

### Adding New Notification Type
1. Add to notification types enum/array in NotificationComponent
2. Add toggle in template
3. Update API request
4. Handle in save method

---

# COMMON PATTERNS & BEST PRACTICES

## Angular Patterns Used

### 1. **Angular Signals**
```typescript
// Define state
count = signal(0);
name = signal<string | null>(null);
items = signal<Item[]>([]);

// Read value
const value = count();  // Call as function

// Update value
count.set(5);           // Replace value
count.update(v => v + 1); // Update based on current

// Computed derived state
doubled = computed(() => this.count() * 2);

// In template
{{ count() }}
{{ doubled() }}
```

### 2. **Dependency Injection**
```typescript
// Use inject() function instead of constructor
private userStateService = inject(UserStateService);
private router = inject(Router);

// Services provided in root
@Injectable({ providedIn: 'root' })
export class MyService { }
```

### 3. **RxJS Observables & Operators**
```typescript
// Subscribe with takeUntil pattern for cleanup
private destroy$ = new Subject<void>();

method$: Observable<Data> = this.service.getData()
  .pipe(
    takeUntil(this.destroy$),  // Unsubscribe on component destroy
    map(data => transform(data)),
    catchError(error => {
      console.error(error);
      return of([]);  // Default value
    }),
    shareReplay(1)  // Cache and share with multiple subscribers
  );

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

### 4. **Reactive Forms**
```typescript
// FormGroup for complex forms
form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(8)]],
  remember: [false],
});

// Submit
onSubmit() {
  if (this.form.valid) {
    console.log(this.form.value);
  }
}

// In template
<input formControl="email" />
<div *ngIf="form.get('email')?.hasError('required')">Required</div>
```

### 5. **Built-in Control Flow**
```html
<!-- Use @if instead of *ngIf -->
@if (isLoading) {
  <app-loading></app-loading>
}

<!-- Use @for instead of *ngFor -->
@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
}

<!-- Use @switch instead of *ngSwitch -->
@switch (status) {
  @case ('active') {
    <span class="badge-green">Active</span>
  }
  @case ('inactive') {
    <span class="badge-red">Inactive</span>
  }
  @default {
    <span>Unknown</span>
  }
}
```

### 6. **Native Class Bindings**
```html
<!-- Use native binding instead of ngClass -->
<div [class.active]="isActive"></div>
<div [class]="{ 'active': isActive, 'disabled': isDisabled }"></div>

<!-- Use native style binding instead of ngStyle -->
<div [style.color]="color"></div>
<div [style]="{ 'font-size': fontSize + 'px', 'color': color }"></div>
```

### 7. **Route Guards as Functions**
```typescript
// Guard pattern (new functional approach)
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  
  if (authService.isLoggedIn()) {
    return true;
  }
  
  return inject(Router).parseUrl('/login');
};

// Use in route
{
  path: 'admin',
  canActivate: [authGuard],
  component: AdminComponent,
}
```

### 8. **API Service Pattern**
```typescript
// Service wraps HTTP calls
@Injectable({ providedIn: 'root' })
export class DataService {
  private api = inject(SugApiService);
  
  getData(): Observable<Data[]> {
    return this.api.get<ApiResponse<Data[]>>('/endpoint')
      .pipe(
        map(response => response.data),
        shareReplay(1)
      );
  }
  
  createData(payload: Data): Observable<Data> {
    return this.api.post('/endpoint', payload);
  }
}

// Component uses service
export class MyComponent {
  data$ = this.dataService.getData();  // Observable
  
  onSave(data: Data) {
    this.dataService.createData(data).subscribe(
      result => console.log('Success', result),
      error => console.error('Error', error)
    );
  }
}
```

## Form Validation Patterns

### Template-Based Validation
```html
<input
  formControl="email"
  [class.error]="form.get('email')?.invalid && form.get('email')?.touched"
/>
<div *ngIf="form.get('email')?.hasError('required')">
  Email is required
</div>
<div *ngIf="form.get('email')?.hasError('email')">
  Invalid email format
</div>
```

### Computed Validation
```typescript
isFormValid = computed(() => {
  return this.emailSignal() && 
         this.passwordSignal().length >= 8 &&
         this.passwordSignal() === this.confirmPasswordSignal();
});

isEmailInvalid = computed(() => {
  return this.emailSignal() && !this.isValidEmail(this.emailSignal());
});
```

## Error Handling Patterns

### Component Error Handling
```typescript
// Show error message to user
hasError = signal(false);
errorMessage = signal<string>('');

loadData() {
  this.service.getData()
    .pipe(
      takeUntil(this.destroy$),
      catchError(error => {
        this.errorMessage.set(error.message || 'Failed to load data');
        this.hasError.set(true);
        return of([]);  // Default value
      })
    )
    .subscribe(data => {
      this.data.set(data);
    });
}

dismissError() {
  this.hasError.set(false);
  this.errorMessage.set('');
}
```

### Dialog Error Handling
```typescript
// Show error in dialog
showErrorDialog(title: string, messages: string[]) {
  this.isInfoDialogVisible = true;
  this.infoDialogTitle = title;
  this.infoDialogMessages = messages.map(msg => ({
    message: msg,
    type: 'error'
  }));
}
```

## Loading State Patterns

```typescript
// Using signals
isLoading = signal(false);
tableData = signal<Item[]>([]);

loadData() {
  this.isLoading.set(true);
  
  this.service.getData()
    .pipe(
      takeUntil(this.destroy$),
      finalize(() => this.isLoading.set(false))
    )
    .subscribe(data => {
      this.tableData.set(data);
    });
}

// In template
@if (isLoading()) {
  <app-loading-spinner></app-loading-spinner>
} @else {
  <table [data]="tableData()"></table>
}
```

## Component Communication Patterns

### Parent to Child (Input)
```typescript
// Child component
export class ChildComponent {
  data = input<string>();  // New syntax
  
  // Or old syntax
  @Input() data: string;
}

// Parent template
<app-child [data]="parentData"></app-child>
```

### Child to Parent (Output)
```typescript
// Child component
export class ChildComponent {
  onSave = output<Data>();  // New syntax
  
  save(data: Data) {
    this.onSave.emit(data);
  }
  
  // Or old syntax
  @Output() onSave = new EventEmitter<Data>();
}

// Parent template
<app-child (onSave)="handleSave($event)"></app-child>
```

### Sibling Communication (Shared Service)
```typescript
// Shared service
@Injectable({ providedIn: 'root' })
export class SharedStateService {
  private dataSubject = new Subject<Data>();
  data$ = this.dataSubject.asObservable();
  
  updateData(data: Data) {
    this.dataSubject.next(data);
  }
}

// Component 1 (Emitter)
export class Component1 {
  constructor(private sharedState: SharedStateService) {}
  
  onAction() {
    this.sharedState.updateData(newData);
  }
}

// Component 2 (Listener)
export class Component2 implements OnInit {
  data: Data;
  
  ngOnInit() {
    this.sharedState.data$.subscribe(data => {
      this.data = data;
    });
  }
}
```

---

# HOW TO WORK ON FEATURES, BUGS & IMPROVEMENTS

## Development Workflow

### 1. **Setup & Understanding**
- [ ] Read the feature requirements thoroughly
- [ ] Understand which app(s) need changes (Messages, Groups, Settings)
- [ ] Check related routes, components, and services
- [ ] Look at similar existing features
- [ ] Ask questions if unclear

### 2. **Local Development**
```bash
# Start all apps in HTTPS mode
npm run dev:all:https

# Or start specific apps
npm run dev:messages:https
npm run dev:groups:https
npm run dev:settings:https

# Open browser
# https://host.signupgenius.rocks:4200
```

### 3. **Code Changes**

#### **Adding a New Feature**
1. Create component file
   ```bash
   touch apps/messages/src/app/components/my-feature/my-feature.ts
   touch apps/messages/src/app/components/my-feature/my-feature.html
   touch apps/messages/src/app/components/my-feature/my-feature.scss
   ```

2. Build component skeleton
   ```typescript
   import { Component, OnInit, OnDestroy, ChangeDetectionStrategy, inject } from '@angular/core';
   import { CommonModule } from '@angular/common';
   import { Subject } from 'rxjs';
   import { takeUntil } from 'rxjs/operators';
   
   @Component({
     selector: 'sug-my-feature',
     standalone: true,
     imports: [CommonModule],
     changeDetection: ChangeDetectionStrategy.OnPush,
     templateUrl: './my-feature.html',
     styleUrl: './my-feature.scss',
   })
   export class MyFeatureComponent implements OnInit, OnDestroy {
     private destroy$ = new Subject<void>();
   
     ngOnInit() {
       // Initialize
     }
   
     ngOnDestroy() {
       this.destroy$.next();
       this.destroy$.complete();
     }
   }
   ```

3. Add route in `entry.routes.ts`
   ```typescript
   { path: 'my-feature', component: MyFeatureComponent }
   ```

4. Create service for API calls
   ```bash
   touch apps/messages/src/app/components/my-feature/my-feature.service.ts
   ```

5. Update templates and styles

#### **Fixing a Bug**
1. Reproduce the bug locally
2. Use browser DevTools to debug
   - Check Network tab for API calls
   - Check Console for errors
   - Check React DevTools (or similar) for component state
3. Locate the relevant code
4. Add console logs or debugger statements
5. Identify root cause
6. Apply fix
7. Test the fix

#### **Making an Improvement**
1. Identify performance bottleneck or UX issue
2. Research solution (caching, memoization, etc.)
3. Implement improvement
4. Test thoroughly
5. Measure improvement if possible

### 4. **Testing**

```bash
# Run tests for specific app
npm run test:messages

# Run specific test file
npm run test:messages -- --include='**/my-feature.spec.ts'

# Run with coverage
npm run test:ci

# Watch mode
npm run test:messages -- --watch
```

**Writing Tests:**
```typescript
// apps/messages/src/app/components/my-feature/my-feature.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { MyFeatureComponent } from './my-feature';

describe('MyFeatureComponent', () => {
  let component: MyFeatureComponent;
  let fixture: ComponentFixture<MyFeatureComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [MyFeatureComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(MyFeatureComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display data when loaded', () => {
    component.data.set([{ id: 1, name: 'Test' }]);
    fixture.detectChanges();
    
    const element = fixture.nativeElement.querySelector('[data-testid="item"]');
    expect(element.textContent).toContain('Test');
  });
});
```

### 5. **Code Quality**

```bash
# Lint all apps
npm run lint

# Fix linting issues
npm run lint:fix

# Format code
npm run format:fix

# Check formatting
npm run format
```

### 6. **Git Workflow**

```bash
# Create feature branch
git checkout -b feature/my-new-feature

# Make changes, commit
git add .
git commit -m "feat: add new feature to messages"

# Push to remote
git push origin feature/my-new-feature

# Create pull request on GitHub/GitLab
# - Link to issue if applicable
# - Describe changes
# - List testing done
```

### 7. **Pull Request Checklist**
- [ ] Code follows project style guide
- [ ] No console errors or warnings
- [ ] Tests written and passing
- [ ] Lint/format checks passing
- [ ] Works on HTTPS (if using auth/cookies)
- [ ] Tested in affected apps
- [ ] Documentation updated if needed
- [ ] No breaking changes

## Common Tasks

### Add a Tab/Feature to an App
```typescript
// 1. Create component
ng g component apps/messages/src/app/components/new-feature

// 2. Add route in entry.routes.ts
{ path: 'new-feature', component: NewFeatureComponent }

// 3. Add tab button in layout component
// In tablayout.html: <a routerLink="/messages/new-feature">New Feature</a>

// 4. Add to imports in layout component
import { NewFeatureComponent } from '../new-feature/new-feature';
```

### Add Form Validation
```typescript
// 1. Create form with validators
form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(8)]],
});

// 2. Show error in template
@if (form.get('email')?.hasError('email') && form.get('email')?.touched) {
  <span class="error">Invalid email format</span>
}
```

### Make API Call
```typescript
// 1. Add method to service
getData(): Observable<Data[]> {
  return this.api.get<ApiResponse<Data[]>>('/endpoint');
}

// 2. Use in component
data$ = this.service.getData().pipe(
  map(response => response.data),
  takeUntil(this.destroy$)
);

// 3. Display in template
@for (item of (data$ | async); track item.id) {
  <div>{{ item.name }}</div>
}
```

### Handle Errors
```typescript
// 1. Add error handling to API call
this.service.getData()
  .pipe(
    takeUntil(this.destroy$),
    catchError(error => {
      this.errorMessage.set(error.message || 'Failed to load');
      return of([]);
    })
  )
  .subscribe(data => {
    this.data.set(data);
  });

// 2. Display error in template
@if (errorMessage()) {
  <div class="alert alert-error">{{ errorMessage() }}</div>
}
```

### Add Loading State
```typescript
// 1. Add loading signal
isLoading = signal(false);

// 2. Set loading in component
this.isLoading.set(true);

// 3. Show loader in template
@if (isLoading()) {
  <app-loading-spinner></app-loading-spinner>
} @else {
  <div>Content</div>
}
```

## Debugging Tips

### Browser DevTools
1. Open F12 or right-click → Inspect
2. Check Console for errors
3. Check Network tab for API calls
4. Set breakpoints in Sources tab

### Chrome Extensions
- Angular DevTools - View component tree and signals
- Redux DevTools - If using state management
- Lighthouse - Performance analysis

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| "Remote not loading" | Check Module Federation config, verify port running |
| "Cannot read property of null" | Add null check with signal/optional chaining |
| "Form not submitting" | Check form validity: `form.valid` |
| "Component not rendering" | Check `ChangeDetectionStrategy`, signal updates |
| "Style not applying" | Check CSS specificity, ViewEncapsulation, ng-deep |
| "Memory leak" | Ensure `takeUntil` and `ngOnDestroy` cleanup |
| "Slow pagination" | Add pagination caching, lazy loading |
| "CORS errors" | Check backend CORS config, API URL |

---

## Quick Reference Commands

```bash
# Development
npm run dev:all:https                  # Start all apps (HTTPS)
npm run dev:messages:https             # Start Messages
npm run dev:groups:https               # Start Groups
npm run dev:settings:https             # Start Settings

# Building
npm run build:all                      # Build all apps
npm run build:messages                 # Build specific app

# Testing
npm run test:messages                  # Test specific app
npm run test:ci                        # Test with coverage

# Code Quality
npm run lint                           # Check lint issues
npm run lint:fix                       # Auto-fix lint issues
npm run format:fix                     # Auto-format code
npm run check:all                      # Format + lint check

# Utilities
npm run clean                          # Clear Nx cache
npm run dep-graph                      # View dependency graph
npm run affected:build                 # Build only affected
npm run affected:test                  # Test only affected

# Git
git checkout -b feature/description    # Create feature branch
git add .                              # Stage changes
git commit -m "type: description"      # Commit
git push origin feature/description    # Push to remote
```

---

## Summary

You're now ready to work on Messages, Groups, and Settings apps if you:

✅ Understand the folder structure and routing of each app
✅ Know how to create components and services
✅ Can add new features and routes
✅ Know how to write forms and handle validation
✅ Understand how to make API calls
✅ Can debug issues using DevTools
✅ Know the testing workflow
✅ Follow Angular best practices and patterns

**Start by:**
1. Running the apps locally
2. Exploring existing components
3. Making a small improvement to get familiar
4. Reading related code before making changes
5. Testing thoroughly before submitting PR

Good luck! 🚀
