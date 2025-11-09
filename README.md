# DealVault - Aged Inventory Marketplace

**Connect motivated car dealerships with retail buyers hunting for below-market deals on aged new inventory.**

---

## 🚗 Concept Overview

DealVault is a two-sided marketplace that helps dealerships move aged inventory (90+ days) while maintaining dealer anonymity until real buyer interest is shown. Buyers get access to hidden "manager's specials" that aren't publicly listed elsewhere.

### For Dealers:
- Move aged inventory quietly without public discounting
- Stay anonymous until a lead is qualified (protects brand/image)
- Pay small monthly subscription instead of large ad budgets
- Only receive serious leads (buyers pay to unlock contact info)

### For Buyers:
- Access to hidden deals normally not available online
- Filter by make, model, discount level, and location
- Pay per unlock or subscribe for unlimited access
- Direct connection to motivated dealers

---

## 🎯 Core Features (MVP)

### Dealer Portal
- ✅ Sign up / Verify dealer license
- ✅ Post inventory manually or via CSV upload
- ✅ View listing analytics (views, unlocks)
- ✅ Manage active listings

### Buyer Portal
- ✅ Browse deals with advanced filtering
- ✅ View VIN-masked listings with pricing
- ✅ Unlock dealer contact info (pay-per-unlock or subscription)
- ✅ See "Deal Heat Score" based on aging + discount

### Admin Dashboard
- ✅ Approve dealer accounts
- ✅ Manage all listings
- ✅ View platform analytics

---

## 🛠️ Tech Stack

- **Frontend**: Next.js 14 + TypeScript + Tailwind CSS
- **Icons**: Lucide React
- **Backend** (Future): Supabase or Firebase
- **Payments**: Stripe
- **Hosting**: Vercel (recommended)

---

## 📦 Installation

### Prerequisites
- Node.js 18+ 
- npm or yarn

### Setup Steps

1. **Clone the repository**
```bash
git clone https://github.com/your-username/dealvault.git
cd dealvault
```

2. **Install dependencies**
```bash
npm install
# or
yarn install
```

3. **Run the development server**
```bash
npm run dev
# or
yarn dev
```

4. **Open in browser**
Navigate to [http://localhost:3000](http://localhost:3000)

---

## 📁 Project Structure

```
dealvault/
├── app/
│   ├── page.tsx                 # Homepage
│   ├── layout.tsx               # Root layout
│   ├── globals.css              # Global styles
│   ├── browse/
│   │   └── page.tsx             # Browse deals page
│   ├── dealer/
│   │   ├── signup/
│   │   │   └── page.tsx         # Dealer registration
│   │   ├── dashboard/
│   │   │   └── page.tsx         # Dealer dashboard
│   │   └── login/
│   │       └── page.tsx         # Dealer login
│   └── buyer/
│       ├── signup/
│       │   └── page.tsx         # Buyer registration
│       └── login/
│           └── page.tsx         # Buyer login
├── data/
│   └── mock-inventory.json      # Mock data for development
├── public/                      # Static assets
├── package.json                 # Dependencies
├── tsconfig.json               # TypeScript config
├── tailwind.config.js          # Tailwind config
├── next.config.js              # Next.js config
└── README.md                   # This file
```

---

## 💰 Revenue Model

### Buyer Pricing
- **Per-Unlock**: $20 per dealer contact unlock
- **Monthly Subscription**: $49/month for unlimited unlocks
  - Unlimited dealer contact unlocks
  - Advanced filtering and alerts
  - Weekly "Hot 5 Deals" newsletter

### Dealer Pricing
- **Professional Plan**: $149/month
  - Up to 20 active listings
  - Anonymous dealer profile
  - CSV bulk upload
  - Analytics dashboard
  - Only receive qualified leads

---

## 🔑 Key Differentiators

1. **Anonymity First**: Dealers remain anonymous until buyers pay to unlock
2. **Quality Leads**: Buyers must pay, filtering out tire-kickers
3. **Deal Heat Score**: Algorithm highlights best deals based on aging + discount
4. **No Public Discounting**: Protects dealer brand and factory relationships
5. **Win-Win**: Dealers move units, buyers get below-market pricing

---

## 📊 Mock Data

The project includes comprehensive mock data in `/data/mock-inventory.json`:
- 6 sample vehicle listings (Porsche, BMW, Mercedes, Audi, Lexus)
- 2 sample dealer profiles
- Full vehicle details (specs, features, pricing, aging info)
- Complete make/model/state reference data

---

## 🚀 Deployment

### Deploy to Vercel (Recommended)

1. Push your code to GitHub
2. Connect your repo to [Vercel](https://vercel.com)
3. Configure environment variables (if needed)
4. Deploy!

### Deploy to Other Platforms
- **Netlify**: Connect GitHub repo
- **AWS Amplify**: Follow AWS deployment guide
- **Self-hosted**: Build with `npm run build` and serve with `npm start`

---

## 🔮 Future Enhancements

### Phase 2 Features
- [ ] Supabase backend integration
- [ ] Stripe payment processing
- [ ] Real-time chat between buyers/dealers
- [ ] Email notifications and alerts
- [ ] Mobile app (React Native)

### Phase 3 Features
- [ ] "Deal Sniffer" AI algorithm for deal recommendations
- [ ] Dealer performance analytics
- [ ] Buyer credit pre-qualification
- [ ] Integration with dealer DMS systems
- [ ] Weekly "Hot 5 Deals" automated newsletter

---

## 🎨 Design Philosophy

- **Clean & Modern**: Gradient backgrounds, card-based layout
- **Dark Theme**: Premium feel for luxury vehicle market
- **Mobile-First**: Fully responsive design
- **Performance**: Optimized with Next.js 14 features
- **Accessibility**: Semantic HTML, proper contrast ratios

---

## 📝 Environment Variables

Create a `.env.local` file in the root directory:

```env
# Supabase (when ready to integrate)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# Stripe (when ready to integrate)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your_stripe_key
STRIPE_SECRET_KEY=your_stripe_secret_key

# Other configs
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

---

## 🤝 Contributing

Contributions welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙋 Support

For questions or issues:
- Open an issue on GitHub
- Email: support@dealvault.com (placeholder)

---

## ✨ Credits

Built with:
- [Next.js](https://nextjs.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Lucide Icons](https://lucide.dev/)
- [TypeScript](https://www.typescriptlang.org/)

---

**DealVault** - Moving inventory, creating value. 🚗💨
