# 📊 Visual Architecture & Flow Diagrams

## App Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        HOST APPLICATION                          │
│                   (Port 4200 - Main Shell)                       │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    Header/Navigation                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─────────────┬──────────────┬──────────────┬────────────────┐ │
│  │  Sidebar    │              │              │                │ │
│  │  ├ Messages │              │              │  Main Content  │ │
│  │  ├ Groups   │  Dynamic     │  Lazy Route  │  Area          │ │
│  │  ├ Settings │  Router      │  Outlet      │  (Remote App)  │ │
│  │  └ More     │              │              │                │ │
│  └─────────────┴──────────────┴──────────────┴────────────────┘ │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                        Footer                               │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
         ↓ Module Federation Dynamic Imports ↓
    ┌────────────────────────────────────────────────┐
    │         REMOTE APPLICATIONS                    │
    │                                                │
    │  ┌──────────────┐  ┌──────────────┐           │
    │  │   Messages   │  │    Groups    │           │
    │  │   Port 4202  │  │   Port 4203  │           │
    │  └──────────────┘  └──────────────┘           │
    │                                                │
    │  ┌──────────────┐                             │
    │  │  Settings    │                             │
    │  │  Port 4204   │                             │
    │  └──────────────┘                             │
    └────────────────────────────────────────────────┘
```

## Messages App Structure

```
┌─────────────────────────────────────────┐
│      MESSAGES APP ARCHITECTURE          │
│      (Port 4202 - Messaging)            │
├─────────────────────────────────────────┤
│                                         │
│  ┌────────────────────────────────────┐ │
│  │   TabLayoutComponent (Tabs)        │ │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ...    │ │
│  │  │ DB  │ │Comp │ │Drft │        │ │
│  │  └─────┘ └─────┘ └─────┘        │ │
│  └────────────────────────────────────┘ │
│         ↓ Router Outlet ↓               │
│  ┌────────────────────────────────────┐ │
│  │  Selected Tab Component            │ │
│  │                                    │ │
│  │  Dashboard, Compose, Draft,        │ │
│  │  Schedule, Sent, etc.              │ │
│  └────────────────────────────────────┘ │
│                                         │
├─────────────────────────────────────────┤
│  Services:                              │
│  ├─ DashboardService                   │
│  ├─ ComposeService                     │
│  ├─ DraftService                       │
│  ├─ ScheduleService                    │
│  └─ SentService                        │
│                                         │
│  Guards:                                │
│  ├─ menuAccessGuard                    │
│  ├─ verificationGuard                  │
│  ├─ scheduleGuard                      │
│  └─ unsavedChangesGuard                │
└─────────────────────────────────────────┘
```

## Groups App Structure

```
┌─────────────────────────────────────────┐
│       GROUPS APP ARCHITECTURE           │
│      (Port 4203 - Group Management)     │
├─────────────────────────────────────────┤
│                                         │
│  ┌────────────────────────────────────┐ │
│  │   GroupLayoutComponent             │ │
│  │   (Layout/Navigation)              │ │
│  └────────────────────────────────────┘ │
│         ↓ Router Outlet ↓               │
│  ┌────────────────────────────────────┐ │
│  │  Routed Component                  │ │
│  │                                    │ │
│  │  • GroupsListComponent             │ │
│  │    ├─ Search, Filter, Sort         │ │
│  │    ├─ Pagination                   │ │
│  │    └─ Action Menu (Edit, Delete)   │ │
│  │                                    │ │
│  │  • GroupsEditComponent             │ │
│  │    ├─ Edit name/members            │ │
│  │    └─ Save changes                 │ │
│  │                                    │ │
│  │  • GroupsExportComponent           │ │
│  │  • GroupsTransferComponent         │ │
│  │  • GroupsMergeComponent            │ │
│  └────────────────────────────────────┘ │
│                                         │
├─────────────────────────────────────────┤
│  Services:                              │
│  ├─ GroupsService (API calls)         │ │
│  └─ [Feature-specific services]       │ │
│                                         │
│  UI Dialogs:                            │
│  ├─ AddGroupDialog                     │ │
│  ├─ DeleteDialog                       │ │
│  ├─ DuplicateDialog                    │ │
│  └─ InfoDialog                         │ │
└─────────────────────────────────────────┘
```

## Settings App Structure

```
┌─────────────────────────────────────────┐
│      SETTINGS APP ARCHITECTURE          │
│     (Port 4204 - User Settings)         │
├─────────────────────────────────────────┤
│                                         │
│  ┌────────────────────────────────────┐ │
│  │  SettingsTabLayoutComponent        │ │
│  │  (Tabbed Interface)                │ │
│  │ ┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐         │ │
│  │ │P ││PW││N ││S ││DT││RP│         │ │
│  │ │r ││d ││o ││i ││ T││ o│         │ │
│  │ │f ││ ││t ││gn││ │ │ P│         │ │
│  │ └──┘└──┘└──┘└──┘└──┘└──┘         │ │
│  └────────────────────────────────────┘ │
│         ↓ Router Outlet ↓               │
│  ┌────────────────────────────────────┐ │
│  │  Tab Component                     │ │
│  │                                    │ │
│  │  • ProfileComponent                │ │
│  │    ├─ Personal Info Form           │ │
│  │    ├─ Organization Info            │ │
│  │    ├─ Address Info                 │ │
│  │    └─ Profile Picture Upload       │ │
│  │                                    │ │
│  │  • PasswordComponent               │ │
│  │    ├─ Current Password             │ │
│  │    ├─ New Password (Validation)    │ │
│  │    └─ Confirm Password             │ │
│  │                                    │ │
│  │  • NotificationComponent           │ │
│  │    ├─ Notification Toggles         │ │
│  │    └─ Frequency Selection          │ │
│  │                                    │ │
│  │  • SignupsComponent                │ │
│  │  • DatetimeComponent               │ │
│  │  • ReferralProgramComponent        │ │
│  └────────────────────────────────────┘ │
│                                         │
├─────────────────────────────────────────┤
│  Services:                              │
│  ├─ ProfileService                     │ │
│  ├─ PasswordService                    │ │
│  ├─ NotificationService                │ │
│  └─ [Feature-specific services]       │ │
│                                         │
│  Shared Services:                       │
│  ├─ UserStateService                   │ │
│  └─ SugApiService                      │ │
└─────────────────────────────────────────┘
```

## Message Compose Flow

```
User Visits /messages
    ↓
