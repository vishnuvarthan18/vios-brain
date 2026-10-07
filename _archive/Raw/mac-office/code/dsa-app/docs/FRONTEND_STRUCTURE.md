# Frontend Structure

## Web app (coordinators + admins)
`web/` — React + Vite + TypeScript

```
web/
├── index.html
├── vite.config.ts
├── tsconfig.json
├── package.json
├── .env.example
└── src/
    ├── main.tsx                          # Entry point
    ├── App.tsx                           # Router + providers
    ├── tokens.css                        # Design tokens (brand colours + fonts)
    │
    ├── api/                              # All API calls — never call fetch() directly in components
    │   ├── client.ts                     # Axios instance + Auth0 token injection
    │   ├── users.ts
    │   ├── hubs.ts
    │   ├── students.ts
    │   ├── sessions.ts
    │   ├── attendance.ts
    │   ├── payments.ts
    │   ├── levels.ts
    │   ├── projects.ts
    │   ├── certificates.ts
    │   ├── notifications.ts
    │   └── openSessions.ts
    │
    ├── components/
    │   ├── ui/                           # Reusable base components
    │   │   ├── Button.tsx                # primary/secondary/danger/ghost variants
    │   │   ├── Input.tsx                 # with label + error state
    │   │   ├── Select.tsx
    │   │   ├── Textarea.tsx
    │   │   ├── Badge.tsx                 # status: paid/due/sponsored/active/inactive
    │   │   ├── Card.tsx
    │   │   ├── Modal.tsx
    │   │   ├── ConfirmModal.tsx          # "Are you sure?" pattern
    │   │   ├── DsaAvatar.tsx             # Robot avatar — prop: dsaId, size
    │   │   ├── EmptyState.tsx            # empty + error states (consistent across app)
    │   │   ├── Spinner.tsx
    │   │   ├── Pagination.tsx
    │   │   ├── Table.tsx                 # data table with sort + pagination
    │   │   ├── SearchInput.tsx           # debounced search
    │   │   ├── Toast.tsx                 # success/error toasts
    │   │   └── OfflineBanner.tsx         # shown when API unreachable
    │   │
    │   ├── layout/
    │   │   ├── Layout.tsx                # sidebar + topbar wrapper
    │   │   ├── Sidebar.tsx               # dark ink, permission-driven nav items
    │   │   ├── TopBar.tsx                # hub name, notification bell, user menu
    │   │   └── HubSwitcher.tsx           # for users with multi-hub access
    │   │
    │   └── features/                     # Feature-specific components
    │       ├── students/
    │       │   ├── StudentRow.tsx        # table row — DSA ID + avatar + status
    │       │   ├── EnrollmentForm.tsx
    │       │   └── ReactivateModal.tsx
    │       ├── sessions/
    │       │   ├── SessionCard.tsx
    │       │   └── AttendanceRoster.tsx
    │       ├── payments/
    │       │   ├── PaymentRow.tsx
    │       │   └── MarkPaidModal.tsx
    │       └── notifications/
    │           ├── NotificationBell.tsx
    │           └── NotificationItem.tsx
    │
    ├── pages/
    │   ├── auth/
    │   │   ├── Login.tsx                 # redirects to Auth0
    │   │   ├── Callback.tsx              # Auth0 callback handler
    │   │   ├── TermsAcceptance.tsx       # first login: T&C must accept
    │   │   └── FirstTimeSetup.tsx        # per-role onboarding
    │   │
    │   ├── dashboard/
    │   │   ├── CoordinatorDashboard.tsx  # hub overview: students, fees, sessions
    │   │   ├── AdminDashboard.tsx        # cross-hub: all hubs summary
    │   │   └── SuperAdminDashboard.tsx   # platform: all sections + alerts
    │   │
    │   ├── students/
    │   │   ├── StudentList.tsx           # paginated, search by DSA ID
    │   │   ├── StudentDetail.tsx         # profile + outcomes + notes + payments
    │   │   ├── StudentEnroll.tsx         # enrollment form
    │   │   └── StudentOutcomes.tsx       # outcome tracking per student
    │   │
    │   ├── sessions/
    │   │   ├── TimetableView.tsx         # weekly view of recurring slots
    │   │   ├── SessionList.tsx           # upcoming + past sessions
    │   │   ├── SessionDetail.tsx         # session info + attendance
    │   │   └── AttendanceView.tsx        # mark attendance (coordinator side)
    │   │
    │   ├── payments/
    │   │   ├── PaymentDashboard.tsx      # expected vs collected, overdue flags
    │   │   └── SponsorshipManager.tsx
    │   │
    │   ├── levels/
    │   │   ├── LevelManager.tsx          # create/reorder levels + outcomes
    │   │   └── OutcomeEditor.tsx
    │   │
    │   ├── portfolios/
    │   │   ├── ProjectList.tsx           # all projects, filter by status
    │   │   └── ProjectReview.tsx         # coordinator approve/reject
    │   │
    │   ├── certificates/
    │   │   ├── CertificateQueue.tsx      # pending approval list
    │   │   └── CertificateDetail.tsx
    │   │
    │   ├── open-sessions/
    │   │   ├── OpenSessionList.tsx
    │   │   ├── OpenSessionCreate.tsx
    │   │   └── OpenSessionDetail.tsx
    │   │
    │   ├── hubs/
    │   │   ├── HubList.tsx
    │   │   ├── HubCreate.tsx
    │   │   └── HubDetail.tsx
    │   │
    │   ├── users/
    │   │   ├── UserList.tsx
    │   │   ├── UserDetail.tsx
    │   │   └── RoleManager.tsx
    │   │
    │   ├── reports/
    │   │   ├── ImpactReport.tsx          # for grant funders
    │   │   ├── AttendanceReport.tsx
    │   │   └── PaymentReport.tsx
    │   │
    │   └── admin/
    │       ├── PlatformSettings.tsx
    │       ├── SectionRegistry.tsx
    │       ├── AuditLog.tsx
    │       ├── PermissionOverrides.tsx
    │       └── IntegrationHealth.tsx
    │
    ├── hooks/
    │   ├── usePermissions.ts             # can('action:name') → boolean
    │   ├── useCurrentHub.ts              # active hub from context
    │   ├── useNotifications.ts           # real-time via Supabase Realtime
    │   ├── useOffline.ts                 # online/offline state
    │   └── useDebounce.ts
    │
    ├── stores/                           # Zustand stores
    │   ├── authStore.ts                  # user, roles, permissions
    │   ├── hubStore.ts                   # active hub
    │   └── notificationStore.ts          # unread count + list
    │
    ├── utils/
    │   ├── avatar.ts                     # DSA robot avatar generator (pure function)
    │   ├── studentCode.ts                # DSA ID formatting
    │   ├── date.ts                       # Sri Lanka timezone helpers
    │   ├── currency.ts                   # LKR formatting
    │   └── errors.ts                     # API error parsing
    │
    └── types/
        ├── api.ts                        # API response types
        └── permissions.ts               # Permission type imports
```

