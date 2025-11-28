# 5 Custom Plugs Documentation

## Overview

This document provides detailed specifications for all 5 custom plugs used in the Tour & Travel Chatbot system.

---

## Plug A: Featured & My Packages

### Purpose
Retrieves featured packages and customer booking history from Google Sheets database.

### Integrations
- **Google Sheets API** (OAuth 2.0)
- **HubSpot CRM** (for customer lookup)

### Modes

#### Mode 1: Get Featured Packages
```
Function: get_featured(limit=5)
Input: None
Output: Array of featured packages

Gets top featured packages where is_featured=TRUE
```

**Response:**
```json
{
  "success": true,
  "packages": [
    {
      "id": "PKG001",
      "name": "Bali Getaway",
      "location": "Bali, Indonesia",
      "duration": 5,
      "price_min": 25000,
      "price_max": 35000,
      "thumbnail": "https://...",
      "highlights": "Temples, Beaches, Rice Paddies"
    }
  ],
  "total": 5
}
```

#### Mode 2: Get My Packages
```
Function: get_my_packages(email)
Input: Customer email address
Output: Array of customer bookings

Fetches all bookings for a customer by email
```

**Request:**
```json
{
  "email": "john@example.com"
}
```

**Response:**
```json
{
  "success": true,
  "bookings": [
    {
      "booking_id": "BOOK001",
      "package_name": "Bali Getaway",
      "travel_dates": "2025-05-15 to 2025-05-19",
      "status": "Confirmed",
      "amount": 70000
    }
  ],
  "total": 1
}
```

### Error Handling
- Invalid email format → Return error 400
- No bookings found → Return empty array
- Google Sheets API timeout → Return error 503

---

## Plug B: Search & Filter Packages

### Purpose
Filters packages based on user preferences (location, budget, duration).

### Integrations
- **Google Sheets API** (OAuth 2.0)
- **OpenAI API** (optional, for NLP preference parsing via Plug E)

### Search Algorithm

1. **Location Matching**
   - Keyword search in location and region_tag fields
   - Scoring: exact match (100) > partial match (50) > region match (30)

2. **Budget Filtering**
   - Filter: min_price <= user_budget <= max_price
   - Allow 10% buffer above max for flexibility

3. **Duration Matching**
   - Filter: abs(package_days - user_days) <= 1
   - Allow +/- 1 day tolerance

4. **Sorting**
   - Primary: Location relevance score (descending)
   - Secondary: Price (ascending)
   - Tertiary: Rating/Popularity (if available)

5. **Limiting Results**
   - Return maximum 10 packages
   - Prioritize featured packages if available

### Request
```json
{
  "location": "Bali",
  "budget": 30000,
  "days": 5,
  "month": "May"
}
```

### Response
```json
{
  "success": true,
  "query": {
    "location": "Bali",
    "budget": 30000,
    "days": 5
  },
  "results": [
    {
      "id": "PKG001",
      "name": "Bali Getaway",
      "location": "Bali, Indonesia",
      "duration": 5,
      "price": "25000-35000",
      "relevance_score": 95
    }
  ],
  "total_results": 3
}
```

### Error Handling
- Missing required fields → Return error 400
- No results found → Return empty results
- Invalid budget/days → Return error 400

---

## Plug C: CRM Lead & Booking Handler

### Purpose
Manages lead creation, CRM synchronization, booking generation, and payment processing.

### Integrations
- **HubSpot CRM API** (OAuth 2.0)
- **Google Sheets API** (OAuth 2.0)
- **Stripe API** (for payments)
- **Email Service** (SendGrid/Mailgun)

### Modes

#### Mode 1: Create Lead
```
Function: create_lead(name, email, phone, package_id, preferences)
```

**Process:**
1. Validate email and phone format
2. Create/Update contact in HubSpot
3. Add properties: package_selected, source, preferences
4. Log lead in Google Sheets "Leads" tab
5. Return lead_id and CRM contact_id

