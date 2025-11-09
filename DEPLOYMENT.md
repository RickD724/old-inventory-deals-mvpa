# Deployment Guide for DealVault

This guide covers deploying DealVault to various platforms.

---

## 🚀 Quick Deploy to Vercel (Recommended)

Vercel is the easiest way to deploy Next.js applications.

### Method 1: GitHub Integration (Recommended)

1. **Push your code to GitHub**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/your-username/dealvault.git
git push -u origin main
```

2. **Connect to Vercel**
   - Go to [vercel.com](https://vercel.com)
   - Click "Add New Project"
   - Import your GitHub repository
   - Vercel will auto-detect Next.js settings

3. **Configure Environment Variables**
   - In Vercel dashboard, go to Settings → Environment Variables
   - Add variables from `.env.example`
   - These will be available at build time

4. **Deploy**
   - Click "Deploy"
   - Vercel will build and deploy automatically
   - Future pushes to `main` will auto-deploy

### Method 2: Vercel CLI

```bash
npm install -g vercel
vercel login
vercel
```

Follow the prompts. Your site will be live in minutes!

---

## 🌐 Deploy to Netlify

1. **Build Command**: `npm run build`
2. **Publish Directory**: `.next`
3. **Environment Variables**: Add from `.env.example`

### Via GitHub

1. Push code to GitHub
2. Go to [netlify.com](https://netlify.com)
3. Click "New site from Git"
4. Select your repository
5. Configure build settings:
   - Build command: `npm run build`
   - Publish directory: `.next`
6. Add environment variables
7. Deploy!

---

## 📦 Deploy to AWS Amplify

1. **Push code to GitHub**

2. **In AWS Amplify Console**:
   - Connect your repository
   - Select branch (main)
   - Build settings (auto-detected for Next.js):
   ```yaml
   version: 1
   frontend:
     phases:
       preBuild:
         commands:
           - npm install
       build:
         commands:
           - npm run build
     artifacts:
       baseDirectory: .next
       files:
         - '**/*'
     cache:
       paths:
         - node_modules/**/*
   ```

3. **Environment Variables**:
   - Add from `.env.example` in Amplify Console

4. **Deploy**: Amplify will build and deploy automatically

---

## 🐳 Deploy with Docker

### Create Dockerfile

```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

### Build and Run

```bash
docker build -t dealvault .
docker run -p 3000:3000 dealvault
```

---

## 🖥️ Self-Hosted (Linux Server)

### Requirements
- Node.js 18+
- PM2 (process manager)
- Nginx (reverse proxy)

### Setup Steps

1. **Install Node.js**
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

2. **Install PM2**
```bash
sudo npm install -g pm2
```

3. **Clone and Build**
```bash
git clone https://github.com/your-username/dealvault.git
cd dealvault
npm install
npm run build
```

4. **Start with PM2**
```bash
pm2 start npm --name "dealvault" -- start
pm2 save
pm2 startup
```

5. **Configure Nginx**
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

6. **Enable SSL with Let's Encrypt**
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

---

## 🔧 Environment Variables

Before deploying, make sure to set these environment variables:

### Required for Production
```env
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

### Optional (when ready)
```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_key

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_xxx
STRIPE_SECRET_KEY=sk_live_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

---

## 📊 Post-Deployment Checklist

- [ ] Custom domain configured
- [ ] SSL certificate installed
- [ ] Environment variables set
- [ ] Analytics configured (Google Analytics, etc.)
- [ ] Error monitoring setup (Sentry, LogRocket)
- [ ] Database connected (Supabase/Firebase)
- [ ] Payment processing configured (Stripe)
- [ ] Email service configured
- [ ] Backup strategy in place
- [ ] Monitoring alerts configured

---

## 🔍 Monitoring & Analytics

### Recommended Tools

1. **Vercel Analytics** (built-in if using Vercel)
2. **Google Analytics**
```typescript
// Add to app/layout.tsx
<Script src="https://www.googletagmanager.com/gtag/js?id=GA_ID" />
```

3. **Sentry** (error tracking)
```bash
npm install @sentry/nextjs
```

4. **LogRocket** (session replay)

---

## 🚨 Troubleshooting

### Build Fails
- Check Node.js version (need 18+)
- Clear `.next` folder: `rm -rf .next`
- Clear node_modules: `rm -rf node_modules && npm install`

### Environment Variables Not Working
- Prefix browser variables with `NEXT_PUBLIC_`
- Restart dev server after changes
- Check deployment platform has variables set

### CSS Not Loading
- Ensure Tailwind is properly configured
- Check `globals.css` is imported in `layout.tsx`
- Clear browser cache

---

## 📈 Scaling Considerations

### When You Outgrow Single Server

1. **Database**: Move to managed Supabase/Firebase
2. **CDN**: Use Cloudflare or Vercel's Edge Network
3. **Caching**: Implement Redis for session storage
4. **Load Balancing**: Use AWS ELB or Nginx
5. **Media Storage**: Use S3 or Cloudinary for images

---

## 🎯 Performance Optimization

- Enable Next.js Image Optimization
- Implement ISR (Incremental Static Regeneration)
- Use dynamic imports for heavy components
- Enable Gzip/Brotli compression
- Optimize images before upload
- Use CDN for static assets

---

**Need help?** Open an issue on GitHub or contact support.
