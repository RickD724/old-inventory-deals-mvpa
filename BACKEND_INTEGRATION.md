# Backend Integration Guide

This guide shows you how to connect DealVault to a real backend (Supabase recommended).

---

## 🗄️ Database Schema

### Tables You'll Need

#### 1. dealers
```sql
CREATE TABLE dealers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  dealership_name TEXT NOT NULL,
  contact_name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  phone TEXT,
  address TEXT,
  city TEXT,
  state TEXT,
  zip_code TEXT,
  dealer_license TEXT UNIQUE NOT NULL,
  verified BOOLEAN DEFAULT false,
  plan TEXT DEFAULT 'professional',
  listings_max INTEGER DEFAULT 20,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 2. buyers
```sql
CREATE TABLE buyers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  phone TEXT,
  plan TEXT DEFAULT 'free',
  unlocks_remaining INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 3. listings
```sql
CREATE TABLE listings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  dealer_id UUID REFERENCES dealers(id) ON DELETE CASCADE,
  
  -- Vehicle Info
  year INTEGER NOT NULL,
  make TEXT NOT NULL,
  model TEXT NOT NULL,
  trim TEXT,
  vin_last_4 TEXT NOT NULL,
  vin_full TEXT NOT NULL, -- Encrypted
  
  -- Pricing
  msrp DECIMAL(10,2) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  discount DECIMAL(10,2) GENERATED ALWAYS AS (msrp - price) STORED,
  discount_percent DECIMAL(5,2) GENERATED ALWAYS AS ((msrp - price) / msrp * 100) STORED,
  
  -- Details
  days_in_stock INTEGER NOT NULL,
  color TEXT,
  interior_color TEXT,
  mileage INTEGER,
  transmission TEXT,
  engine TEXT,
  horsepower INTEGER,
  drivetrain TEXT,
  fuel_type TEXT,
  mpg TEXT,
  
  -- Location
  location TEXT NOT NULL,
  city TEXT,
  state TEXT,
  zip_code TEXT,
  
  -- Metadata
  features JSONB,
  notes TEXT,
  image_urls TEXT[],
  
  -- Metrics
  views INTEGER DEFAULT 0,
  unlocks INTEGER DEFAULT 0,
  deal_heat INTEGER GENERATED ALWAYS AS (
    CASE 
      WHEN days_in_stock > 150 AND discount_percent > 9 THEN 95 + (discount_percent::int % 5)
      WHEN days_in_stock > 120 AND discount_percent > 8 THEN 85 + (discount_percent::int % 10)
      ELSE 75 + (discount_percent::int % 10)
    END
  ) STORED,
  
  -- Status
  status TEXT DEFAULT 'active',
  listed_date DATE DEFAULT CURRENT_DATE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 4. unlocks
```sql
CREATE TABLE unlocks (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  buyer_id UUID REFERENCES buyers(id),
  listing_id UUID REFERENCES listings(id),
  dealer_id UUID REFERENCES dealers(id),
  unlock_type TEXT, -- 'subscription' or 'single'
  amount_paid DECIMAL(10,2),
  unlocked_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 5. subscriptions
```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL, -- buyer_id or dealer_id
  user_type TEXT NOT NULL, -- 'buyer' or 'dealer'
  plan TEXT NOT NULL,
  status TEXT DEFAULT 'active', -- 'active', 'cancelled', 'expired'
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  current_period_start TIMESTAMPTZ,
  current_period_end TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 🔐 Authentication Setup

### Using Supabase Auth

1. **Install Supabase Client**
```bash
npm install @supabase/supabase-js @supabase/auth-helpers-nextjs
```

2. **Create Supabase Client**

`lib/supabase.ts`:
```typescript
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

3. **Update Login Pages**

Replace mock login in `app/dealer/login/page.tsx`:
```typescript
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault()
  
  const { data, error } = await supabase.auth.signInWithPassword({
    email: formData.email,
    password: formData.password,
  })
  
  if (error) {
    alert('Login failed: ' + error.message)
    return
  }
  
  // Redirect to dashboard
  window.location.href = '/dealer/dashboard'
}
```

---

## 💳 Stripe Integration

### 1. Install Stripe
```bash
npm install stripe @stripe/stripe-js
```

### 2. Create Stripe Client

`lib/stripe.ts`:
```typescript
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
})
```

### 3. Create Checkout Session

`app/api/create-checkout-session/route.ts`:
```typescript
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'

export async function POST(req: Request) {
  const { priceId, userId } = await req.json()
  
  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [
      {
        price: priceId, // Create in Stripe Dashboard
        quantity: 1,
      },
    ],
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
    client_reference_id: userId,
  })
  
  return NextResponse.json({ sessionId: session.id })
}
```

### 4. Handle Webhooks

`app/api/webhooks/stripe/route.ts`:
```typescript
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'
import { supabase } from '@/lib/supabase'

export async function POST(req: Request) {
  const body = await req.text()
  const sig = req.headers.get('stripe-signature')!
  
  let event
  
  try {
    event = stripe.webhooks.constructEvent(
      body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    return NextResponse.json({ error: 'Webhook Error' }, { status: 400 })
  }
  
  // Handle the event
  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object
      // Update subscription in database
      break
      
    case 'customer.subscription.deleted':
      // Handle cancellation
      break
  }
  
  return NextResponse.json({ received: true })
}
```

---

## 🔄 API Routes

### Get Listings

`app/api/listings/route.ts`:
```typescript
import { NextResponse } from 'next/server'
import { supabase } from '@/lib/supabase'

export async function GET(req: Request) {
  const { searchParams } = new URL(req.url)
  const make = searchParams.get('make')
  const minDiscount = searchParams.get('minDiscount')
  const location = searchParams.get('location')
  
  let query = supabase
    .from('listings')
    .select('*')
    .eq('status', 'active')
    .order('deal_heat', { ascending: false })
  
  if (make) query = query.eq('make', make)
  if (minDiscount) query = query.gte('discount_percent', parseFloat(minDiscount))
  if (location) query = query.ilike('location', `%${location}%`)
  
  const { data, error } = await query
  
  if (error) {
    return NextResponse.json({ error: error.message }, { status: 500 })
  }
  
  return NextResponse.json(data)
}
```

### Unlock Dealer Contact

`app/api/unlock/route.ts`:
```typescript
import { NextResponse } from 'next/server'
import { supabase } from '@/lib/supabase'
import { stripe } from '@/lib/stripe'

export async function POST(req: Request) {
  const { listingId, buyerId, paymentMethodId } = await req.json()
  
  // Create payment intent
  const paymentIntent = await stripe.paymentIntents.create({
    amount: 2000, // $20.00
    currency: 'usd',
    payment_method: paymentMethodId,
    confirm: true,
  })
  
  if (paymentIntent.status !== 'succeeded') {
    return NextResponse.json({ error: 'Payment failed' }, { status: 400 })
  }
  
  // Get full listing details including dealer contact
  const { data: listing } = await supabase
    .from('listings')
    .select(`
      *,
      dealer:dealers (
        dealership_name,
        contact_name,
        email,
        phone
      )
    `)
    .eq('id', listingId)
    .single()
  
  // Record the unlock
  await supabase.from('unlocks').insert({
    buyer_id: buyerId,
    listing_id: listingId,
    dealer_id: listing.dealer_id,
    unlock_type: 'single',
    amount_paid: 20.00,
  })
  
  // Increment unlock counter
  await supabase
    .from('listings')
    .update({ unlocks: listing.unlocks + 1 })
    .eq('id', listingId)
  
  return NextResponse.json({
    dealer: listing.dealer,
    vehicleDetails: listing
  })
}
```

---

## 🖼️ Image Upload

### Using Supabase Storage

```typescript
// app/dealer/listings/new/page.tsx

const handleImageUpload = async (file: File) => {
  const fileExt = file.name.split('.').pop()
  const fileName = `${Math.random()}.${fileExt}`
  const filePath = `listings/${fileName}`
  
  const { data, error } = await supabase.storage
    .from('listing-images')
    .upload(filePath, file)
  
  if (error) {
    console.error('Upload error:', error)
    return null
  }
  
  const { data: { publicUrl } } = supabase.storage
    .from('listing-images')
    .getPublicUrl(filePath)
  
  return publicUrl
}
```

---

## 📧 Email Notifications

### Using Resend

1. **Install**
```bash
npm install resend
```

2. **Send Welcome Email**
```typescript
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

async function sendWelcomeEmail(to: string, name: string) {
  await resend.emails.send({
    from: 'DealVault <hello@dealvault.com>',
    to,
    subject: 'Welcome to DealVault',
    html: `<h1>Welcome ${name}!</h1><p>Thanks for joining DealVault...</p>`
  })
}
```

---

## 🔍 Search with Algolia (Optional)

For advanced search:

```bash
npm install algoliasearch
```

```typescript
import algoliasearch from 'algoliasearch'

const client = algoliasearch(
  process.env.ALGOLIA_APP_ID!,
  process.env.ALGOLIA_API_KEY!
)

const index = client.initIndex('listings')

// Index a listing
await index.saveObject({
  objectID: listing.id,
  year: listing.year,
  make: listing.make,
  model: listing.model,
  price: listing.price,
  location: listing.location,
})
```

---

## 🔒 Security Best Practices

1. **Row Level Security (RLS)** in Supabase
```sql
-- Dealers can only see their own listings
CREATE POLICY "Dealers can view own listings"
ON listings FOR SELECT
USING (auth.uid() = dealer_id);

-- Buyers can see active listings
CREATE POLICY "Buyers can view active listings"
ON listings FOR SELECT
USING (status = 'active');
```

2. **API Rate Limiting**
```typescript
// middleware.ts
import { ratelimit } from '@/lib/redis'

export async function middleware(req: Request) {
  const ip = req.headers.get('x-forwarded-for') || 'anonymous'
  const { success } = await ratelimit.limit(ip)
  
  if (!success) {
    return new Response('Too many requests', { status: 429 })
  }
}
```

3. **Environment Variables**
- Never commit `.env` files
- Use different keys for dev/prod
- Rotate keys regularly

---

## 📊 Analytics Integration

### Google Analytics
```typescript
// app/layout.tsx
import Script from 'next/script'

<Script
  src={`https://www.googletagmanager.com/gtag/js?id=${process.env.NEXT_PUBLIC_GA_ID}`}
  strategy="afterInteractive"
/>
<Script id="google-analytics" strategy="afterInteractive">
  {`
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', '${process.env.NEXT_PUBLIC_GA_ID}');
  `}
</Script>
```

---

## 🧪 Testing

### Unit Tests (Jest)
```typescript
// __tests__/utils/calculateDealHeat.test.ts
import { calculateDealHeat } from '@/lib/utils'

describe('calculateDealHeat', () => {
  it('calculates heat score correctly', () => {
    const score = calculateDealHeat(150, 10.0)
    expect(score).toBeGreaterThan(90)
  })
})
```

### E2E Tests (Playwright)
```typescript
// tests/browse.spec.ts
import { test, expect } from '@playwright/test'

test('can browse and filter deals', async ({ page }) => {
  await page.goto('/browse')
  await page.click('text=Show Filters')
  await page.selectOption('select[name="make"]', 'Porsche')
  await expect(page.locator('.deal-card')).toHaveCount(2)
})
```

---

## 🚀 Deployment Checklist

- [ ] Database schema created
- [ ] Authentication working
- [ ] Stripe integration tested
- [ ] Email notifications configured
- [ ] Image uploads working
- [ ] API routes secured
- [ ] Environment variables set
- [ ] Analytics tracking
- [ ] Error monitoring (Sentry)
- [ ] Performance optimized
- [ ] SEO meta tags
- [ ] SSL certificate
- [ ] Domain configured
- [ ] Backups automated

---

**Ready to build!** Follow this guide step-by-step to connect your backend. 🚀