**Request:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91-9876543210",
  "package_id": "PKG001",
  "source": "featured_package",
  "preferences": {
    "location": "Bali",
    "budget": 30000,
    "days": 5
  }
}
```

**Response:**
```json
{
  "success": true,
  "lead_id": "LEAD001",
  "crm_id": "contact-123456",
  "email": "john@example.com",
  "status": "New"
}
```

#### Mode 2: Create Booking
```
Function: create_booking(lead_id, package_id, travelers, dates, email)
```

**Process:**
1. Fetch package details
2. Calculate costs (base × travelers + tax)
3. Generate invoice PDF
4. Create booking record in Sheets
5. Create Deal in HubSpot
6. Generate Stripe payment link
7. Send email with invoice + payment link

**Request:**
```json
{
  "lead_id": "LEAD001",
  "package_id": "PKG001",
  "number_of_travelers": 2,
  "travel_start_date": "2025-05-15",
  "travel_end_date": "2025-05-19",
  "email": "john@example.com"
}
```

**Response:**
```json
{
  "success": true,
  "booking_id": "BOOK001",
  "invoice_url": "https://...",
  "payment_link": "https://checkout.stripe.com/pay/...",
  "total_amount": 70000,
  "status": "Pending Payment"
}
```

#### Mode 3: Update Booking (Webhook)
```
Trigger: Stripe payment.intent.succeeded webhook
```

**Process:**
1. Verify webhook signature
2. Update booking status to "Confirmed"
3. Update deal stage in HubSpot to "Won"
4. Send confirmation email to customer
5. Log transaction in Sheets

---

## Plug D: OTP Verification (Brownie Points)

### Purpose
Securely verifies phone numbers using SMS OTP.

### Integrations
- **Twilio SMS API**
- **Session Storage** (Redis/encrypted)

### Flow

1. **Send OTP**
   - User enters phone number
   - Generate random 6-digit OTP
   - Send via Twilio SMS
   - Store in encrypted session (5-min TTL)
   - Return success message

2. **Verify OTP**
   - User enters OTP from SMS
   - Validate against session
   - Return success/failure
   - Clear session on success

### Request
```json
{
  "action": "send",
  "phone": "+91-9876543210"
}
```

### Response
```json
{
  "success": true,
  "message": "OTP sent to +91-9876543210",
  "expires_in_seconds": 300
}
```

### Verification
```json
{
  "action": "verify",
  "phone": "+91-9876543210",
  "otp": "123456"
}
```

### Response
```json
{
  "success": true,
  "verified": true,
  "message": "Phone number verified"
}
```

---

## Plug E: AI Helper (Brownie Points)

### Purpose
AI-powered features for itinerary generation, NLP, sentiment analysis, and FAQ.

### Integrations
- **OpenAI API** (GPT-4 or GPT-3.5-turbo)
- **Embeddings API** (for semantic search)

### Capabilities

#### 1. Itinerary Generation
```
Input: Package details + traveler profile
Output: Personalized day-by-day itinerary
```

**Example:**
```
Input:
- Package: Bali Getaway (5 days)
- Traveler: Family with kids, budget-conscious

Output:
- Day 1: Arrival, relaxation at beach
- Day 2: Family-friendly water sports
- Day 3: Cultural tour (temples, markets)
- Day 4: Adventure (rafting) with safety measures
- Day 5: Souvenir shopping, departure
```

#### 2. NLP Preference Parsing
```
Input: Natural language preference
Output: Structured search query
```

**Example:**
```
Input: "3 nights in Himalayas with snow, adventure activities, budget under 25k"

Output:
{
  "location": "Himachal Pradesh",
  "duration": 3,
  "budget": 25000,
  "activities": ["trekking", "snow", "adventure"],
  "season": "winter"
}
```

#### 3. Sentiment Analysis
```
Input: Customer feedback text
Output: Sentiment + themes + severity
```

**Example:**
```
Input: "Hotel was amazing but guide was not helpful"

Output:
{
  "sentiment": "mixed",
  "positive_themes": ["accommodation"],
  "negative_themes": ["guide_service"],
  "severity": "medium"
}
```

#### 4. FAQ Q&A
```
Input: Customer question
Output: Answer from knowledge base
```

**Example:**
```
Question: "What's your cancellation policy?"
Answer: "Full refund if cancelled 30+ days before travel, 
50% refund for 15-30 days, no refund within 14 days."
```

### Configuration
```yaml
Model: gpt-4 or gpt-3.5-turbo
Temperature: 0.7
Max Tokens: 500
Top-p: 0.9
Frequency Penalty: 0.0
Presence Penalty: 0.0
```

---

## Testing Checklist

- [ ] Plug A retrieves featured packages correctly
- [ ] Plug A fetches customer booking history
- [ ] Plug B searches and filters packages accurately
- [ ] Plug C creates leads in HubSpot
- [ ] Plug C generates invoices and payment links
- [ ] Plug C processes payment webhooks
- [ ] Plug D sends and verifies OTP via SMS
- [ ] Plug E generates personalized itineraries
- [ ] Plug E analyzes sentiment correctly
- [ ] All error cases are handled gracefully

---

**Last Updated:** November 28, 2025
**Version:** 1.0.0