---

## Mobile app (trainers + parents)
`mobile/` — React + Vite + Capacitor (same component library, different routing)

```
mobile/
├── capacitor.config.ts
├── android/
├── ios/
├── package.json
└── src/
    ├── main.tsx
    ├── App.tsx                           # Mobile router
    │
    ├── components/
    │   ├── ui/                           # Same base components as web (import from shared/)
    │   └── mobile/                       # Mobile-specific
    │       ├── BottomTabBar.tsx
    │       ├── OfflineBanner.tsx         # prominent offline indicator
    │       └── SyncIndicator.tsx         # "3 sessions waiting to sync"
    │
    ├── pages/
    │   ├── trainer/
    │   │   ├── TodaySessions.tsx         # trainer home — today's sessions
    │   │   ├── AttendanceMarker.tsx      # mark attendance — DSA ID roster
    │   │   ├── StudentList.tsx           # hub students (view only)
    │   │   ├── StudentProgress.tsx       # outcomes + notes
    │   │   ├── ProjectCreate.tsx         # create + submit portfolio
    │   │   └── OutcomeUpdate.tsx         # bulk outcome marking
    │   │
    │   └── parent/
    │       ├── ChildDashboard.tsx        # progress overview
    │       ├── AttendanceHistory.tsx
    │       ├── OutcomeProgress.tsx
    │       ├── PaymentHistory.tsx        # own payments
    │       ├── Notes.tsx                 # parent-visible notes
    │       └── OpenSessions.tsx          # browse + register for events
    │
    ├── offline/
    │   ├── queue.ts                      # offline action queue (Capacitor Preferences)
    │   ├── sync.ts                       # sync queue on reconnect
    │   └── cache.ts                      # cache student + session data
    │
    └── hooks/
        ├── useOfflineQueue.ts
        ├── useNetwork.ts
        └── useSync.ts
```

---

## Shared utilities (optional shared package)

```
shared/
├── src/
│   ├── avatar.ts                         # Robot avatar generator
│   ├── studentCode.ts                    # DSA ID formatting + validation
│   ├── permissions.ts                    # Permission type (not the middleware)
│   └── constants.ts                      # Shared constants
```

---

## Design token reference (`web/src/tokens.css`)

```css
:root {
  /* Brand */
  --violet-500: #7008C0;   /* Primary — 8.45:1 contrast on white */
  --violet-600: #590699;
  --violet-700: #430473;
  --violet-50:  #EAD2FD;
  --orange-500: #F86800;   /* Accent fill — use with white text */
  --orange-700: #A14300;   /* Orange TEXT on light backgrounds — 6.33:1 */
  --orange-50:  #FFE0CA;

  /* Neutrals */
  --ink:        #13101A;   /* Dark surface background */
  --ink-soft:   #2A2533;
  --muted:      #7C7689;
  --line:       #E4DEEC;
  --paper-2:    #F2EEF7;
  --paper:      #FAF8FC;

  /* Semantic */
  --success:    #1F9D63;
  --danger:     #D23F3F;
  --warning:    #E0A800;

  /* Fonts */
  --font-display: 'Outfit', sans-serif;
  --font-body:    'DM Sans', sans-serif;
  --font-mono:    'Space Grotesk', monospace;   /* DSA IDs + codes */
}
```

---

## Web .env.example

```
VITE_API_URL=http://localhost:3001
VITE_AUTH0_DOMAIN=dreamspace.au.auth0.com
VITE_AUTH0_CLIENT_ID=[web_spa_client_id]
VITE_AUTH0_AUDIENCE=https://appapi.dreamspace.academy
VITE_SUPABASE_URL=https://[ref].supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
```

## Mobile .env.example

```
VITE_API_URL=http://localhost:3001
VITE_AUTH0_DOMAIN=dreamspace.au.auth0.com
VITE_AUTH0_CLIENT_ID=[native_client_id]
VITE_AUTH0_AUDIENCE=https://appapi.dreamspace.academy
VITE_AUTH0_REDIRECT=academy.dreamspace.app://callback
VITE_SUPABASE_URL=https://[ref].supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
VITE_FCM_VAPID_KEY=[web_push_cert_key]
```
