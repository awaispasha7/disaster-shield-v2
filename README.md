# DisasterShield - Core-1

A digital-first disaster recovery platform that connects homeowners with qualified contractors instantly.

## Overview

DisasterShield streamlines the insurance claim process by:
- Providing a mobile-friendly intake form for quick claim filing
- Automatically generating professional claim packets
- Matching homeowners with qualified local contractors
- Facilitating secure payments and project tracking

## Tech Stack

- **Frontend**: React 18 + TypeScript + Vite, Tailwind CSS, Radix UI (shadcn/ui)
- **Backend**: Supabase (PostgreSQL, Auth, Storage, Edge Functions)
- **Payments**: Stripe Checkout with dynamic product creation
- **Email**: Resend (via Supabase Edge Functions)
- **SMS**: Twilio integration
- **PDF Generation**: @react-pdf/renderer
- **Deployment**: Vercel (frontend) + Supabase (backend)

## Setup Instructions

### 1. Environment Variables

Copy `.env.example` to `.env.local` and fill in your credentials:

```bash
cp .env.example .env.local
```

### 2. Supabase Setup

1. Create a new Supabase project
2. Run the SQL migration in `/supabase/migrations/create_core_schema.sql`
3. Create storage buckets:
   - `media` (private) - for damage photos, videos, and voice notes
   - `claim-packets` (private) - for generated PDF documents (optional)
4. Update `.env.local` with your Supabase credentials

### 3. Base URL Configuration

The application uses the `VITE_APP_URL` environment variable to generate URLs for emails and redirects. Make sure to set this variable according to your deployment environment:

```bash
# For local development
VITE_APP_URL=http://localhost:5173

# For production
VITE_APP_URL=https://disaster-shield-v2.vercel.app
```

You can also use the provided script to update this value automatically:

```bash
# For local development
node scripts/update-env-url.js local

# For production
node scripts/update-env-url.js production
```

### 4. Email Configuration

The application uses Resend for sending emails through Supabase Edge Functions. By default, emails are mocked in development to avoid using up your Resend quota.

#### Deploy the Email Edge Function

```bash
# Deploy the email function to Supabase
npm run deploy:email-function

# Configure email secrets (API key, sender, and disable mock mode)
npm run set:email-secrets
```

#### Toggle between mock and real emails

```bash
# Enable mock mode (emails will be simulated but not actually sent)
npm run set:email-mock

# Disable mock mode (real emails will be sent using Resend)
npm run set:email-real
```

The email Edge Function logs all email attempts in the console, whether they're mocked or actually sent.

### 3. Stripe Setup

1. Create a Stripe account
2. Get your API keys from the Stripe dashboard
3. Set up a webhook endpoint pointing to `/api/payments/webhook`
4. Update `.env.local` with your Stripe credentials

### 4. Email Setup (Resend)

1. Create a Resend account
2. Verify your domain
3. Get your API key and update `.env.local`

### 5. Install Dependencies & Run

```bash
npm install
npm run dev
```

## Database Schema

The core schema includes:

- **profiles**: User accounts with role-based access (client, contractor, admin)
- **contractors**: Business information, service capabilities, and availability
- **projects**: Insurance claims and project tracking with status management
- **media**: Photo/video documentation storage with project associations
- **match_requests**: Contractor job invitations with acceptance tracking
- **contractor_estimates**: Dynamic pricing estimates with cost breakdowns
- **stripe_customers**: Payment customer mapping
- **stripe_orders**: Payment transaction records
- **stripe_subscriptions**: Subscription management (if applicable)
- **fnol_records**: First Notice of Loss submission tracking
- **insurance_companies**: Insurance provider configuration and API settings
- **notifications**: In-app notification system with read/unread status

## Storage Buckets

- **media**: Private bucket for damage photos, videos, and voice notes
- **claim-packets**: Private bucket for generated PDF documents (optional)

## Seed Data

Sample contractors can be loaded via CSV or through the admin interface. Example format:

