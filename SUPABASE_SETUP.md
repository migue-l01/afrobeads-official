# Supabase Setup & Debugging Guide

## 1. SUPABASE CLIENT CONFIGURATION

### File: `src/integrations/supabase/client.ts`

Replace placeholder values with your actual Supabase credentials:

```typescript
const supabaseUrl = "https://YOUR_PROJECT_ID.supabase.co";
const supabaseAnonKey = "YOUR_PUBLIC_ANON_KEY";
```

**To find your credentials:**
1. Log in to https://app.supabase.com
2. Go to Project Settings → API
3. Copy the URL (under "Project URL")
4. Copy the anon key (under "Project API keys" → "anon" → "public")

---

## 2. DATABASE SETUP

### Required Table: `products`

Run this SQL in Supabase SQL Editor:

```sql
-- Create products table
CREATE TABLE IF NOT EXISTS products (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  name TEXT NOT NULL,
  category TEXT NOT NULL,
  price NUMERIC NOT NULL,
  currency TEXT DEFAULT 'Ksh',
  description TEXT,
  image_url TEXT,
  is_new BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Enable Row Level Security
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Public READ access (customers can see all active products)
CREATE POLICY "public_read_products" ON products
  FOR SELECT USING (true);

-- Authenticated users can INSERT products (logged-in admins)
CREATE POLICY "authenticated_insert_products" ON products
  FOR INSERT WITH CHECK (auth.role() = 'authenticated');

-- Authenticated users can UPDATE products
CREATE POLICY "authenticated_update_products" ON products
  FOR UPDATE USING (auth.role() = 'authenticated');

-- Authenticated users can DELETE products
CREATE POLICY "authenticated_delete_products" ON products
  FOR DELETE USING (auth.role() = 'authenticated');
```

### Verify table exists:
1. Go to Supabase Dashboard → Tables
2. You should see `products` table with all columns listed above

---

## 3. STORAGE SETUP

### Create Storage Bucket: `product-images`

1. Go to Supabase Dashboard → Storage
2. Click "New Bucket"
3. Name: `product-images`
4. Make Public: **YES**
5. Click Create

### Set Bucket Policies:

Go to `product-images` bucket → Policies, add:

```sql
-- Allow public read access
CREATE POLICY "public_read_images" ON storage.objects
  FOR SELECT USING (bucket_id = 'product-images');

-- Allow authenticated users to upload
CREATE POLICY "authenticated_upload_images" ON storage.objects
  FOR INSERT WITH CHECK (bucket_id = 'product-images' AND auth.role() = 'authenticated');

-- Allow authenticated users to delete
CREATE POLICY "authenticated_delete_images" ON storage.objects
  FOR DELETE USING (bucket_id = 'product-images' AND auth.role() = 'authenticated');
```

### Verify bucket is public:
- Settings → Storage → Bucket Options → Public (should be enabled)

---

## 4. AUTHENTICATION (OPTIONAL)

If you want to enable admin authentication:

1. Go to Supabase → Authentication → Providers
2. Enable "Email" provider
3. Create test user: Authentication → Users → Add User
4. Enable auth in `AdminDashboard.tsx` by uncommenting the `checkAuth()` call

For now, the admin panel works without authentication (public access).

---

## 5. COMMON ERRORS & FIXES

### Error: "Failed to fetch products"
**Cause:** Supabase URL or key is incorrect

**Fix:**
```bash
# Check browser console (F12 → Console tab)
# Look for "Fetched products:" logs
# If not appearing, verify credentials in src/integrations/supabase/client.ts
```

### Error: "Storage bucket not found"
**Cause:** `product-images` bucket doesn't exist in Supabase Storage

**Fix:**
1. Go to Supabase Dashboard → Storage
2. Create bucket named `product-images`
3. Make it public
4. Retry upload

### Error: "Row level security policy violation"
**Cause:** RLS policies are too restrictive

**Fix:**
Option 1: Disable RLS temporarily (for testing only):
```sql
-- In Supabase SQL Editor
ALTER TABLE products DISABLE ROW LEVEL SECURITY;
ALTER TABLE storage.objects DISABLE ROW LEVEL SECURITY;
```

Option 2: Check that policies are properly set (see Database Setup above)

### Error: "401 Unauthorized or 403 Forbidden"
**Cause:** Invalid anon key or CORS issue

**Fix:**
1. Verify anon key is correct in `client.ts`
2. Go to Supabase → Authentication → URL Configuration
3. Add your Netlify domain to "Redirect URLs" and "Site URL"
   - Development: `http://localhost:8080`
   - Production: `https://afrobeads.netlify.app`

### Error: "Network request failed / Failed to fetch"
**Cause:** CORS or network issues

**Fix:**
1. Check browser console (F12) for full error
2. Ensure Netlify domain is in Supabase → Authentication → URL Configuration
3. Check that Supabase project is not paused