TabLayoutComponent Renders
    ↓
Tabs Appear: Dashboard | Compose | Draft | Schedule | Sent
    ↓
User Clicks "Compose" Tab
    ↓
Router navigates to /messages/compose
    ↓
Compose Component Loads
    ↓
Compose Sub-tabs appear: Email | Template | Text
    ↓
User Clicks "Email" Tab
    ↓
ComposeEmailComponent Loads
    ├─ Form Renders with fields:
    │  ├─ Send To (Signups/Groups/Individuals)
    │  ├─ Subject
    │  ├─ Message Body
    │  └─ Action Buttons (Send/Draft)
    │
    ├─ User Fills Form
    │
    └─ User Clicks "Send" or "Save as Draft"
        ↓
        Form Validation
        ↓
        @if (valid)
          API Call via ComposeService
          ↓
          Success Page Displays
          ↓
          Redirect to /messages/dashboard
        @else
          Show Validation Errors
        @endif
```

## Group List Operations Flow

```
User Visits /groups
    ↓
GroupsListComponent Loads
    ↓
Load Groups via GroupsService.getAllGroupsPaginated()
    ↓
Display Table with Groups
│
├─ User Types in Search
│  ├─ Debounce 300ms
│  └─ Call API with search term
│     ↓
│     Update table
│
├─ User Clicks Sort Header
│  └─ Call API with sort params
│     ↓
│     Update table
│
├─ User Changes Page (Pagination)
│  └─ Call API with page number
│     ↓
│     Update table
│
├─ User Clicks Edit (Action Menu)
│  └─ Navigate to /groups/edit/:groupid
│     ↓
│     GroupsEditComponent Loads
│     ↓
│     Edit and Save
│
├─ User Clicks Delete (Action Menu)
│  └─ DeleteDialog Opens
│     ├─ Confirm deletion
│     └─ Call GroupsService.deleteGroup()
│         ↓
│         Remove from table
│
└─ User Clicks Merge (Action Menu)
   └─ Navigate to /groups/merge
      ↓
      GroupsMergeComponent Loads
      ↓
      Select Groups to Merge
      ↓
      Execute Merge via API
```

## Settings Change Flow

```
User Visits /settings
    ↓
SettingsTabLayoutComponent Renders
    ↓
Default Route: /settings/profile
    ↓
ProfileComponent Loads
    ├─ Load user data via ProfileService
    ├─ Populate form with user data
    └─ Display form
        ↓
        User Edits Fields
        ↓
        User Clicks "Save"
        ↓
        Form Validation
        ├─ @if (valid)
        │  ├─ API Call via ProfileService
        │  ├─ On Success: Show success message
        │  ├─ Update UserStateService
        │  └─ Form closes/resets
        │
        └─ @else
           Show validation errors
        
├─ User Clicks "Password" Tab
│  └─ PasswordComponent Loads
│     ├─ Input current password
│     ├─ Input new password (validation)
│     └─ Confirm new password
│        ↓
│        User Clicks "Change Password"
│        ↓
│        API Call via PasswordService
│        ↓
│        Success/Error Message
│
└─ User Clicks "Notifications" Tab
   └─ NotificationComponent Loads
      ├─ Load preferences
      ├─ Toggle notification types
      ├─ Select frequency
      └─ Click "Save"
         ↓
         API Call to save preferences
         ↓
         Preferences Updated