```csv
company_name,contact_name,email,phone,service_areas,trades,capacity,calendly_url
Gulf Rapid Restore,James Miller,james@gulfrapid.com,+19415551212,"[""Sarasota"",""Charlotte""]","[""water_mitigation""]",active,https://calendly.com/gulfrapid/inspection
```

## Workflow

1. **Homeowner Intake**: 5-step wizard collecting damage details and media
2. **Packet Generation**: Automated PDF creation with photos and claim details
3. **Contractor Matching**: Algorithm scores contractors by location, trade fit, and availability
4. **Email Invitations**: Secure token-based accept/decline links sent to top contractors
5. **Job Assignment**: First contractor to accept gets the job, others are notified
6. **Payment Processing**: Stripe Checkout for security deposits
7. **Project Tracking**: Real-time status updates and communication tools

## Testing

Run the smoke test flow:
1. Complete intake form at `/intake`
2. Check packet generation works
3. Verify contractor matching emails
4. Test accept/decline flows
5. Confirm payment processing

## Key Features

### 🏠 For Homeowners (Clients)

#### Claim Filing & Management
- **Comprehensive Intake Form**: 5-step wizard for filing insurance claims with damage details
- **Media Upload**: Upload photos, videos, and voice notes documenting damage
- **Project Dashboard**: View all claims, track status, and manage projects
- **Project Portal**: Detailed project view with timeline, media gallery, and contractor information
- **Project Deletion**: Ability to delete claims if needed

#### Contractor Matching & Selection
- **Automated Matching**: Intelligent algorithm matches contractors based on:
  - Geographic proximity
  - Service area coverage
  - Trade expertise (water mitigation, mold, roofing, fire/smoke, general contracting)
  - Availability and capacity
- **Browse Contractors**: Search and filter contractors by trade, location, and ratings
- **Review Estimates**: Compare multiple contractor estimates side-by-side
- **Estimate Acceptance**: Accept or reject contractor estimates with detailed breakdowns
- **Manual Rematching**: Trigger contractor matching process from project portal

#### Payment Processing
- **Shopping Cart System**: Add multiple payment items to cart
- **Secure Payments**: Stripe Checkout integration with PCI compliance
- **Payment Groups**:
  - **Core Project**: Security deposit ($500 refundable), service fee ($99), repair cost estimate (dynamic)
  - **FNOL Generation**: Optional FNOL generation fee ($100)
- **Payment Status Tracking**: Real-time updates on payment completion
- **Payment History**: View all completed and pending payments

#### Insurance Integration (FNOL)
- **FNOL Generation**: First Notice of Loss document creation
- **Insurance Company Selection**: Choose from configured insurance providers
- **Multiple Submission Methods**: API, manual, email, or fax submission
- **Direct API Integration**: Automated submission to major insurance companies
- **Status Tracking**: Monitor FNOL submission status and acknowledgments
- **Document Download**: Access generated FNOL documents

### 🔨 For Contractors

#### Job Management
- **Browse Available Jobs**: View and filter available projects by location, trade, and status
- **Job Applications**: Apply to projects matching your expertise
- **Assigned Projects Dashboard**: View all assigned projects with status tracking
- **Project Details**: Access full project information including media and client details

#### Estimate System
- **Dynamic Pricing**: Set your own rates and pricing for each project
- **Detailed Estimates**: Submit comprehensive cost breakdowns with:
  - Total amount
  - Line-item breakdown
  - Timeline estimates
  - Notes and additional information
- **Estimate Management**: Update or modify estimates before client acceptance

#### Profile Management
- **Company Profile**: Manage business information, contact details, and branding
- **Service Areas**: Define geographic coverage areas
- **Trade Specialties**: Specify expertise areas (water mitigation, mold, roofing, etc.)
- **Capacity Management**: Set and update availability status
- **Calendly Integration**: Link scheduling calendars for inspections

### 👨‍💼 For Administrators

#### System Management
- **Admin Dashboard**: Comprehensive overview with key metrics:
  - Total projects and active projects
  - Contractor statistics
  - User management
  - FNOL submission tracking
