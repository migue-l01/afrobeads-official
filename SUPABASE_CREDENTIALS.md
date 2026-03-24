# SUPABASE HARDCODED CREDENTIALS - UPDATE REQUIRED

⚠️ **IMPORTANT:** You must replace the placeholder values below with your actual Supabase project credentials.

## File to Update:
`src/integrations/supabase/client.ts`

## Current State (Placeholders):
```typescript
const supabaseUrl = "https://YOUR_PROJECT_ID.supabase.co";
const supabaseAnonKey = "YOUR_PUBLIC_ANON_KEY";
```

## How to Get Your Real Credentials:

### Step 1: Go to Supabase Dashboard
- URL: https://app.supabase.com
- Sign in with your account

### Step 2: Create or Select Your Project
- Click "New Project" or select existing project
- If creating new:
  - Name: `afrobeads` (or your choice)
  - Database Password: Create a strong password
  - Region: Select closest to your users
  - Click "Create new project" (takes ~2 minutes)

### Step 3: Find Your Credentials
- Click on your project
- Go to **Settings** (⚙️ icon) → **API**
- You'll see:
  - **Project URL** - Copy this (looks like: `https://xxxxx.supabase.co`)
  - **Project API keys** section:
    - Look for **anon** → **public** key - Copy this

### Step 4: Update the File

Open `src/integrations/supabase/client.ts` and replace:

```typescript
// BEFORE (Placeholders):
const supabaseUrl = "https://YOUR_PROJECT_ID.supabase.co";
const supabaseAnonKey = "YOUR_PUBLIC_ANON_KEY";

// AFTER (With your actual values):
const supabaseUrl = "https://abcdefghijk.supabase.co";  // Your project URL
const supabaseAnonKey = "eyJhbGciOiJIUzI1NiIsInR5c...";  // Your anon key
```

## Example (FAKE VALUES - DO NOT USE):
```typescript
const supabaseUrl = "https://myafroobeads.supabase.co";
const supabaseAnonKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
```

## Verification Checklist:
- [ ] supabaseUrl starts with `https://`
- [ ] supabaseUrl ends with `.supabase.co`
- [ ] supabaseAnonKey is a long string (usually 200+ characters)
- [ ] Not using placeholder text like `YOUR_PROJECT_ID`
- [ ] File saved after editing
- [ ] Dev server restarted (`npm run dev`)

## Testing After Update:
1. Open browser console (F12)
2. Go to http://localhost:8080/
3. Look for these logs:
   ```
   ✅ "ProductCarousel: Loading products..."
   ✅ "Fetched products: [...]"
   ```
4. If no products shown, go to admin panel and add one

## If Still Getting Errors:
1. **"Failed to fetch products"** → Check credentials are correct
2. **"bucket not found"** → Create `product-images` storage bucket in Supabase
3. **"row level security violation"** → Run the SQL setup in SUPABASE_SETUP.md

## Security Note:
⚠️ **These keys are hardcoded and will be visible in the browser.**

For production:
- Use environment variables
- Rotate keys regularly
- Monitor Supabase for unauthorized access
- Use Row Level Security (RLS) policies to restrict access
