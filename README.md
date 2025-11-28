# tour-travel-chatbot
A comprehensive tour and travel chatbot for Zoho SalesIQ with Google Sheets integration, HubSpot CRM sync, Stripe payments, and AI-powered features. Includes 5 custom plugs for featured packages, search, leads, OTP, and AI assistance.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [System Components](#system-components)
- [Database Schema](#database-schema)
- [5 Custom Plugs](#5-custom-plugs)
- [Chatbot Flows](#chatbot-flows)
- [Integration Guides](#integration-guides)
- [Setup Instructions](#setup-instructions)
- [API Reference](#api-reference)

## Overview

Tour & Travel Chatbot is an enterprise-grade AI-powered chatbot built specifically for the tour and travel industry. Deployed on **Zoho SalesIQ**, it provides seamless package discovery, booking management, and customer engagement through intelligent conversations.

### Key Benefits
- **Automated Package Recommendations**: AI-powered search filters packages by location, budget, and duration
- **Complete Lead Management**: Captures visitor information and syncs with HubSpot CRM
- **Instant Payments**: Integrated Stripe/Razorpay checkout for immediate payment processing
- **OTP Verification**: Secure phone number validation (Brownie Points)
- **AI-Enhanced Features**: Smart itinerary generation, sentiment analysis, and FAQ Q&A

## Features

### Core Features
✅ **Featured Packages Carousel** - Display trending packages with instant booking
✅ **Smart Package Search** - Filter by location, budget, and duration
✅ **Lead Capture** - Collect visitor details with validation
✅ **My Packages Dashboard** - Customers view their bookings and provide feedback
✅ **Invoice Generation** - Automated PDF invoices with booking details
✅ **Payment Integration** - Stripe & Razorpay checkout links
✅ **Feedback Management** - Collect and analyze customer reviews

### Brownie Point Features (Advanced)
✅ **Phone OTP Verification** - Two-factor verification via SMS
✅ **OAuth 2.0 Authentication** - Secure API integrations
✅ **AI Itinerary Generation** - LLM-powered day-by-day planning
✅ **Sentiment Analysis** - Automatic feedback categorization
✅ **Natural Language Processing** - Convert text preferences to structured data

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND: Zoho SalesIQ Chatbot            │
│                         (User Interface)
└─────────────────────────────────────────────────────────────────┘
                                 ↓
        ┌────────────────────────┼────────────────────────┐
        ↓                        ↓                        ↓
   [PLUG A]              [PLUG B]                [PLUG C]
   Featured &            Search &            Lead & Booking
   My Packages           Filter              Handler
        ↓                    ↓                        ↓
   ┌─────────────┬──────────────────┬──────────────────┬─────────────┐
   ↓             ↓                  ↓                  ↓             ↓
 Google Sheets  HubSpot CRM    Google Sheets    Stripe API    Email Service
 (Packages)     (Leads)        (Bookings)      (Payment)      (Notifications)
   ↓             ↓                  ↓                  ↓             ↓
   └─────────────┴──────────────────┴──────────────────┴─────────────┘
        ↓                        ↓                        ↓
   [PLUG D]              [PLUG E]
   OTP Verification      AI Helper
        ↓                        ↓
   Twilio SMS API        OpenAI/LLM API
```

## System Components

### 1. **Data Layer** - Google Sheets
- `Packages` - Tour package catalog
- `Leads` - Captured visitor information
- `Bookings` - Booking records and invoices
- `Feedback` - Customer reviews and ratings

### 2. **CRM Layer** - HubSpot
- Contact management
- Lead scoring and tracking
- Deal management (per booking)
- Activity history

### 3. **Payment Layer** - Stripe/Razorpay
- Payment link generation
- Transaction processing
- Webhook listeners for confirmations
- Invoice management

### 4. **AI Layer** - OpenAI API
- Itinerary personalization
- NLP for preference parsing
- Sentiment analysis
- FAQ answering

### 5. **Communication Layer**
- Email (transactional)
- SMS (OTP, confirmations)
- Bot notifications

## Database Schema

### Packages Table
```json
{
  "package_id": "PKG001",
  "package_name": "Bali Getaway",
  "location": "Bali, Indonesia",
  "duration_days": 5,
  "min_price": 25000,
  "max_price": 35000,
  "is_featured": true,
  "itinerary_summary": "5 days exploring temples, beaches...",
  "inclusions": "Flight, Hotel, Meals, Guide",
  "exclusions": "Alcoholic Beverages",
  "best_time": "April-October",
  "thumbnail_url": "https://...",
  "region_tag": "Southeast Asia",
  "created_date": "2025-01-15"
}
```

### Leads Table
```json
{
  "lead_id": "LEAD001",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91-9876543210",
  "selected_package_id": "PKG001",
  "preference_location": "Bali",
  "preference_budget": "30000",
  "preference_days": 5,
  "created_date": "2025-01-16",
  "status": "New",
  "source": "featured_package"
}
```

### Bookings Table
```json
{
  "booking_id": "BOOK001",
  "lead_id": "LEAD001",
  "package_id": "PKG001",
  "customer_name": "John Doe",
  "customer_email": "john@example.com",
  "number_of_travelers": 2,
  "travel_start_date": "2025-05-15",
  "travel_end_date": "2025-05-19",
  "total_cost": 70000,
  "subtotal": 50000,
  "tax": 9000,
  "discount": 0,
  "invoice_number": "INV-2025-001",
  "invoice_url": "https://...",
  "payment_status": "Completed",
  "payment_method": "Card",
  "payment_gateway_ref": "ch_1234567890",
  "created_date": "2025-01-16"
}
```


## 5 Custom Plugs

### Plug A: Featured & My Packages
Retrieves featured packages and customer booking history

**API Endpoints:**
- `GET /plugs/packages/featured` - Returns top featured packages
- `GET /plugs/packages/my-packages?email=xxx@email.com` - Returns customer bookings

**Response:**
```json
{
  "packages": [{
    "id": "PKG001",
    "name": "Bali Getaway",
    "location": "Bali, Indonesia",
    "duration": 5,
    "price_range": "₹25,000 - ₹35,000",
    "thumbnail": "https://..."
  }],
  "total": 5
}
```

### Plug B: Search & Filter
Filters packages based on user preferences

**Parameters:**
- `location` (string) - Destination name
- `budget` (number) - Max budget in INR
- `days` (number) - Preferred duration

**Algorithm:**
1. Search location by keyword matching and region tags
2. Filter by price range
3. Match duration +/- 1 day tolerance
4. Sort by relevance (location match weighted highest)
5. Return max 10 results

### Plug C: CRM Lead & Booking Handler
Manages lead creation, booking, and payment processing

**Modes:**
- `create_lead` - Create contact in HubSpot
- `create_booking` - Generate invoice and payment link
- `update_booking` - Update after payment

**Integrations:**
- HubSpot CRM (OAuth 2.0)
- Google Sheets (Append booking record)
- Stripe API (Create checkout session)
- Email Service (Send invoice)

### Plug D: OTP Verification (Brownie Points)
Verifies phone numbers with SMS OTP

**Flow:**
1. User enters phone number
2. Generate random 6-digit OTP
3. Send via Twilio SMS API
4. Store in encrypted session (5-min TTL)
5. User enters OTP for verification
6. Validate and proceed

### Plug E: AI Helper (Brownie Points)
AI-powered features using OpenAI API

**Capabilities:**
1. **Itinerary Generation** - Personalize day-by-day plans
2. **NLP Preference Parsing** - Convert text to structured filters
3. **Sentiment Analysis** - Analyze feedback (positive/negative/neutral)
4. **FAQ Answering** - Respond to policy questions

**Configuration:**
- Model: GPT-4 or GPT-3.5-turbo
- Temperature: 0.7
- Max tokens: 500

## Chatbot Flows

### Flow 1: Featured Packages
```
Welcome
  ↓
User: [Featured Packages]
  ↓
Bot: Display 5-package carousel
  ↓
User: [Get Details]
  ↓
Capture: Name → Email → Phone (OTP)
  ↓
Bot: Show full package details
  ↓
User: [Proceed to Book]
  ↓
Capture: Travelers, Travel Dates
  ↓
Generate Invoice → Send Payment Link
  ↓
Payment Complete → Confirmation Email
```

### Flow 2: Find a Package
```
Welcome
  ↓
User: [Find Package]
  ↓
Bot: Ask Destination
  ↓
Bot: Ask Budget Range
  ↓
Bot: Ask Number of Days
  ↓
[Plug B] Search & Filter
  ↓
Bot: Display matching packages (carousel)
  ↓
[Same as Flow 1 from "Get Details"]
```

### Flow 3: My Packages
```
Welcome
  ↓
User: [My Packages]
  ↓
Bot: Ask Email
  ↓
[Plug A] Fetch bookings
  ↓
Bot: Display booking list
  ↓
User: [Feedback] or [Book Again]
  ↓
If Feedback:
  - Ask Rating (1-5)
  - Ask Comments
  - [Plug E] Analyze sentiment
  - Store in Sheets
↓
If Book Again:
  - [Same booking flow]
```

## Integration Guides

### Google Sheets OAuth 2.0 Setup
1. Create Google Cloud Project
2. Enable Sheets API
3. Create Service Account
4. Generate JSON key
5. Configure in Plug settings:
   ```
   GOOGLE_SHEETS_API_KEY=<json_key>
   SHEET_ID=<spreadsheet_id>
   ```

### HubSpot CRM Integration
1. Create HubSpot account
2. Generate API key from Settings → API Keys
3. Configure in Plug C:
   ```
   HUBSPOT_API_KEY=<api_key>
   HUBSPOT_PORTAL_ID=<portal_id>
   ```

### Stripe Payment Gateway
1. Create Stripe account
2. Get API keys from Dashboard
3. Configure in Plug C:
   ```
   STRIPE_SECRET_KEY=<secret_key>
   STRIPE_PUBLISHABLE_KEY=<publishable_key>
   ```

### Twilio SMS for OTP
1. Create Twilio account
2. Get Account SID and Auth Token
3. Configure in Plug D:
   ```
   TWILIO_ACCOUNT_SID=<sid>
   TWILIO_AUTH_TOKEN=<token>
   TWILIO_PHONE_NUMBER=<+1xxx...>
   ```

### OpenAI API for AI Features
1. Create OpenAI account
2. Generate API key
3. Configure in Plug E:
   ```
   OPENAI_API_KEY=<api_key>
   OPENAI_MODEL=gpt-4
   ```

## Setup Instructions

### Prerequisites
- Zoho SalesIQ account
- Google account (for Sheets)
- HubSpot account (free tier)
- Stripe or Razorpay account
- Twilio account (optional, for OTP)
- OpenAI account (optional, for AI features)

### Installation Steps
1. Clone this repository
2. Copy `.env.example` to `.env`
3. Fill in all API keys and credentials
4. Create Google Sheets database (see Database Schema)
5. Import Plug A, B, C, D, E into Zoho SalesIQ
6. Configure webhook URLs for Stripe
7. Test each flow in staging
8. Deploy to production

## API Reference

### Lead Creation
```bash
POST /api/leads/create
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91-9876543210",
  "package_id": "PKG001",
  "source": "featured_package"
}

Response: 201 Created
{
  "lead_id": "LEAD001",
  "crm_id": "contact-123456",
  "status": "New"
}
```

### Booking Creation
```bash
POST /api/bookings/create
Content-Type: application/json

{
  "lead_id": "LEAD001",
  "package_id": "PKG001",
  "number_of_travelers": 2,
  "travel_start_date": "2025-05-15",
  "email": "john@example.com"
}

Response: 201 Created
{
  "booking_id": "BOOK001",
  "invoice_url": "https://...",
  "payment_link": "https://checkout.stripe.com/pay/...",
  "status": "Pending Payment"
}
```

## Directory Structure
```
tour-travel-chatbot/
├── README.md
├── docs/
│   ├── PLUGS.md
│   ├── FLOWS.md
│   ├── INTEGRATIONS.md
│   └── API.md
├── plugs/
│   ├── plug-a-featured-packages.yaml
│   ├── plug-b-search-filter.yaml
│   ├── plug-c-crm-booking.yaml
│   ├── plug-d-otp-verification.yaml
│   └── plug-e-ai-helper.yaml
├── sheets/
│   ├── packages-template.csv
│   ├── leads-template.csv
│   └── bookings-template.csv
├── configs/
│   ├── .env.example
│   └── zoho-saleiq-config.json
└── examples/
    ├── conversation-flow-featured.md
    ├── conversation-flow-search.md
    └── conversation-flow-mypackages.md
```

## License
MIT License - See LICENSE file

## Support & Contact
- GitHub Issues: Report bugs and feature requests
- Documentation: See /docs folder
- Contact: [Your Contact Info]

---

**Last Updated:** November 28, 2025
**Version:** 1.0.0
**Status:** Production Ready ✅
