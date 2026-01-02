# Product Requirements Document (PRD)
# Silsilah Genealogy Application - Modern Monolith Refactor

**Document Version:** 1.0
**Date:** January 2, 2026
**Author:** Engineering Team
**Status:** Draft

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Project Overview](#2-project-overview)
3. [Current State Analysis](#3-current-state-analysis)
4. [Target Architecture](#4-target-architecture)
5. [Feature Requirements](#5-feature-requirements)
6. [Technical Requirements](#6-technical-requirements)
7. [Vue.js Component Architecture](#7-vuejs-component-architecture)
8. [API & Data Flow Design](#8-api--data-flow-design)
9. [Database Considerations](#9-database-considerations)
10. [Migration Strategy](#10-migration-strategy)
11. [Testing Strategy](#11-testing-strategy)
12. [Security Considerations](#12-security-considerations)
13. [Performance Requirements](#13-performance-requirements)
14. [Internationalization (i18n)](#14-internationalization-i18n)
15. [Deployment Strategy](#15-deployment-strategy)
16. [Risks & Mitigations](#16-risks--mitigations)
17. [Success Metrics](#17-success-metrics)
18. [Appendices](#18-appendices)

---

## 1. Executive Summary

### 1.1 Purpose

This PRD outlines the comprehensive plan for refactoring the Silsilah genealogy application from a traditional Laravel + Blade server-side rendered application to a modern monolith architecture using **Laravel**, **Inertia.js**, and **Vue.js 3**.

### 1.2 Goals

| Goal | Description |
|------|-------------|
| **Modern Stack** | Adopt Inertia.js + Vue.js 3 for a reactive, component-based frontend |
| **Developer Experience** | Improve DX with TypeScript, hot module replacement, and modern tooling |
| **User Experience** | Deliver SPA-like experience with faster navigation and better interactivity |
| **Maintainability** | Create reusable Vue components and establish clear architectural patterns |
| **Performance** | Optimize initial load times and enable partial page updates |
| **Future-Ready** | Build a foundation for future features like real-time updates and PWA capabilities |

### 1.3 Success Criteria

- [ ] 100% feature parity with existing application
- [ ] All existing tests pass in the new architecture
- [ ] Page load time improvement of ≥30%
- [ ] Component test coverage ≥80%
- [ ] Zero regression in functionality
- [ ] Full bilingual support (Indonesian/English) maintained

### 1.4 Out of Scope

- Mobile native application
- Complete UI/UX redesign (styling remains Bootstrap-based)
- New feature development (except where modernization enables improvements)
- Database schema changes (migration preserves existing schema)

---

## 2. Project Overview

### 2.1 Application Description

**Silsilah** (meaning "genealogy" in Indonesian) is a comprehensive family tree management system that enables users to:

- Record and manage family member profiles
- Establish and visualize family relationships (parents, children, spouses)
- Track marriages, divorces, births, and deaths
- Visualize family trees and genealogical charts
- Upload and manage family photos
- Track cemetery locations with map integration
- Manage upcoming birthdays

### 2.2 Target Users

| User Type | Description | Key Needs |
|-----------|-------------|-----------|
| **Family Historians** | Primary users documenting family lineage | Easy data entry, visualization, accuracy |
| **Family Members** | Contributors adding their own information | Simple profile management, photo uploads |
| **Administrators** | System managers with full access | Backup/restore, user management, data integrity |

### 2.3 Key Stakeholders

- Development Team
- End Users (Family Researchers)
- System Administrators
- Open Source Community

---

## 3. Current State Analysis

### 3.1 Technology Stack

| Layer | Current Technology | Version |
|-------|-------------------|---------|
| **Backend Framework** | Laravel | 8.x |
| **PHP Version** | PHP | 7.3+ / 8.0+ |
| **Frontend Rendering** | Blade Templates | - |
| **CSS Framework** | Bootstrap (SASS) | 3.3.7 |
| **JavaScript** | jQuery + Vanilla JS | 3.4.0 |
| **Build Tool** | Laravel Mix (Webpack) | 1.x |
| **Database** | MySQL/MariaDB | 5.7+ |
| **Testing** | PHPUnit | 9.x |

### 3.2 Codebase Statistics

| Metric | Count |
|--------|-------|
| Controllers | 9 |
| Models | 3 (User, Couple, UserMetadata) |
| Blade Templates | 36 |
| Routes | ~30 |
| Database Tables | 4 |
| Test Files | 10 |
| Language Files | 20 (10 per locale) |

### 3.3 Current Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser                                  │
├─────────────────────────────────────────────────────────────────┤
│                    Full Page Requests                            │
│                         ↓  ↑                                     │
├─────────────────────────────────────────────────────────────────┤
│                    Laravel Router                                │
│                         ↓                                        │
├─────────────────────────────────────────────────────────────────┤
│                     Controllers                                  │
│   ┌──────────┐  ┌──────────┐  ┌──────────────────┐             │
│   │  Users   │  │ Couples  │  │ FamilyActions    │  ...        │
│   └──────────┘  └──────────┘  └──────────────────┘             │
│                         ↓                                        │
├─────────────────────────────────────────────────────────────────┤
│                 Eloquent Models                                  │
│   ┌──────────┐  ┌──────────┐  ┌──────────────────┐             │
│   │   User   │  │  Couple  │  │  UserMetadata    │             │
│   └──────────┘  └──────────┘  └──────────────────┘             │
│                         ↓                                        │
├─────────────────────────────────────────────────────────────────┤
│                 MySQL Database                                   │
│   ┌──────────┐  ┌──────────┐  ┌──────────────────┐             │
│   │  users   │  │ couples  │  │  user_metadata   │  ...        │
│   └──────────┘  └──────────┘  └──────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Blade Templates                               │
│   ┌──────────┐  ┌──────────┐  ┌──────────────────┐             │
│   │ Layouts  │  │  Views   │  │    Partials      │             │
│   └──────────┘  └──────────┘  └──────────────────┘             │
│                         ↓                                        │
│              HTML + jQuery + Plugins                             │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 Current Pain Points

| Issue | Impact | Priority |
|-------|--------|----------|
| Full page reloads on navigation | Poor UX, slow perceived performance | High |
| jQuery-based interactivity | Difficult to maintain, not reactive | High |
| No component reusability | Code duplication across partials | Medium |
| Bootstrap 3 (deprecated) | Security/compatibility concerns | Medium |
| No TypeScript | Reduced developer productivity | Medium |
| Limited client-side validation | Poor form UX | Low |
| Manual DOM manipulation | Error-prone, hard to debug | Medium |

### 3.5 Current Features Inventory

#### 3.5.1 User/Person Management
- [x] Create, read, update, delete family members
- [x] Photo upload and management
- [x] Profile attributes (name, DOB, gender, contact info)
- [x] Death date and cemetery location tracking
- [x] Map integration for cemetery locations (Leaflet.js)

#### 3.5.2 Family Relationships
- [x] Set/change father and mother
- [x] Add children
- [x] Add spouses (husband/wife)
- [x] Support for parent couples
- [x] Sibling calculation
- [x] Delete and replace user functionality

#### 3.5.3 Family Visualization
- [x] Family tree view (descendants)
- [x] Alternative tree view (tree2)
- [x] Genealogical chart (ancestors + descendants)
- [x] Search functionality with family cards

#### 3.5.4 Authentication & Authorization
- [x] User registration and login
- [x] Password reset and change
- [x] Policy-based authorization
- [x] Admin role (email-based)

#### 3.5.5 Admin Features
- [x] Database backup creation
- [x] Backup download and restore
- [x] Backup file management

#### 3.5.6 Other Features
- [x] Upcoming birthdays display
- [x] Bilingual support (ID/EN)
- [x] Manager tracking for records

---

## 4. Target Architecture

### 4.1 New Technology Stack

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| **Backend Framework** | Laravel | 11.x | API endpoints, business logic, authentication |
| **PHP Version** | PHP | 8.2+ | Modern language features, performance |
| **Inertia.js** | Inertia.js | 1.x | SPA-like navigation, bridges Laravel + Vue |
| **Frontend Framework** | Vue.js | 3.4+ | Reactive UI components |
| **Build Tool** | Vite | 5.x | Fast HMR, ES modules, optimized builds |
| **CSS Framework** | Bootstrap | 5.3+ | Updated styling, no jQuery dependency |
| **TypeScript** | TypeScript | 5.x | Type safety, better DX |
| **State Management** | Pinia | 2.x | Global state (auth, preferences) |
| **Form Handling** | @inertiajs/vue3 + VeeValidate | - | Form state, validation |
| **HTTP Client** | Axios (via Inertia) | - | API requests |
| **Testing (Frontend)** | Vitest + Vue Test Utils | - | Component testing |
| **Testing (Backend)** | PHPUnit + Pest | - | Feature and unit testing |

### 4.2 Target Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Browser                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                         Vue.js 3 SPA                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        Vue Router*                               │   │
│  │                    (Managed by Inertia)                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐   │
│  │   Pages/    │  │ Components/ │  │  Composables │  │   Stores   │   │
│  │   Views     │  │   Shared    │  │   (Hooks)    │  │  (Pinia)   │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘   │
│                              ↓                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      Inertia.js Client                           │   │
│  │            (XHR requests, partial reloads, history)              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓ JSON/XHR
┌─────────────────────────────────────────────────────────────────────────┐
│                           Laravel Backend                                │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    Inertia.js Server Adapter                     │   │
│  │              (Renders props, handles page components)            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                         Controllers                              │   │
│  │    (Return Inertia::render() instead of view())                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐   │
│  │ Requests │  │ Policies │  │ Services │  │ Resources (optional) │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────────┘   │
│                              ↓                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      Eloquent Models                             │   │
│  │              User  |  Couple  |  UserMetadata                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      MySQL Database                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Directory Structure (Target)

```
silsilah/
├── app/
│   ├── Http/
│   │   ├── Controllers/           # Controllers (returning Inertia responses)
│   │   ├── Middleware/
│   │   │   └── HandleInertiaRequests.php  # Shared data middleware
│   │   └── Requests/              # Form request validation
│   ├── Models/                    # Eloquent models (unchanged)
│   ├── Policies/                  # Authorization policies (unchanged)
│   ├── Services/                  # Business logic services (new)
│   └── Providers/
├── bootstrap/
├── config/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── public/
├── resources/
│   ├── js/
│   │   ├── app.ts                 # Vue application entry point
│   │   ├── bootstrap.ts           # Axios, plugins setup
│   │   ├── types/                 # TypeScript type definitions
│   │   │   ├── index.d.ts
│   │   │   ├── models.ts          # User, Couple, etc.
│   │   │   └── inertia.d.ts
│   │   ├── Pages/                 # Inertia page components
│   │   │   ├── Auth/
│   │   │   │   ├── Login.vue
│   │   │   │   ├── Register.vue
│   │   │   │   ├── ForgotPassword.vue
│   │   │   │   ├── ResetPassword.vue
│   │   │   │   └── ChangePassword.vue
│   │   │   ├── Users/
│   │   │   │   ├── Index.vue      # Search/list
│   │   │   │   ├── Show.vue       # Profile view
│   │   │   │   ├── Edit.vue       # Edit form (tabbed)
│   │   │   │   ├── Chart.vue      # Genealogical chart
│   │   │   │   ├── Tree.vue       # Family tree
│   │   │   │   ├── Death.vue      # Death info
│   │   │   │   └── Marriages.vue  # Marriage list
│   │   │   ├── Couples/
│   │   │   │   ├── Show.vue
│   │   │   │   └── Edit.vue
│   │   │   ├── Birthdays/
│   │   │   │   └── Index.vue
│   │   │   ├── Backups/
│   │   │   │   └── Index.vue
│   │   │   └── Error.vue          # Error page
│   │   ├── Components/            # Reusable Vue components
│   │   │   ├── Layout/
│   │   │   │   ├── AppLayout.vue
│   │   │   │   ├── NavBar.vue
│   │   │   │   ├── Footer.vue
│   │   │   │   └── UserProfileLayout.vue
│   │   │   ├── User/
│   │   │   │   ├── ProfileCard.vue
│   │   │   │   ├── ProfilePhoto.vue
│   │   │   │   ├── FamilyInfo.vue
│   │   │   │   ├── ChildrenList.vue
│   │   │   │   ├── SiblingsList.vue
│   │   │   │   ├── ParentSpouseInfo.vue
│   │   │   │   └── ActionButtons.vue
│   │   │   ├── Family/
│   │   │   │   ├── FamilyTree.vue
│   │   │   │   ├── FamilyTreeNode.vue
│   │   │   │   ├── GenealogyChart.vue
│   │   │   │   ├── ChartPerson.vue
│   │   │   │   └── ChartSibling.vue
│   │   │   ├── Form/
│   │   │   │   ├── TextInput.vue
│   │   │   │   ├── SelectInput.vue
│   │   │   │   ├── DatePicker.vue
│   │   │   │   ├── RadioGroup.vue
│   │   │   │   ├── PhotoUpload.vue
│   │   │   │   ├── FormSection.vue
│   │   │   │   └── InputError.vue
│   │   │   ├── Map/
│   │   │   │   ├── LeafletMap.vue
│   │   │   │   └── LocationPicker.vue
│   │   │   ├── Modal/
│   │   │   │   ├── ConfirmModal.vue
│   │   │   │   └── FormModal.vue
│   │   │   └── UI/
│   │   │       ├── Button.vue
│   │   │       ├── Badge.vue
│   │   │       ├── Card.vue
│   │   │       ├── Tabs.vue
│   │   │       ├── Alert.vue
│   │   │       ├── Pagination.vue
│   │   │       └── LanguageSwitcher.vue
│   │   ├── Composables/           # Vue composables (hooks)
│   │   │   ├── useAuth.ts
│   │   │   ├── useFlash.ts
│   │   │   ├── useLocale.ts
│   │   │   ├── useUser.ts
│   │   │   └── useConfirmation.ts
│   │   ├── Stores/                # Pinia stores
│   │   │   ├── auth.ts
│   │   │   └── preferences.ts
│   │   └── Utils/                 # Utility functions
│   │       ├── date.ts
│   │       ├── formatters.ts
│   │       └── validators.ts
│   ├── css/
│   │   ├── app.scss               # Main stylesheet
│   │   ├── _variables.scss        # SCSS variables
│   │   ├── _tree.scss             # Family tree styles
│   │   └── _chart.scss            # Chart styles
│   ├── views/
│   │   └── app.blade.php          # Single Blade template (Inertia root)
│   └── lang/                      # Language files (JSON format for Vue)
│       ├── en.json
│       └── id.json
├── routes/
│   └── web.php                    # All routes (Inertia-based)
├── tests/
│   ├── Feature/                   # Laravel feature tests
│   ├── Unit/                      # Laravel unit tests
│   └── js/                        # Vue component tests
│       ├── Components/
│       ├── Pages/
│       └── setup.ts
├── package.json
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.js             # (Optional: if migrating to Tailwind)
└── composer.json
```

---

## 5. Feature Requirements

### 5.1 Feature Parity Matrix

All existing features must be maintained in the new architecture.

| Feature | Current | Target | Priority | Complexity |
|---------|---------|--------|----------|------------|
| User Registration | ✅ Blade | Vue Page | P0 | Low |
| User Login/Logout | ✅ Blade | Vue Page | P0 | Low |
| Password Reset | ✅ Blade | Vue Page | P0 | Low |
| Password Change | ✅ Blade | Vue Page | P0 | Low |
| User Profile View | ✅ Blade | Vue Page | P0 | Medium |
| User Profile Edit | ✅ Blade | Vue Page (Tabbed) | P0 | High |
| Photo Upload | ✅ jQuery | Vue Component | P0 | Medium |
| User Search | ✅ Blade | Vue Page | P0 | Low |
| Set Father/Mother | ✅ Form POST | Vue Modal/Form | P0 | Medium |
| Add Child | ✅ Form POST | Vue Modal/Form | P0 | Medium |
| Add Spouse | ✅ Form POST | Vue Modal/Form | P0 | Medium |
| Family Tree View | ✅ CSS + Blade | Vue Component | P0 | High |
| Genealogy Chart | ✅ CSS + Blade | Vue Component | P0 | High |
| Couple Management | ✅ Blade | Vue Page | P0 | Medium |
| Birthday List | ✅ Blade | Vue Page | P0 | Low |
| Cemetery Map | ✅ Leaflet + Blade | Vue + Leaflet | P0 | Medium |
| Language Switching | ✅ Session | Vue + Inertia | P0 | Medium |
| Admin Backups | ✅ Blade | Vue Page | P1 | Medium |
| Delete/Replace User | ✅ Modal + POST | Vue Modal | P1 | Medium |

### 5.2 Enhanced Features (Modernization Benefits)

| Feature | Description | Priority |
|---------|-------------|----------|
| **SPA Navigation** | No full page reloads, instant navigation | P0 |
| **Form Auto-save** | Draft saving for complex forms | P2 |
| **Real-time Validation** | Client-side validation with server sync | P0 |
| **Loading States** | Skeleton loaders, progress indicators | P0 |
| **Offline Indicator** | Show when connection is lost | P2 |
| **Keyboard Shortcuts** | Quick navigation and actions | P3 |
| **Search Autocomplete** | Instant search results as typing | P1 |
| **Interactive Tree** | Expandable/collapsible tree nodes | P1 |
| **Drag-drop Photo** | Modern photo upload experience | P2 |

### 5.3 Page Requirements

#### 5.3.1 Authentication Pages

**Login Page (`Auth/Login.vue`)**
- Email and password fields with validation
- "Remember me" checkbox
- Link to forgot password
- Link to registration
- Error display for invalid credentials
- Loading state during submission

**Registration Page (`Auth/Register.vue`)**
- Nickname (required)
- Email (required, unique)
- Password (required, min 8 chars)
- Password confirmation
- Gender selection (radio buttons)
- Terms acceptance (optional)
- Redirect to profile on success

**Forgot Password (`Auth/ForgotPassword.vue`)**
- Email input
- Success message display
- Rate limiting feedback

**Reset Password (`Auth/ResetPassword.vue`)**
- Token handling (URL parameter)
- New password input
- Confirmation input
- Redirect to login on success

**Change Password (`Auth/ChangePassword.vue`)**
- Current password verification
- New password input
- Confirmation input
- Success notification

#### 5.3.2 User Pages

**User Search/Index (`Users/Index.vue`)**
```vue
Props:
  - users: Paginated<User>
  - searchQuery: string

Features:
  - Search input with debounce
  - User cards grid display
  - Pagination controls
  - Empty state handling
  - Quick view popover (optional)
```

**User Profile (`Users/Show.vue`)**
```vue
Props:
  - user: User (with relationships)
  - father: User | null
  - mother: User | null
  - siblings: User[]
  - children: User[]
  - spouses: User[]
  - marriages: Couple[]

Features:
  - Profile photo display
  - Basic info card
  - Family relationships sidebar
  - Action buttons (edit, tree, chart)
  - Quick add family member modals
```

**User Edit (`Users/Edit.vue`)**
```vue
Props:
  - user: User
  - editableUser: User
  - allUsers: User[] (for select dropdowns)

Features:
  - Tabbed interface:
    1. Profile (name, DOB, gender, birth order)
    2. Death (date, cemetery, map location)
    3. Contact (email, phone, address, city)
    4. Account (change password - if own profile)
  - Photo upload component
  - Map location picker
  - Form validation
  - Unsaved changes warning
```

**Family Tree (`Users/Tree.vue`)**
```vue
Props:
  - user: User
  - descendants: NestedTree<User>
  - maxGenerations: number

Features:
  - Hierarchical tree visualization
  - Expandable/collapsible nodes
  - Click to navigate to profile
  - Generation indicators
  - Responsive layout
```

**Genealogy Chart (`Users/Chart.vue`)**
```vue
Props:
  - user: User
  - ancestors: AncestorData
  - children: User[]
  - siblings: User[]

Features:
  - Centered user display
  - Grandparents row (paternal + maternal)
  - Parents row
  - Siblings row
  - Children row
  - Click navigation
```

**Death Information (`Users/Death.vue`)**
```vue
Props:
  - user: User
  - cemeteryData: CemeteryLocation

Features:
  - Death date display
  - Cemetery name and address
  - Interactive Leaflet map
  - Edit link (if authorized)
```

**User Marriages (`Users/Marriages.vue`)**
```vue
Props:
  - user: User
  - marriages: Couple[]

Features:
  - List of marriages with dates
  - Spouse information
  - Children from each marriage
  - Links to couple details
```

#### 5.3.3 Couple Pages

**Couple Show (`Couples/Show.vue`)**
```vue
Props:
  - couple: Couple (with husband, wife, children)

Features:
  - Husband and wife profiles
  - Marriage/divorce dates
  - Children list
  - Edit button (if authorized)
```

**Couple Edit (`Couples/Edit.vue`)**
```vue
Props:
  - couple: Couple

Features:
  - Marriage date picker
  - Divorce date picker (optional)
  - Validation (divorce after marriage)
```

#### 5.3.4 Birthday Page

**Birthday Index (`Birthdays/Index.vue`)**
```vue
Props:
  - upcomingBirthdays: Birthday[]
  - daysAhead: number (60)

Features:
  - List grouped by date
  - Age calculation
  - Days remaining badge
  - Link to profile
```

#### 5.3.5 Admin Pages

**Backup Management (`Backups/Index.vue`)**
```vue
Props:
  - backups: BackupFile[]

Features:
  - List of backup files with dates/sizes
  - Create new backup button
  - Download buttons
  - Restore functionality with confirmation
  - Delete with confirmation
  - Upload backup file
```

---

## 6. Technical Requirements

### 6.1 Laravel Backend Requirements

#### 6.1.1 Laravel Version Upgrade

Upgrade from Laravel 8.x to Laravel 11.x:

```bash
# Required PHP version
PHP >= 8.2

# Key package updates
laravel/framework: ^11.0
inertiajs/inertia-laravel: ^1.0
```

#### 6.1.2 Inertia.js Server Setup

**Installation:**
```bash
composer require inertiajs/inertia-laravel
php artisan inertia:middleware
```

**HandleInertiaRequests Middleware:**
```php
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    protected $rootView = 'app';

    public function share(Request $request): array
    {
        return array_merge(parent::share($request), [
            'auth' => [
                'user' => $request->user() ? [
                    'id' => $request->user()->id,
                    'nickname' => $request->user()->nickname,
                    'name' => $request->user()->name,
                    'email' => $request->user()->email,
                    'gender_id' => $request->user()->gender_id,
                    'photo_path' => $request->user()->photo_path,
                    'is_admin' => is_system_admin($request->user()),
                ] : null,
            ],
            'flash' => [
                'success' => fn () => $request->session()->get('success'),
                'error' => fn () => $request->session()->get('error'),
            ],
            'locale' => app()->getLocale(),
            'translations' => $this->getTranslations(),
        ]);
    }

    protected function getTranslations(): array
    {
        $locale = app()->getLocale();
        $file = resource_path("lang/{$locale}.json");

        return file_exists($file)
            ? json_decode(file_get_contents($file), true)
            : [];
    }
}
```

#### 6.1.3 Controller Pattern (Inertia)

**Before (Blade):**
```php
public function show(User $user)
{
    return view('users.show', compact('user'));
}
```

**After (Inertia):**
```php
use Inertia\Inertia;
use Inertia\Response;

public function show(User $user): Response
{
    return Inertia::render('Users/Show', [
        'user' => $user->load(['father', 'mother', 'couples']),
        'siblings' => $user->siblings,
        'children' => $user->childs()->orderBy('birth_order')->get(),
        'spouses' => $user->gender_id === 1
            ? $user->wifes
            : $user->husbands,
    ]);
}
```

#### 6.1.4 Form Handling

**Form Request Validation (unchanged):**
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UpdateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'nickname' => ['required', 'string', 'max:255'],
            'name' => ['nullable', 'string', 'max:255'],
            'gender_id' => ['required', 'in:1,2'],
            'dob' => ['nullable', 'date'],
            'yob' => ['nullable', 'integer', 'min:1800', 'max:' . date('Y')],
            // ... other rules
        ];
    }
}
```

**Controller with Redirect:**
```php
public function update(UpdateUserRequest $request, User $user)
{
    $user->update($request->validated());

    return redirect()
        ->route('users.show', $user)
        ->with('success', __('user.updated'));
}
```

### 6.2 Frontend Requirements

#### 6.2.1 Vue 3 + TypeScript Setup

**vite.config.ts:**
```typescript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.scss', 'resources/js/app.ts'],
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
    ],
    resolve: {
        alias: {
            '@': '/resources/js',
        },
    },
});
```

**tsconfig.json:**
```json
{
    "compilerOptions": {
        "target": "ESNext",
        "module": "ESNext",
        "moduleResolution": "bundler",
        "strict": true,
        "jsx": "preserve",
        "sourceMap": true,
        "resolveJsonModule": true,
        "esModuleInterop": true,
        "lib": ["ESNext", "DOM"],
        "baseUrl": ".",
        "paths": {
            "@/*": ["resources/js/*"]
        },
        "types": ["vite/client"]
    },
    "include": ["resources/js/**/*.ts", "resources/js/**/*.vue"],
    "exclude": ["node_modules"]
}
```

#### 6.2.2 Application Entry Point

**resources/js/app.ts:**
```typescript
import '../css/app.scss';

import { createApp, h, DefineComponent } from 'vue';
import { createInertiaApp, Link, Head } from '@inertiajs/vue3';
import { createPinia } from 'pinia';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';
import { ZiggyVue } from 'ziggy-js';

const pinia = createPinia();

createInertiaApp({
    title: (title) => `${title} - Silsilah`,
    resolve: (name) => resolvePageComponent(
        `./Pages/${name}.vue`,
        import.meta.glob<DefineComponent>('./Pages/**/*.vue')
    ),
    setup({ el, App, props, plugin }) {
        createApp({ render: () => h(App, props) })
            .use(plugin)
            .use(pinia)
            .use(ZiggyVue)
            .component('Link', Link)
            .component('Head', Head)
            .mount(el);
    },
    progress: {
        color: '#4B5563',
    },
});
```

#### 6.2.3 Type Definitions

**resources/js/types/models.ts:**
```typescript
export interface User {
    id: string;
    nickname: string;
    name: string | null;
    gender_id: 1 | 2;
    gender: string;
    father_id: string | null;
    mother_id: string | null;
    parent_id: string | null;
    dob: string | null;
    yob: number | null;
    dod: string | null;
    yod: number | null;
    birth_order: number | null;
    email: string | null;
    address: string | null;
    city: string | null;
    phone: string | null;
    photo_path: string | null;
    manager_id: string | null;
    age: number | null;
    age_string: string | null;
    birthday: string | null;
    created_at: string;
    updated_at: string;

    // Relationships
    father?: User | null;
    mother?: User | null;
    childs?: User[];
    siblings?: User[];
    couples?: Couple[];
    wifes?: User[];
    husbands?: User[];
    metadata?: UserMetadata[];
}

export interface Couple {
    id: string;
    husband_id: string;
    wife_id: string;
    marriage_date: string | null;
    divorce_date: string | null;
    manager_id: string | null;
    created_at: string;
    updated_at: string;

    // Relationships
    husband?: User;
    wife?: User;
    childs?: User[];
}

export interface UserMetadata {
    id: string;
    user_id: string;
    key: string;
    value: string;
}

export interface CemeteryLocation {
    name: string | null;
    address: string | null;
    latitude: number | null;
    longitude: number | null;
}

export interface Birthday {
    user: User;
    date: string;
    age: number;
    days_remaining: number;
}

export interface BackupFile {
    filename: string;
    size: string;
    created_at: string;
}

export interface Paginated<T> {
    data: T[];
    current_page: number;
    last_page: number;
    per_page: number;
    total: number;
    from: number;
    to: number;
    links: PaginationLink[];
}

export interface PaginationLink {
    url: string | null;
    label: string;
    active: boolean;
}
```

**resources/js/types/inertia.d.ts:**
```typescript
import { PageProps as InertiaPageProps } from '@inertiajs/core';
import { User } from './models';

declare module '@inertiajs/core' {
    interface PageProps extends InertiaPageProps {
        auth: {
            user: User | null;
        };
        flash: {
            success: string | null;
            error: string | null;
        };
        locale: 'en' | 'id';
        translations: Record<string, string>;
    }
}
```

#### 6.2.4 Package Dependencies

**package.json:**
```json
{
    "name": "silsilah",
    "private": true,
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vue-tsc --noEmit && vite build",
        "test": "vitest",
        "test:coverage": "vitest --coverage",
        "lint": "eslint resources/js --ext .vue,.ts",
        "lint:fix": "eslint resources/js --ext .vue,.ts --fix",
        "typecheck": "vue-tsc --noEmit"
    },
    "dependencies": {
        "@inertiajs/vue3": "^1.0.0",
        "@vueuse/core": "^10.0.0",
        "axios": "^1.6.0",
        "bootstrap": "^5.3.0",
        "leaflet": "^1.9.0",
        "pinia": "^2.1.0",
        "vue": "^3.4.0",
        "vue-i18n": "^9.0.0",
        "ziggy-js": "^2.0.0"
    },
    "devDependencies": {
        "@types/leaflet": "^1.9.0",
        "@types/node": "^20.0.0",
        "@vitejs/plugin-vue": "^5.0.0",
        "@vue/test-utils": "^2.4.0",
        "eslint": "^8.0.0",
        "eslint-plugin-vue": "^9.0.0",
        "jsdom": "^24.0.0",
        "laravel-vite-plugin": "^1.0.0",
        "sass": "^1.70.0",
        "typescript": "^5.3.0",
        "vite": "^5.0.0",
        "vitest": "^1.0.0",
        "vue-tsc": "^1.8.0"
    }
}
```

---

## 7. Vue.js Component Architecture

### 7.1 Component Design Principles

1. **Single Responsibility**: Each component does one thing well
2. **Props Down, Events Up**: Unidirectional data flow
3. **Composition API**: Use `<script setup>` syntax
4. **TypeScript**: Full type safety for props and emits
5. **Scoped Styles**: CSS scoped to component (when not using Bootstrap)

### 7.2 Core Layout Components

#### 7.2.1 AppLayout.vue

```vue
<script setup lang="ts">
import { computed } from 'vue';
import { usePage, Head } from '@inertiajs/vue3';
import NavBar from './NavBar.vue';
import FlashMessages from '../UI/FlashMessages.vue';

interface Props {
    title?: string;
}

const props = withDefaults(defineProps<Props>(), {
    title: 'Silsilah',
});

const page = usePage();
const flash = computed(() => page.props.flash);
</script>

<template>
    <Head :title="title" />

    <div class="app-layout">
        <NavBar />

        <main class="container py-4">
            <FlashMessages :flash="flash" />
            <slot />
        </main>

        <footer class="footer mt-auto py-3 bg-light">
            <div class="container text-center">
                <span class="text-muted">
                    &copy; {{ new Date().getFullYear() }} Silsilah
                </span>
            </div>
        </footer>
    </div>
</template>
```

#### 7.2.2 NavBar.vue

```vue
<script setup lang="ts">
import { computed } from 'vue';
import { usePage, Link, router } from '@inertiajs/vue3';
import LanguageSwitcher from '../UI/LanguageSwitcher.vue';

const page = usePage();
const user = computed(() => page.props.auth.user);
const isAdmin = computed(() => user.value?.is_admin ?? false);

const logout = () => {
    router.post(route('logout'));
};
</script>

<template>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <Link :href="route('home')" class="navbar-brand">
                Silsilah
            </Link>

            <button
                class="navbar-toggler"
                type="button"
                data-bs-toggle="collapse"
                data-bs-target="#navbarNav"
            >
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <Link :href="route('users.index')" class="nav-link">
                            {{ $t('app.search') }}
                        </Link>
                    </li>
                    <li v-if="user" class="nav-item">
                        <Link :href="route('birthdays.index')" class="nav-link">
                            {{ $t('birthday.birthdays') }}
                        </Link>
                    </li>
                    <li v-if="isAdmin" class="nav-item">
                        <Link :href="route('backups.index')" class="nav-link">
                            {{ $t('backup.backup') }}
                        </Link>
                    </li>
                </ul>

                <ul class="navbar-nav">
                    <LanguageSwitcher />

                    <template v-if="user">
                        <li class="nav-item dropdown">
                            <a
                                class="nav-link dropdown-toggle"
                                href="#"
                                role="button"
                                data-bs-toggle="dropdown"
                            >
                                {{ user.nickname }}
                            </a>
                            <ul class="dropdown-menu dropdown-menu-end">
                                <li>
                                    <Link
                                        :href="route('users.show', user.id)"
                                        class="dropdown-item"
                                    >
                                        {{ $t('app.profile') }}
                                    </Link>
                                </li>
                                <li><hr class="dropdown-divider"></li>
                                <li>
                                    <button
                                        @click="logout"
                                        class="dropdown-item"
                                    >
                                        {{ $t('auth.logout') }}
                                    </button>
                                </li>
                            </ul>
                        </li>
                    </template>
                    <template v-else>
                        <li class="nav-item">
                            <Link :href="route('login')" class="nav-link">
                                {{ $t('auth.login') }}
                            </Link>
                        </li>
                        <li class="nav-item">
                            <Link :href="route('register')" class="nav-link">
                                {{ $t('auth.register') }}
                            </Link>
                        </li>
                    </template>
                </ul>
            </div>
        </div>
    </nav>
</template>
```

### 7.3 Form Components

#### 7.3.1 TextInput.vue

```vue
<script setup lang="ts">
import { computed } from 'vue';

interface Props {
    modelValue: string | number | null;
    label?: string;
    type?: string;
    error?: string;
    required?: boolean;
    disabled?: boolean;
    placeholder?: string;
}

const props = withDefaults(defineProps<Props>(), {
    type: 'text',
    required: false,
    disabled: false,
});

const emit = defineEmits<{
    'update:modelValue': [value: string];
}>();

const inputClasses = computed(() => ({
    'form-control': true,
    'is-invalid': !!props.error,
}));

const updateValue = (event: Event) => {
    emit('update:modelValue', (event.target as HTMLInputElement).value);
};
</script>

<template>
    <div class="mb-3">
        <label v-if="label" class="form-label">
            {{ label }}
            <span v-if="required" class="text-danger">*</span>
        </label>
        <input
            :type="type"
            :value="modelValue"
            :class="inputClasses"
            :disabled="disabled"
            :placeholder="placeholder"
            :required="required"
            @input="updateValue"
        />
        <div v-if="error" class="invalid-feedback">
            {{ error }}
        </div>
    </div>
</template>
```

#### 7.3.2 DatePicker.vue

```vue
<script setup lang="ts">
import { ref, watch } from 'vue';

interface Props {
    modelValue: string | null;
    label?: string;
    error?: string;
    required?: boolean;
    min?: string;
    max?: string;
}

const props = defineProps<Props>();

const emit = defineEmits<{
    'update:modelValue': [value: string | null];
}>();

const inputValue = ref(props.modelValue);

watch(() => props.modelValue, (newValue) => {
    inputValue.value = newValue;
});

const updateValue = (event: Event) => {
    const value = (event.target as HTMLInputElement).value;
    emit('update:modelValue', value || null);
};
</script>

<template>
    <div class="mb-3">
        <label v-if="label" class="form-label">
            {{ label }}
            <span v-if="required" class="text-danger">*</span>
        </label>
        <input
            type="date"
            :value="inputValue"
            class="form-control"
            :class="{ 'is-invalid': error }"
            :min="min"
            :max="max"
            :required="required"
            @input="updateValue"
        />
        <div v-if="error" class="invalid-feedback">
            {{ error }}
        </div>
    </div>
</template>
```

#### 7.3.3 PhotoUpload.vue

```vue
<script setup lang="ts">
import { ref, computed } from 'vue';
import { router } from '@inertiajs/vue3';

interface Props {
    currentPhoto: string | null;
    userId: string;
    defaultPhoto?: string;
}

const props = defineProps<Props>();

const fileInput = ref<HTMLInputElement>();
const isUploading = ref(false);
const previewUrl = ref<string | null>(null);

const displayUrl = computed(() => {
    return previewUrl.value || props.currentPhoto || props.defaultPhoto;
});

const triggerFileSelect = () => {
    fileInput.value?.click();
};

const handleFileChange = (event: Event) => {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (!file) return;

    // Preview
    previewUrl.value = URL.createObjectURL(file);

    // Upload
    isUploading.value = true;

    router.post(route('users.photo-upload', props.userId), {
        photo: file,
    }, {
        forceFormData: true,
        onSuccess: () => {
            isUploading.value = false;
        },
        onError: () => {
            isUploading.value = false;
            previewUrl.value = null;
        },
    });
};
</script>

<template>
    <div class="photo-upload text-center">
        <div class="photo-preview mb-3">
            <img
                :src="displayUrl"
                :alt="$t('user.photo')"
                class="rounded-circle"
                width="150"
                height="150"
            />
        </div>

        <input
            ref="fileInput"
            type="file"
            accept="image/*"
            class="d-none"
            @change="handleFileChange"
        />

        <button
            type="button"
            class="btn btn-outline-primary"
            :disabled="isUploading"
            @click="triggerFileSelect"
        >
            <span v-if="isUploading">
                {{ $t('app.uploading') }}...
            </span>
            <span v-else>
                {{ $t('user.change_photo') }}
            </span>
        </button>
    </div>
</template>

<style scoped>
.photo-preview img {
    object-fit: cover;
    border: 3px solid #dee2e6;
}
</style>
```

### 7.4 Family Visualization Components

#### 7.4.1 FamilyTree.vue

```vue
<script setup lang="ts">
import { computed } from 'vue';
import FamilyTreeNode from './FamilyTreeNode.vue';
import type { User } from '@/types/models';

interface TreeNode {
    user: User;
    children: TreeNode[];
    spouse?: User;
}

interface Props {
    rootUser: User;
    descendants: TreeNode;
    maxGenerations?: number;
}

const props = withDefaults(defineProps<Props>(), {
    maxGenerations: 5,
});
</script>

<template>
    <div class="family-tree">
        <div class="tree-container">
            <FamilyTreeNode
                :node="descendants"
                :generation="0"
                :max-generations="maxGenerations"
            />
        </div>
    </div>
</template>

<style scoped>
.family-tree {
    overflow-x: auto;
    padding: 20px;
}

.tree-container {
    display: flex;
    justify-content: center;
}
</style>
```

#### 7.4.2 FamilyTreeNode.vue

```vue
<script setup lang="ts">
import { computed } from 'vue';
import { Link } from '@inertiajs/vue3';
import type { User } from '@/types/models';

interface TreeNode {
    user: User;
    children: TreeNode[];
    spouse?: User;
}

interface Props {
    node: TreeNode;
    generation: number;
    maxGenerations: number;
}

const props = defineProps<Props>();

const hasChildren = computed(() => {
    return props.node.children && props.node.children.length > 0;
});

const showChildren = computed(() => {
    return props.generation < props.maxGenerations && hasChildren.value;
});

const getPhotoUrl = (user: User): string => {
    if (user.photo_path) {
        return `/storage/${user.photo_path}`;
    }
    return user.gender_id === 1
        ? '/images/icon-male.png'
        : '/images/icon-female.png';
};
</script>

<template>
    <div class="tree-node">
        <div class="node-content">
            <div class="person-card">
                <Link :href="route('users.show', node.user.id)">
                    <img
                        :src="getPhotoUrl(node.user)"
                        :alt="node.user.nickname"
                        class="person-photo"
                    />
                    <div class="person-name">{{ node.user.nickname }}</div>
                </Link>
                <div v-if="node.user.yob" class="person-year">
                    {{ node.user.yob }}
                </div>
            </div>

            <div v-if="node.spouse" class="spouse-card">
                <span class="spouse-connector">∞</span>
                <Link :href="route('users.show', node.spouse.id)">
                    <img
                        :src="getPhotoUrl(node.spouse)"
                        :alt="node.spouse.nickname"
                        class="person-photo"
                    />
                    <div class="person-name">{{ node.spouse.nickname }}</div>
                </Link>
            </div>
        </div>

        <div v-if="showChildren" class="children-container">
            <div class="tree-connector"></div>
            <div class="children-row">
                <FamilyTreeNode
                    v-for="child in node.children"
                    :key="child.user.id"
                    :node="child"
                    :generation="generation + 1"
                    :max-generations="maxGenerations"
                />
            </div>
        </div>
    </div>
</template>

<style scoped>
.tree-node {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 10px;
}

.node-content {
    display: flex;
    align-items: center;
    gap: 10px;
}

.person-card, .spouse-card {
    text-align: center;
    padding: 10px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.person-photo {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    object-fit: cover;
}

.person-name {
    font-weight: 500;
    margin-top: 5px;
}

.person-year {
    font-size: 0.8em;
    color: #666;
}

.spouse-connector {
    font-size: 1.5em;
    color: #666;
}

.children-container {
    margin-top: 20px;
    position: relative;
}

.tree-connector {
    width: 2px;
    height: 20px;
    background: #ccc;
    margin: 0 auto;
}

.children-row {
    display: flex;
    justify-content: center;
    gap: 20px;
    position: relative;
}

.children-row::before {
    content: '';
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    width: calc(100% - 40px);
    height: 2px;
    background: #ccc;
}
</style>
```

#### 7.4.3 LeafletMap.vue

```vue
<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

interface Props {
    latitude: number | null;
    longitude: number | null;
    zoom?: number;
    editable?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
    zoom: 15,
    editable: false,
});

const emit = defineEmits<{
    'update:latitude': [value: number];
    'update:longitude': [value: number];
}>();

const mapContainer = ref<HTMLDivElement>();
let map: L.Map | null = null;
let marker: L.Marker | null = null;

onMounted(() => {
    if (!mapContainer.value) return;

    const center: L.LatLngExpression = props.latitude && props.longitude
        ? [props.latitude, props.longitude]
        : [-0.87887, 117.4863]; // Default: Indonesia

    map = L.map(mapContainer.value).setView(center, props.zoom);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors',
    }).addTo(map);

    if (props.latitude && props.longitude) {
        marker = L.marker([props.latitude, props.longitude]).addTo(map);
    }

    if (props.editable) {
        map.on('click', (e: L.LeafletMouseEvent) => {
            const { lat, lng } = e.latlng;

            if (marker) {
                marker.setLatLng([lat, lng]);
            } else if (map) {
                marker = L.marker([lat, lng]).addTo(map);
            }

            emit('update:latitude', lat);
            emit('update:longitude', lng);
        });
    }
});

watch([() => props.latitude, () => props.longitude], ([lat, lng]) => {
    if (map && lat && lng) {
        map.setView([lat, lng], props.zoom);

        if (marker) {
            marker.setLatLng([lat, lng]);
        } else {
            marker = L.marker([lat, lng]).addTo(map);
        }
    }
});
</script>

<template>
    <div ref="mapContainer" class="leaflet-map"></div>
</template>

<style scoped>
.leaflet-map {
    height: 400px;
    width: 100%;
    border-radius: 8px;
}
</style>
```

### 7.5 Composables

#### 7.5.1 useAuth.ts

```typescript
import { computed } from 'vue';
import { usePage } from '@inertiajs/vue3';
import type { User } from '@/types/models';

export function useAuth() {
    const page = usePage();

    const user = computed<User | null>(() => page.props.auth.user);
    const isAuthenticated = computed(() => !!user.value);
    const isAdmin = computed(() => user.value?.is_admin ?? false);

    const can = (action: string, resource?: User | null): boolean => {
        if (!user.value) return false;
        if (isAdmin.value) return true;

        switch (action) {
            case 'edit':
                if (!resource) return false;
                return user.value.id === resource.id ||
                       user.value.id === resource.manager_id;
            case 'delete':
                if (!resource) return false;
                return user.value.id !== resource.id &&
                       (isAdmin.value || user.value.id === resource.manager_id);
            default:
                return false;
        }
    };

    return {
        user,
        isAuthenticated,
        isAdmin,
        can,
    };
}
```

#### 7.5.2 useLocale.ts

```typescript
import { computed } from 'vue';
import { usePage, router } from '@inertiajs/vue3';

export function useLocale() {
    const page = usePage();

    const locale = computed(() => page.props.locale);
    const translations = computed(() => page.props.translations);

    const t = (key: string, replacements: Record<string, string> = {}): string => {
        let translation = translations.value[key] || key;

        Object.entries(replacements).forEach(([k, v]) => {
            translation = translation.replace(`:${k}`, v);
        });

        return translation;
    };

    const setLocale = (newLocale: 'en' | 'id') => {
        router.visit(window.location.href, {
            data: { lang: newLocale },
            preserveState: true,
        });
    };

    return {
        locale,
        t,
        setLocale,
    };
}
```

#### 7.5.3 useFlash.ts

```typescript
import { ref, watch, computed } from 'vue';
import { usePage } from '@inertiajs/vue3';

export function useFlash() {
    const page = usePage();
    const visible = ref(true);

    const flash = computed(() => page.props.flash);
    const hasSuccess = computed(() => !!flash.value.success);
    const hasError = computed(() => !!flash.value.error);
    const hasFlash = computed(() => hasSuccess.value || hasError.value);

    watch(flash, () => {
        visible.value = true;

        // Auto-hide after 5 seconds
        if (hasSuccess.value) {
            setTimeout(() => {
                visible.value = false;
            }, 5000);
        }
    }, { deep: true });

    const dismiss = () => {
        visible.value = false;
    };

    return {
        flash,
        visible,
        hasSuccess,
        hasError,
        hasFlash,
        dismiss,
    };
}
```

---

## 8. API & Data Flow Design

### 8.1 Inertia Data Flow

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│   Vue Page   │────▶│  Inertia.js   │────▶│   Laravel    │
│  Component   │     │    Client     │     │  Controller  │
└──────────────┘     └───────────────┘     └──────────────┘
       ▲                    │                      │
       │                    │                      ▼
       │              XHR Request          ┌──────────────┐
       │              (JSON Props)         │   Eloquent   │
       │                    │              │    Models    │
       │                    ▼              └──────────────┘
       │              ┌───────────────┐           │
       │              │  HandleInertia │           │
       └──────────────│   Requests     │◀──────────┘
                      │  (Middleware)  │    Props
                      └───────────────┘
```

### 8.2 Shared Data (via HandleInertiaRequests)

Data shared on every request:

```php
[
    'auth' => [
        'user' => [
            'id' => 'uuid',
            'nickname' => 'string',
            'email' => 'string',
            'is_admin' => 'boolean',
            // ... minimal user data
        ],
    ],
    'flash' => [
        'success' => 'string|null',
        'error' => 'string|null',
    ],
    'locale' => 'en|id',
    'translations' => [/* all translations */],
]
```

### 8.3 Page-Specific Props Examples

**Users/Show.vue Props:**
```typescript
{
    user: User,
    father: User | null,
    mother: User | null,
    siblings: User[],
    children: User[],
    spouses: User[],
    marriages: Couple[],
    canEdit: boolean,
    canDelete: boolean,
}
```

**Users/Edit.vue Props:**
```typescript
{
    user: User,
    allUsers: { id: string; nickname: string }[],  // For select dropdowns
    cemeteryLocation: CemeteryLocation,
}
```

**Users/Tree.vue Props:**
```typescript
{
    user: User,
    descendants: TreeNode,  // Nested structure
    maxGenerations: number,
}
```

### 8.4 Form Submission Pattern

**Vue Component:**
```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3';

const form = useForm({
    nickname: props.user.nickname,
    name: props.user.name,
    gender_id: props.user.gender_id,
    dob: props.user.dob,
    // ...
});

const submit = () => {
    form.patch(route('users.update', props.user.id), {
        preserveScroll: true,
        onSuccess: () => {
            // Handle success (flash message shown automatically)
        },
    });
};
</script>
```

**Controller:**
```php
public function update(UpdateUserRequest $request, User $user)
{
    $this->authorize('edit', $user);

    $user->update($request->validated());

    return redirect()
        ->route('users.show', $user)
        ->with('success', __('user.updated'));
}
```

### 8.5 Family Action Endpoints

| Action | Method | Route | Controller Method |
|--------|--------|-------|-------------------|
| Set Father | POST | `/family-actions/{user}/set-father` | `setFather` |
| Set Mother | POST | `/family-actions/{user}/set-mother` | `setMother` |
| Add Child | POST | `/family-actions/{user}/add-child` | `addChild` |
| Add Wife | POST | `/family-actions/{user}/add-wife` | `addWife` |
| Add Husband | POST | `/family-actions/{user}/add-husband` | `addHusband` |
| Set Parent | POST | `/family-actions/{user}/set-parent` | `setParent` |

**Request Payload Pattern:**
```typescript
// Creating new family member
{
    new_member: true,
    nickname: string,
    gender_id: 1 | 2,
    yob?: number,
}

// Linking existing member
{
    new_member: false,
    user_id: string,
}
```

---

## 9. Database Considerations

### 9.1 No Schema Changes

The migration to Inertia.js + Vue.js does not require database schema changes. All existing tables, columns, and relationships remain unchanged:

| Table | Purpose | Status |
|-------|---------|--------|
| `users` | Family member profiles | Unchanged |
| `couples` | Marriage relationships | Unchanged |
| `user_metadata` | Flexible key-value storage | Unchanged |
| `password_resets` | Password reset tokens | Unchanged |

### 9.2 UUID Primary Keys

The application uses UUID primary keys for all models. This requires:

```typescript
// TypeScript: Ensure IDs are typed as strings
interface User {
    id: string;  // UUID, not number
    // ...
}
```

```vue
<!-- Vue: Use string comparison for IDs -->
<template>
    <div v-if="user.id === selectedId">...</div>
</template>
```

### 9.3 Query Optimization for Inertia

Since Inertia sends all props as JSON, optimize queries to avoid N+1 problems:

```php
// Good: Eager load relationships
public function show(User $user): Response
{
    $user->load([
        'father',
        'mother',
        'childs' => fn($q) => $q->orderBy('birth_order'),
        'couples.husband',
        'couples.wife',
    ]);

    return Inertia::render('Users/Show', [
        'user' => $user,
    ]);
}

// Bad: Lazy loading in props
return Inertia::render('Users/Show', [
    'user' => $user,
    'children' => $user->childs,  // N+1 if not eager loaded
]);
```

### 9.4 Data Transformation

Use Laravel API Resources when complex transformation is needed:

```php
// app/Http/Resources/UserResource.php
class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'nickname' => $this->nickname,
            'name' => $this->name,
            'gender' => $this->gender,
            'age' => $this->age,
            'age_string' => $this->age_string,
            'photo_url' => $this->photo_path
                ? Storage::url($this->photo_path)
                : null,
            'father' => new UserResource($this->whenLoaded('father')),
            'mother' => new UserResource($this->whenLoaded('mother')),
            // ...
        ];
    }
}
```

---

## 10. Migration Strategy

### 10.1 Migration Approach: Incremental Hybrid

We recommend an **incremental hybrid approach** that allows both Blade and Vue to coexist during migration:

```
Phase 1: Setup        ──▶ Weeks 1-2
Phase 2: Core Pages   ──▶ Weeks 3-6
Phase 3: Complex UI   ──▶ Weeks 7-10
Phase 4: Cleanup      ──▶ Weeks 11-12
Phase 5: Polish       ──▶ Weeks 13-14
```

### 10.2 Phase 1: Infrastructure Setup

**Tasks:**
- [ ] Upgrade Laravel to 11.x
- [ ] Install and configure Inertia.js server-side
- [ ] Set up Vite with Vue 3 and TypeScript
- [ ] Install frontend dependencies
- [ ] Create base layout components
- [ ] Set up HandleInertiaRequests middleware
- [ ] Configure Ziggy for route generation
- [ ] Set up Pinia store structure
- [ ] Configure Vitest for component testing
- [ ] Create TypeScript type definitions

**Deliverables:**
- Working Inertia.js setup
- Base AppLayout.vue and NavBar.vue
- TypeScript types for all models
- Development environment running

### 10.3 Phase 2: Core Pages Migration

**Priority Order:**

| Order | Page | Complexity | Dependencies |
|-------|------|------------|--------------|
| 1 | Login | Low | None |
| 2 | Register | Low | None |
| 3 | User Search/Index | Low | AppLayout |
| 4 | User Profile (Show) | Medium | ProfileCard, FamilyInfo |
| 5 | Birthdays | Low | None |
| 6 | Password Change | Low | None |
| 7 | Password Reset | Low | None |

**Migration Pattern per Page:**

1. Create Vue page component
2. Update controller to return `Inertia::render()`
3. Create necessary child components
4. Add TypeScript interfaces for props
5. Test functionality
6. Remove old Blade template

### 10.4 Phase 3: Complex UI Migration

**Tasks:**
- [ ] User Edit page (tabbed interface)
- [ ] Photo upload component
- [ ] Family Tree visualization
- [ ] Genealogy Chart visualization
- [ ] Cemetery Map integration
- [ ] Couple management pages
- [ ] Family action modals (add parent, child, spouse)
- [ ] Delete/Replace user functionality

### 10.5 Phase 4: Admin & Cleanup

**Tasks:**
- [ ] Backup management page
- [ ] Remove all Blade templates (except app.blade.php)
- [ ] Remove jQuery and related plugins
- [ ] Remove Bootstrap 3, update to Bootstrap 5
- [ ] Remove old SCSS files
- [ ] Update webpack.mix.js to vite.config.ts only
- [ ] Clean up unused npm packages

### 10.6 Phase 5: Polish & Optimization

**Tasks:**
- [ ] Performance optimization
- [ ] Loading states and skeleton screens
- [ ] Error boundary implementation
- [ ] Accessibility audit (WCAG 2.1)
- [ ] SEO meta tags
- [ ] PWA consideration
- [ ] Documentation update

### 10.7 Rollback Strategy

Each phase should be deployable independently. Keep Blade fallbacks until Vue replacements are fully tested:

```php
// Temporary hybrid controller
public function show(User $user)
{
    if (config('app.use_inertia')) {
        return Inertia::render('Users/Show', [...]);
    }

    return view('users.show', [...]);
}
```

---

## 11. Testing Strategy

### 11.1 Backend Testing (PHPUnit/Pest)

Existing tests should continue to pass with minimal modifications.

**Test Categories:**

| Category | Coverage Target | Tools |
|----------|-----------------|-------|
| Unit Tests | 90% | PHPUnit |
| Feature Tests | 80% | PHPUnit + Laravel Testing |
| Policy Tests | 100% | PHPUnit |
| Integration Tests | 70% | PHPUnit |

**Inertia-Specific Testing:**

```php
// tests/Feature/Users/ShowUserTest.php
use Inertia\Testing\AssertableInertia;

public function test_user_profile_page_returns_correct_props(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->get(route('users.show', $user));

    $response->assertInertia(fn (AssertableInertia $page) => $page
        ->component('Users/Show')
        ->has('user')
        ->has('siblings')
        ->has('children')
        ->where('user.id', $user->id)
    );
}
```

### 11.2 Frontend Testing (Vitest)

**Test Categories:**

| Category | Coverage Target | Tools |
|----------|-----------------|-------|
| Component Tests | 80% | Vitest + Vue Test Utils |
| Composable Tests | 90% | Vitest |
| Integration Tests | 60% | Vitest + MSW |

**Example Component Test:**

```typescript
// tests/js/Components/User/ProfileCard.spec.ts
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import ProfileCard from '@/Components/User/ProfileCard.vue';

describe('ProfileCard', () => {
    it('displays user nickname', () => {
        const wrapper = mount(ProfileCard, {
            props: {
                user: {
                    id: '1',
                    nickname: 'John Doe',
                    gender_id: 1,
                    photo_path: null,
                },
            },
        });

        expect(wrapper.text()).toContain('John Doe');
    });

    it('shows male icon when no photo and gender is male', () => {
        const wrapper = mount(ProfileCard, {
            props: {
                user: {
                    id: '1',
                    nickname: 'John',
                    gender_id: 1,
                    photo_path: null,
                },
            },
        });

        const img = wrapper.find('img');
        expect(img.attributes('src')).toContain('icon-male');
    });
});
```

**Example Composable Test:**

```typescript
// tests/js/Composables/useAuth.spec.ts
import { describe, it, expect, vi } from 'vitest';
import { useAuth } from '@/Composables/useAuth';

vi.mock('@inertiajs/vue3', () => ({
    usePage: () => ({
        props: {
            auth: {
                user: { id: '1', nickname: 'Test', is_admin: false },
            },
        },
    }),
}));

describe('useAuth', () => {
    it('returns authenticated user', () => {
        const { user, isAuthenticated } = useAuth();

        expect(isAuthenticated.value).toBe(true);
        expect(user.value?.nickname).toBe('Test');
    });
});
```

### 11.3 E2E Testing (Optional)

Consider Playwright for critical user flows:

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test('user can login and view profile', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL(/\/users\//);
    await expect(page.locator('.profile-card')).toBeVisible();
});
```

---

## 12. Security Considerations

### 12.1 Existing Security Measures (Maintained)

| Measure | Implementation | Status |
|---------|----------------|--------|
| CSRF Protection | Laravel middleware | ✅ Maintained |
| XSS Prevention | Vue's auto-escaping | ✅ Enhanced |
| SQL Injection | Eloquent ORM | ✅ Maintained |
| Authentication | Laravel Auth | ✅ Maintained |
| Authorization | Policies | ✅ Maintained |
| Password Hashing | bcrypt | ✅ Maintained |

### 12.2 Inertia-Specific Security

**CSRF Token Handling:**
Inertia automatically includes CSRF tokens in XHR requests.

**Props Security:**
Never expose sensitive data in Inertia props:

```php
// Bad: Exposing password hash
return Inertia::render('Users/Show', [
    'user' => $user,  // Includes password field
]);

// Good: Select only needed fields
return Inertia::render('Users/Show', [
    'user' => $user->only([
        'id', 'nickname', 'name', 'gender_id', 'photo_path',
    ]),
]);

// Better: Use API Resource
return Inertia::render('Users/Show', [
    'user' => new UserResource($user),
]);
```

### 12.3 Frontend Security

**v-html Usage:**
Avoid `v-html` with user content. If necessary, sanitize:

```vue
<!-- Dangerous -->
<div v-html="user.bio"></div>

<!-- Safe: Use text interpolation -->
<div>{{ user.bio }}</div>
```

**External Links:**
Always use `rel="noopener"` for external links:

```vue
<a :href="externalUrl" target="_blank" rel="noopener">...</a>
```

---

## 13. Performance Requirements

### 13.1 Performance Targets

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| First Contentful Paint | ~2.5s | <1.5s | Lighthouse |
| Time to Interactive | ~4.0s | <2.5s | Lighthouse |
| Largest Contentful Paint | ~3.5s | <2.0s | Lighthouse |
| Bundle Size (JS) | ~400KB | <200KB | Vite build |
| Lighthouse Score | ~65 | >85 | Lighthouse |

### 13.2 Optimization Strategies

**Code Splitting:**
```typescript
// vite.config.ts
export default defineConfig({
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    vendor: ['vue', '@inertiajs/vue3', 'pinia'],
                    leaflet: ['leaflet'],
                },
            },
        },
    },
});
```

**Lazy Loading:**
```typescript
// Large components loaded on demand
const FamilyTree = defineAsyncComponent(() =>
    import('@/Components/Family/FamilyTree.vue')
);
```

**Image Optimization:**
```vue
<template>
    <img
        :src="photoUrl"
        loading="lazy"
        :alt="user.nickname"
    />
</template>
```

**Partial Reloads:**
```typescript
// Only reload specific props
router.reload({ only: ['users'] });
```

---

## 14. Internationalization (i18n)

### 14.1 Translation File Migration

Migrate from PHP arrays to JSON format for frontend usage:

**Before (PHP):**
```php
// resources/lang/en/user.php
return [
    'name' => 'Name',
    'nickname' => 'Nickname',
    'gender' => 'Gender',
];
```

**After (JSON):**
```json
// resources/lang/en.json
{
    "user.name": "Name",
    "user.nickname": "Nickname",
    "user.gender": "Gender"
}
```

### 14.2 Vue i18n Integration

**Option 1: Custom Composable (Recommended)**

```typescript
// Composables/useLocale.ts
export function useLocale() {
    const page = usePage();
    const translations = computed(() => page.props.translations);

    const t = (key: string): string => {
        return translations.value[key] || key;
    };

    return { t };
}
```

**Usage:**
```vue
<script setup>
import { useLocale } from '@/Composables/useLocale';
const { t } = useLocale();
</script>

<template>
    <label>{{ t('user.name') }}</label>
</template>
```

**Option 2: Global Plugin**

```typescript
// plugins/i18n.ts
export const i18nPlugin = {
    install(app: App, options: { translations: Record<string, string> }) {
        app.config.globalProperties.$t = (key: string) => {
            return options.translations[key] || key;
        };
    },
};
```

### 14.3 Language Switching

```vue
<script setup lang="ts">
import { router } from '@inertiajs/vue3';
import { useLocale } from '@/Composables/useLocale';

const { locale } = useLocale();

const switchLanguage = (lang: 'en' | 'id') => {
    router.visit(window.location.pathname, {
        data: { lang },
        preserveState: true,
    });
};
</script>

<template>
    <div class="dropdown">
        <button class="btn btn-link dropdown-toggle">
            {{ locale === 'en' ? 'English' : 'Indonesia' }}
        </button>
        <ul class="dropdown-menu">
            <li>
                <button @click="switchLanguage('en')">English</button>
            </li>
            <li>
                <button @click="switchLanguage('id')">Indonesia</button>
            </li>
        </ul>
    </div>
</template>
```

---

## 15. Deployment Strategy

### 15.1 Build Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install PHP Dependencies
        run: composer install --no-dev --optimize-autoloader

      - name: Install Node Dependencies
        run: npm ci

      - name: Build Assets
        run: npm run build

      - name: Run Tests
        run: |
          php artisan test
          npm test

      - name: Deploy
        # Deploy steps...
```

### 15.2 Environment Configuration

```env
# Production .env additions
VITE_APP_NAME="${APP_NAME}"
```

### 15.3 Asset Versioning

Vite automatically handles cache busting with content hashing:

```html
<!-- Generated output -->
<script src="/build/assets/app-a1b2c3d4.js"></script>
<link href="/build/assets/app-e5f6g7h8.css" rel="stylesheet">
```

### 15.4 Server Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| PHP | 8.2 | 8.3 |
| Node.js | 18 LTS | 20 LTS |
| MySQL | 5.7 | 8.0 |
| Memory | 512MB | 1GB |

---

## 16. Risks & Mitigations

### 16.1 Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Breaking changes in Laravel 11 | Medium | High | Thorough testing, staged migration |
| Complex tree visualization bugs | High | Medium | Extensive component testing, visual regression tests |
| Performance regression | Low | High | Performance benchmarking, bundle analysis |
| TypeScript learning curve | Medium | Low | Team training, gradual adoption |
| Leaflet integration issues | Medium | Medium | Dedicated map component, fallback handling |

### 16.2 Project Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Extended timeline | Medium | Medium | Phased approach, clear milestones |
| Feature parity gaps | Low | High | Comprehensive feature checklist, UAT |
| User adaptation | Low | Low | Maintain familiar UI patterns |
| Rollback necessity | Low | High | Hybrid approach enables partial rollback |

### 16.3 Contingency Plans

**If timeline extends:**
- Prioritize core features (P0)
- Defer polish items (P2, P3)
- Consider extended hybrid phase

**If critical bugs discovered:**
- Maintain ability to serve Blade fallbacks
- Feature flags for gradual rollout
- Hotfix deployment process

---

## 17. Success Metrics

### 17.1 Technical Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Test Coverage (Backend) | >80% | PHPUnit coverage report |
| Test Coverage (Frontend) | >70% | Vitest coverage report |
| Lighthouse Performance | >85 | Lighthouse CI |
| Bundle Size Reduction | >30% | Vite build stats |
| Build Time | <60s | CI pipeline metrics |
| TypeScript Errors | 0 | `vue-tsc --noEmit` |

### 17.2 User Experience Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Page Load Time | <2s | Real User Monitoring |
| Time to Interactive | <3s | Lighthouse |
| Error Rate | <0.1% | Error tracking |
| User Complaints | 0 critical | Support tickets |

### 17.3 Development Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Code Review Time | <24h | PR metrics |
| Bug Fix Time | <48h | Issue tracking |
| Feature Velocity | Maintain/Improve | Sprint velocity |
| Developer Satisfaction | >80% | Team survey |

---

## 18. Appendices

### 18.1 Appendix A: Package.json (Complete)

```json
{
    "name": "silsilah",
    "private": true,
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vue-tsc --noEmit && vite build",
        "preview": "vite preview",
        "test": "vitest",
        "test:ui": "vitest --ui",
        "test:coverage": "vitest --coverage",
        "lint": "eslint resources/js --ext .vue,.ts,.tsx",
        "lint:fix": "eslint resources/js --ext .vue,.ts,.tsx --fix",
        "format": "prettier --write resources/js",
        "typecheck": "vue-tsc --noEmit"
    },
    "dependencies": {
        "@inertiajs/vue3": "^1.0.14",
        "@vueuse/core": "^10.7.0",
        "axios": "^1.6.2",
        "bootstrap": "^5.3.2",
        "leaflet": "^1.9.4",
        "pinia": "^2.1.7",
        "vue": "^3.4.0",
        "ziggy-js": "^2.0.0"
    },
    "devDependencies": {
        "@types/leaflet": "^1.9.8",
        "@types/node": "^20.10.0",
        "@typescript-eslint/eslint-plugin": "^6.13.0",
        "@typescript-eslint/parser": "^6.13.0",
        "@vitejs/plugin-vue": "^5.0.0",
        "@vue/test-utils": "^2.4.3",
        "@vitest/coverage-v8": "^1.1.0",
        "@vitest/ui": "^1.1.0",
        "eslint": "^8.55.0",
        "eslint-plugin-vue": "^9.19.0",
        "jsdom": "^23.0.0",
        "laravel-vite-plugin": "^1.0.0",
        "postcss": "^8.4.32",
        "prettier": "^3.1.0",
        "sass": "^1.69.5",
        "typescript": "^5.3.0",
        "vite": "^5.0.0",
        "vitest": "^1.1.0",
        "vue-tsc": "^1.8.25"
    }
}
```

### 18.2 Appendix B: Composer.json Updates

```json
{
    "require": {
        "php": "^8.2",
        "laravel/framework": "^11.0",
        "inertiajs/inertia-laravel": "^1.0",
        "tightenco/ziggy": "^2.0"
    },
    "require-dev": {
        "fakerphp/faker": "^1.23",
        "laravel/pint": "^1.13",
        "mockery/mockery": "^1.6",
        "nunomaduro/collision": "^8.0",
        "pestphp/pest": "^2.28",
        "pestphp/pest-plugin-laravel": "^2.2"
    }
}
```

### 18.3 Appendix C: File Migration Checklist

#### Controllers
- [ ] `HomeController` → Returns `Inertia::render('Dashboard')`
- [ ] `UsersController` → All methods return Inertia responses
- [ ] `FamilyActionsController` → Returns redirects with flash
- [ ] `CouplesController` → Returns Inertia responses
- [ ] `BirthdayController` → Returns Inertia response
- [ ] `BackupsController` → Returns Inertia responses
- [ ] Auth Controllers → Use Inertia auth scaffolding

#### Views (Blade → Vue)
- [ ] `layouts/app.blade.php` → Keep as Inertia root
- [ ] `users/show.blade.php` → `Pages/Users/Show.vue`
- [ ] `users/edit.blade.php` → `Pages/Users/Edit.vue`
- [ ] `users/search.blade.php` → `Pages/Users/Index.vue`
- [ ] `users/chart.blade.php` → `Pages/Users/Chart.vue`
- [ ] `users/tree.blade.php` → `Pages/Users/Tree.vue`
- [ ] `users/tree2.blade.php` → `Pages/Users/Tree.vue` (merged)
- [ ] `users/death.blade.php` → `Pages/Users/Death.vue`
- [ ] `users/marriages.blade.php` → `Pages/Users/Marriages.vue`
- [ ] `couples/show.blade.php` → `Pages/Couples/Show.vue`
- [ ] `couples/edit.blade.php` → `Pages/Couples/Edit.vue`
- [ ] `birthdays/index.blade.php` → `Pages/Birthdays/Index.vue`
- [ ] `backups/index.blade.php` → `Pages/Backups/Index.vue`
- [ ] `auth/login.blade.php` → `Pages/Auth/Login.vue`
- [ ] `auth/register.blade.php` → `Pages/Auth/Register.vue`
- [ ] `passwords/*.blade.php` → `Pages/Auth/*.vue`

#### Partials → Components
- [ ] `partials/nav.blade.php` → `Components/Layout/NavBar.vue`
- [ ] `users/partials/profile.blade.php` → `Components/User/ProfileCard.vue`
- [ ] `users/partials/childs.blade.php` → `Components/User/ChildrenList.vue`
- [ ] `users/partials/siblings.blade.php` → `Components/User/SiblingsList.vue`
- [ ] `users/partials/parent-spouse.blade.php` → `Components/User/ParentSpouseInfo.vue`
- [ ] `users/partials/action-buttons.blade.php` → `Components/User/ActionButtons.vue`
- [ ] `users/partials/update_photo.blade.php` → `Components/Form/PhotoUpload.vue`
- [ ] `users/partials/edit_*.blade.php` → Integrated in `Pages/Users/Edit.vue`

### 18.4 Appendix D: Route Definitions (Target)

```php
// routes/web.php

use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

// Public routes
Route::get('/', [UsersController::class, 'index'])->name('home');
Route::get('/profile-search', [UsersController::class, 'index'])->name('users.search');

// Authentication routes
Route::middleware('guest')->group(function () {
    Route::get('/login', [LoginController::class, 'create'])->name('login');
    Route::post('/login', [LoginController::class, 'store']);
    Route::get('/register', [RegisterController::class, 'create'])->name('register');
    Route::post('/register', [RegisterController::class, 'store']);
    Route::get('/forgot-password', [ForgotPasswordController::class, 'create'])->name('password.request');
    Route::post('/forgot-password', [ForgotPasswordController::class, 'store'])->name('password.email');
    Route::get('/reset-password/{token}', [ResetPasswordController::class, 'create'])->name('password.reset');
    Route::post('/reset-password', [ResetPasswordController::class, 'store'])->name('password.update');
});

// Authenticated routes
Route::middleware('auth')->group(function () {
    Route::post('/logout', [LoginController::class, 'destroy'])->name('logout');

    Route::get('/password/change', [ChangePasswordController::class, 'edit'])->name('password.change');
    Route::patch('/password/change', [ChangePasswordController::class, 'update']);

    // User routes
    Route::get('/users/{user}', [UsersController::class, 'show'])->name('users.show');
    Route::get('/users/{user}/edit', [UsersController::class, 'edit'])->name('users.edit');
    Route::patch('/users/{user}', [UsersController::class, 'update'])->name('users.update');
    Route::delete('/users/{user}', [UsersController::class, 'destroy'])->name('users.destroy');
    Route::get('/users/{user}/chart', [UsersController::class, 'chart'])->name('users.chart');
    Route::get('/users/{user}/tree', [UsersController::class, 'tree'])->name('users.tree');
    Route::get('/users/{user}/death', [UsersController::class, 'death'])->name('users.death');
    Route::patch('/users/{user}/photo-upload', [UsersController::class, 'photoUpload'])->name('users.photo-upload');
    Route::get('/users/{user}/marriages', [UsersController::class, 'marriages'])->name('users.marriages');

    // Family actions
    Route::post('/family-actions/{user}/set-father', [FamilyActionsController::class, 'setFather'])->name('family-actions.set-father');
    Route::post('/family-actions/{user}/set-mother', [FamilyActionsController::class, 'setMother'])->name('family-actions.set-mother');
    Route::post('/family-actions/{user}/add-child', [FamilyActionsController::class, 'addChild'])->name('family-actions.add-child');
    Route::post('/family-actions/{user}/add-wife', [FamilyActionsController::class, 'addWife'])->name('family-actions.add-wife');
    Route::post('/family-actions/{user}/add-husband', [FamilyActionsController::class, 'addHusband'])->name('family-actions.add-husband');
    Route::post('/family-actions/{user}/set-parent', [FamilyActionsController::class, 'setParent'])->name('family-actions.set-parent');

    // Couple routes
    Route::get('/couples/{couple}', [CouplesController::class, 'show'])->name('couples.show');
    Route::get('/couples/{couple}/edit', [CouplesController::class, 'edit'])->name('couples.edit');
    Route::patch('/couples/{couple}', [CouplesController::class, 'update'])->name('couples.update');

    // Birthdays
    Route::get('/birthdays', [BirthdayController::class, 'index'])->name('birthdays.index');

    // Admin routes
    Route::middleware('admin')->prefix('backups')->group(function () {
        Route::get('/', [BackupsController::class, 'index'])->name('backups.index');
        Route::post('/', [BackupsController::class, 'store'])->name('backups.store');
        Route::get('/{fileName}/dl', [BackupsController::class, 'download'])->name('backups.download');
        Route::post('/{fileName}/restore', [BackupsController::class, 'restore'])->name('backups.restore');
        Route::delete('/{fileName}', [BackupsController::class, 'destroy'])->name('backups.destroy');
        Route::post('/upload', [BackupsController::class, 'upload'])->name('backups.upload');
    });
});
```

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-02 | Engineering Team | Initial draft |

---

**End of Document**