```

## Data Flow: Signals & Observables

```
API Call
  ↓
Service Method
  ↓
Observable (RxJS)
  │
  ├─ map: Transform data
  ├─ catchError: Handle errors
  ├─ takeUntil: Cleanup on destroy
  └─ shareReplay: Cache for subscribers
  ↓
Component Subscribe
  ↓
Update Signal(s)
  ├─ signal.set(value)
  └─ signal.update(fn)
  ↓
Computed Signal
  ├─ computed(() => { return signal(); })
  ├─ Auto-recalculates when dependencies change
  └─ Display in template: {{ computed() }}
  ↓
Template Renders
  ↓
UI Updates on Screen
```

## Service & State Management Pattern

```
┌─────────────────────────────┐
│    USER STATE SERVICE       │
│   (Shared Across Apps)      │
├─────────────────────────────┤
│                             │
│  userProfile$               │
│  ├─ Observable of profile  │
│  ├─ Cached after load      │
│  └─ Shared with all apps   │
│                             │
│  userProfile (Signal)       │
│  ├─ Reactive state         │
│  └─ Auto-updates UI        │
│                             │
│  Methods:                   │
│  ├─ loadUserProfile()      │
│  ├─ setUserProfile()       │
│  ├─ getPlanDisplayName()   │
│  ├─ isUserVerified()       │
│  └─ convertESTtoUserTZ()   │
│                             │
└─────────────────────────────┘
         ↓ Injected in ↓
┌─────────────────────────────┐
│   COMPONENT/SERVICE         │
├─────────────────────────────┤
│                             │
│ userStateService =          │
│   inject(UserStateService)  │
│                             │
│ Usage:                       │
│ ├─ userStateService.        │
│ │  userProfile$             │
│ │  .subscribe(profile => {})│
│ │                           │
│ ├─ const profile =          │
│ │  userStateService        │
│ │  .getCurrentProfile()     │
│ │                           │
│ └─ const plan =             │
│    userStateService.        │
│    getPlanDisplayName()     │
│                             │
└─────────────────────────────┘
         ↓ Provides Data to ↓
┌─────────────────────────────┐
│       TEMPLATE              │
├─────────────────────────────┤
│                             │
│ <!-- Using Observable -->  │
│ {{ data$ | async }}         │
│                             │
│ <!-- Using Signal -->       │
│ {{ data() }}                │
│                             │
│ <!-- Using Computed -->     │
│ {{ doubledData() }}         │
│                             │
└─────────────────────────────┘
```

## Module Federation Loading

```
Host at https://host.signupgenius.rocks:4200
    ↓
Host Route: /messages
    ↓
Router checks: canActivate: [menuAccessGuard]
    ↓
Guard allows access (user has permission)
    ↓
Lazy Load: import('messages/Routes')
    ↓
Module Federation looks up 'messages' remote
    ↓
Remote URL: https://messages.signupgenius.rocks:4202
    ↓
Network: GET https://messages.signupgenius.rocks:4202/remoteEntry.js
    ↓
Download and execute remote bundle
    ↓
Extract: m.remoteRoutes
    ↓
Merge routes into host routing tree
    ↓
Component loads and renders
    ↓
User sees Messages app
```

## Authentication & Guard Flow

```
User tries to access /messages
    ↓
Router checks route guards
    ├─ canActivate: [menuAccessGuard]
    │  ├─ Get user profile from UserStateService
    │  ├─ Check: profile.menuaccess.messages === true?
    │  ├─ @if (true) → Allow access ✅
    │  └─ @else → Check alternatives
    │            ├─ Can access groups? → Redirect
    │            ├─ Can access reports? → Redirect
    │            └─ Cannot access anything → Redirect to default
    │
    └─ canDeactivate: [unsavedChangesGuard]
       ├─ Is form dirty?
       ├─ @if (yes) → Show "Discard changes?" dialog
       │            ├─ User clicks "Yes" → Allow leave
       │            └─ User clicks "No" → Stay
       └─ @if (no) → Allow leave ✅
```

## API Response Handling Pattern

```
API Request
    ↓
    ├─ Success Response (200)
    │  ├─ Type: ApiResponse<T>
    │  ├─ {
    │  │    success: true,
    │  │    data: T,
    │  │    message?: string
    │  │  }
    │  └─ Update UI with data
    │
    ├─ Error Response (4xx, 5xx)
    │  ├─ Type: HttpErrorResponse
    │  ├─ catchError operator
    │  ├─ Extract error message
    │  └─ Show error to user
    │
    └─ Network Error
       ├─ No response from server
       ├─ catchError operator
       └─ Show "Network error" message
```

---

This visual reference helps understand:
- How the three apps are loaded and organized
- The component hierarchy in each app
- User flows through features
- Service and state management patterns
- Data flow from API to UI
- Module Federation loading mechanism
- Authentication and guard workflows