- **User Management**: View and manage all platform users
- **System Analytics**: Track platform performance and usage

#### Insurance Company Management
- **Insurance Provider Configuration**: Add and manage insurance companies
- **API Integration Setup**: Configure API endpoints for automated FNOL submission
- **Submission Method Configuration**: Set up manual, email, fax, or API submission methods
- **Peril Support Configuration**: Define supported perils per insurance company

### 🔔 Communication & Notifications

#### Real-time Notifications
- **In-App Notification Bell**: Real-time notifications with unread count
- **Notification Types**:
  - Job posted
  - Contractor matched
  - Job accepted/declined
  - Payment received
  - Project updates
  - Inspection scheduled
- **Mark as Read**: Individual and bulk read status management
- **Click-to-Navigate**: Notifications link directly to relevant pages

#### Email Notifications
- **Transactional Emails**: Automated emails for key events:
  - Contractor job invitations
  - Job acceptance/decline confirmations
  - Payment completion notifications
  - Estimate received alerts
  - FNOL submission status updates
- **Email Templates**: Professional branded email templates
- **Mock Mode**: Development mode for testing without sending real emails

#### SMS Notifications
- **Twilio Integration**: SMS alerts for critical events
- **Contractor Matching**: SMS notifications for job opportunities

### 🔐 Security & Authentication

#### User Authentication
- **Email/Password Auth**: Secure authentication via Supabase Auth
- **Password Reset**: Forgot password flow with email verification
- **Email Verification**: Account verification system
- **Role-Based Access Control**: Client, Contractor, and Admin roles

#### Data Security
- **Row Level Security (RLS)**: Database-level access control
- **Private Storage**: Secure file storage with signed URLs
- **HMAC Tokens**: Secure token-based contractor acceptance links
- **Stripe Webhook Verification**: Signature verification for payment webhooks
- **Input Validation**: Zod schema validation for all forms

### 📱 User Experience

#### Responsive Design
- **Mobile-First**: Optimized for emergency situations on mobile devices
- **Progressive Web App**: Works seamlessly across all devices
- **Touch-Friendly**: Large buttons and intuitive gestures
- **Accessibility**: WCAG compliant design

#### Interface Features
- **Responsive Navigation**: Mobile-friendly navigation bar
- **Status Badges**: Visual status indicators throughout the app
- **Loading States**: Clear feedback during async operations
- **Error Handling**: User-friendly error messages and recovery
- **Toast Notifications**: Non-intrusive success/error messages

### 🔄 Workflow Automation

#### Automated Workflows
- **Complete Workflow System**: End-to-end automation using n8n workflows for:
  - Contractor matching and scoring
  - Email invitation sending
  - Match request creation
  - Status updates
- **SMS Matching Workflow**: Alternative SMS-based contractor matching
- **Workflow Error Handling**: Graceful error handling and retry logic

#### Business Logic
- **Contractor Scoring Algorithm**: Multi-factor scoring system
- **Top Contractor Selection**: Automatic selection of best-matched contractors
- **First-Come-First-Served**: First contractor to accept gets the job
- **Automatic Notifications**: Status updates sent to all relevant parties

## Security

- Row Level Security (RLS) on all database tables
- Private storage buckets with signed URLs
- HMAC-based tokens for contractor acceptance
- Stripe webhook signature verification
- Input validation with Zod schemas

## Development

The application follows modern React best practices:
- **Component Architecture**: Reusable UI components with Radix UI primitives
- **Type Safety**: Full TypeScript coverage with strict type checking
- **State Management**: React hooks and context for state management
- **Routing**: React Router for client-side navigation
- **Form Handling**: React Hook Form with Zod validation
- **Real-time Updates**: Supabase real-time subscriptions
- **Error Boundaries**: Comprehensive error handling and user feedback
- **Responsive Design**: Mobile-first approach with Tailwind CSS

## Deployment

Deploy to Vercel with environment variables configured in the project settings. Ensure Stripe webhook URLs are updated to point to your production domain.