### Error: "Cannot read property 'data' of undefined"
**Cause:** Supabase client is not properly imported

**Fix:**
```typescript
// ✅ CORRECT
import { supabase } from "@/integrations/supabase/client";

// ❌ WRONG
const supabase = createClient(...) // Duplicate client
```

Ensure you're importing the shared client, not creating new ones.

---

## 6. DEBUGGING CHECKLIST

### Before Testing the App:
- [ ] Replace `YOUR_PROJECT_ID` in `src/integrations/supabase/client.ts`
- [ ] Replace `YOUR_PUBLIC_ANON_KEY` in `src/integrations/supabase/client.ts`
- [ ] Created `products` table in Supabase
- [ ] Created `product-images` storage bucket
- [ ] Set up RLS policies (or disabled temporarily)
- [ ] Added Netlify domain to Supabase → Authentication → URL Configuration

### While Testing:
1. Open browser console (F12)
2. Check for these logs:
   ```
   ✅ "ProductCarousel: Loading products..."
   ✅ "Fetched products:" followed by array
   ✅ "Image uploaded successfully:" followed by URL
   ```

3. Look for errors like:
   ```
   ❌ "Error loading products:"
   ❌ "Error uploading image:"
   ❌ "supabase is not defined"
   ❌ "bucket not found"
   ```

### Test Flow:
1. **Customer Page**
   - Go to http://localhost:8080/
   - Check console for "Fetched products:"
   - If no products, go to Admin and add one

2. **Admin Panel**
   - Go to http://localhost:8080/admin
   - Add product with name, category, price, image
   - Check console for "Inserting product into Supabase:"
   - Check for success/error toast message
   - Verify image appears in Supabase Storage bucket

3. **Real-time Sync**
   - Open 2 browser windows
   - Add product in admin window
   - Check if it appears in customer window without refresh

---

## 7. QUICK SETUP SCRIPT

```bash
# Terminal commands to set up database (paste in Supabase SQL Editor):

-- 1. Create table
CREATE TABLE products (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  name TEXT NOT NULL,
  category TEXT NOT NULL,
  price NUMERIC NOT NULL,
  currency TEXT DEFAULT 'Ksh',
  description TEXT,
  image_url TEXT,
  is_new BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- 2. Enable RLS
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- 3. Create policies
CREATE POLICY "public_read" ON products FOR SELECT USING (true);
CREATE POLICY "authenticated_insert" ON products FOR INSERT WITH CHECK (auth.role() = 'authenticated');
CREATE POLICY "authenticated_update" ON products FOR UPDATE USING (auth.role() = 'authenticated');
CREATE POLICY "authenticated_delete" ON products FOR DELETE USING (auth.role() = 'authenticated');

-- 4. Storage bucket created via Dashboard UI (Storage → New Bucket)
```

---

## 8. ENVIRONMENT VARIABLES (NOT USED - HARDCODED)

⚠️ **Important:** Keys are intentionally hardcoded in the client as per requirements.

If you want to switch to environment variables in the future:

```bash
# .env.local (create this file in root)
VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_PUBLIC_ANON_KEY
```

Then update `src/integrations/supabase/client.ts`:
```typescript
const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;
```

---

## 9. TESTING API CALLS

### Test Product Fetch in Console:
```javascript
// Paste in Browser Console (F12)
import { fetchProducts } from './src/data/products.js';
const products = await fetchProducts();
console.log('Products:', products);
```

### Test Storage Upload in Console:
```javascript
// Paste in Browser Console
import { supabase } from './src/integrations/supabase/client.js';

const file = new File(['test'], 'test.txt', { type: 'text/plain' });
const { data, error } = await supabase.storage
  .from('product-images')
  .upload('test/' + Date.now() + '.txt', file);

console.log(data, error);
```

---

## 10. SUPPORT & NEXT STEPS

### If still getting errors:
1. Check browser console (F12 → Console)
2. Copy full error message
3. Verify credentials one more time
4. Clear browser cache (Ctrl+Shift+Delete)
5. Restart dev server (npm run dev)

### For production deployment:
1. Create new Supabase project for production
2. Update credentials in `src/integrations/supabase/client.ts`
3. Add production Netlify domain to Supabase → Authentication → URL Configuration
4. Deploy to Netlify
5. Test all functionality on deployed URL

---

## Reference: Code Changes Made

- **`src/data/products.ts`** - Fixed fetchProducts to return empty array on error instead of throwing
- **`src/pages/AdminDashboard.tsx`** - Added comprehensive error handling for image upload and product insert
- **`src/pages/Category.tsx`** - Added error state and proper loading/error rendering
- **`src/pages/ProductDetail.tsx`** - Added error handling for product fetch
- **`src/components/content/ProductCarousel.tsx`** - Added error state and improved real-time subscription

All components now have:
- Try/catch blocks
- Console error logging
- User-facing error messages via toast
- Loading states
- Real-time subscriptions for auto-refresh
