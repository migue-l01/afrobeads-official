# Testing Checklist - Supabase Integration Verification

## ✅ PRE-TESTING SETUP

Before running tests, complete these steps:

### 1. Update Credentials
- [ ] Open `src/integrations/supabase/client.ts`
- [ ] Replace `YOUR_PROJECT_ID` with your actual Supabase project URL
- [ ] Replace `YOUR_PUBLIC_ANON_KEY` with your actual anon key
- [ ] Save the file
- [ ] Restart dev server: `npm run dev`

### 2. Create Database Table
- [ ] Go to https://app.supabase.com → Your Project → SQL Editor
- [ ] Copy all SQL from `SUPABASE_SETUP.md` section 2
- [ ] Paste in SQL Editor and run
- [ ] Verify table appears in Database → Tables

### 3. Create Storage Bucket
- [ ] Go to Supabase → Storage
- [ ] Click "New Bucket"
- [ ] Name: `product-images` (exact spelling)
- [ ] Make Public: YES
- [ ] Create bucket

### 4. Add CORS Configuration (if needed)
- [ ] Go to Supabase → Authentication → URL Configuration
- [ ] Add "Site URL": `http://localhost:8080` (for dev)
- [ ] Add "Redirect URLs": `http://localhost:8080` (for dev)
- [ ] Save

---

## 🧪 TESTING SCENARIOS

### Test 1: Customer Page - Load Products
**Objective:** Verify products fetch correctly on homepage

**Steps:**
1. Open http://localhost:8080 in browser
2. Open Developer Tools (F12)
3. Go to Console tab
4. Look for these logs (in order):
   - [ ] "ProductCarousel: Loading products..."
   - [ ] "Fetched products: Array(...)" - shows array
   - [ ] "ProductCarousel: Real-time subscription active"

**Expected Result:**
- [ ] No errors in console
- [ ] "Fetched products:" log shows product array
- [ ] Products carousel may be empty (if no products added yet)

**If Failed:**
- [ ] Check browser console for error messages
- [ ] Verify credentials in client.ts
- [ ] Check Supabase dashboard - is project running?
- [ ] Check if products table exists

---

### Test 2: Admin Panel - Add Product
**Objective:** Verify product creation and image upload works

**Steps:**
1. Go to http://localhost:8080/admin
2. Click "Add Product" button
3. Fill form:
   - Name: "Test Necklace"
   - Category: "Necklaces"
   - Price: "1500"
   - Image: Upload a test image (JPG/PNG)
   - Description: "Test product" (optional)
4. Keep switches on (Mark as New, Active)
5. Click "Add Product" button
6. Watch console for these logs:
   - [ ] "Uploading image to Supabase Storage: ..."
   - [ ] File size and type shown
   - [ ] "Image uploaded successfully: https://..."
   - [ ] "Inserting product into Supabase: {...}"
   - [ ] Success toast: "Product added!"

**Expected Result:**
- [ ] No errors in console
- [ ] Success message appears
- [ ] Product appears in admin list below

**If Failed:**
- [ ] **Error: "Storage bucket not found"** → Create `product-images` bucket in Supabase
- [ ] **Error: "Upload failed"** → Check file size < 10MB and is image
- [ ] **Error: "Insert failed"** → Check RLS policies are set correctly

---

### Test 3: Product Appears on Customer Page
**Objective:** Verify product syncs to customer page

**Steps:**
1. Go back to http://localhost:8080
2. Scroll down to "Products" carousel section
3. Within 2-3 seconds, refresh the page (F5)
4. Check console for:
   - [ ] "ProductCarousel: Product change detected"
   - [ ] Product now appears in carousel

**Expected Result:**
- [ ] Your newly added product appears
- [ ] Product shows: name, image, category, price
- [ ] Image loads correctly from Supabase Storage

**If Failed:**
- [ ] Check if product was actually added (check admin list)
- [ ] Verify image_url is not null in Supabase products table
- [ ] Check image URL is accessible in browser (open URL directly)

---

### Test 4: Cross-Device Real-time Sync
**Objective:** Verify real-time updates work

**Setup:**
1. Open http://localhost:8080 in Browser 1 (customer page)
2. Open http://localhost:8080/admin in Browser 2 (admin panel)
3. Arrange windows side-by-side

**Steps:**
1. In Browser 2 (admin), add another product
2. Watch Browser 1 (customer) carousel
3. Within 1-2 seconds, the new product should appear WITHOUT page refresh
4. Check console in Browser 1 for:
   - [ ] "Product change detected"
   - [ ] "Fetched products: Array(...)" with updated count

**Expected Result:**
- [ ] New product appears in carousel within 1-2 seconds
- [ ] No manual refresh needed (real-time sync works)
- [ ] Both windows stay in sync

**If Failed:**
- [ ] Check if Realtime is enabled in Supabase → Extensions
- [ ] Check browser console for subscription errors
- [ ] May need to manually refresh page to verify data is in DB

---

### Test 5: Edit Product
**Objective:** Verify product editing works

**Steps:**
1. Go to http://localhost:8080/admin
2. Find the product you added
3. Click "Edit" button
4. Change name to "Updated Test Necklace"
5. Click "Update" button
6. Check console for:
   - [ ] "Updating product in Supabase: ..."
   - [ ] Success toast: "Product updated!"

**Expected Result:**
- [ ] Product name changes in list
- [ ] Changes appear on customer page within 1-2 seconds

---

### Test 6: Delete Product
**Objective:** Verify product deletion works

**Steps:**
1. Go to http://localhost:8080/admin
2. Find a product to delete
3. Click trash icon
4. Check console for:
   - [ ] "Deleting product from Supabase: ..."
   - [ ] Success toast: "Product deleted"

**Expected Result:**
- [ ] Product disappears from admin list
- [ ] Product disappears from customer page within 1-2 seconds

---

### Test 7: Product Detail Page
**Objective:** Verify individual product page loads correctly

**Steps:**
1. Go to http://localhost:8080
2. Click on a product in carousel
3. Check logs for:
   - [ ] "ProductDetail: Loading product..."
   - [ ] "Fetched products: Array(...)"
4. Page loads with product details

**Expected Result:**
- [ ] Product name, image, price, description shown
- [ ] "Add to Cart" button works
- [ ] "Add to Favorites" button works
- [ ] No errors in console

---

### Test 8: Error Handling - Invalid Credentials
**Objective:** Verify error handling works

**Steps To Simulate Error:**
1. Edit `src/integrations/supabase/client.ts`
2. Change one character in the anon key (break it)
3. Restart dev server
4. Go to http://localhost:8080
5. Check console for error message

**Expected Result:**
- [ ] Page shows loading state then error message
- [ ] Console shows detailed error
- [ ] Error is user-friendly (not cryptic)

**Then Fix:**
- [ ] Restore correct anon key
- [ ] Restart dev server
- [ ] Page works again

---

### Test 9: Error Handling - File Upload Validation
**Objective:** Verify file validation works

**Steps:**
1. Go to http://localhost:8080/admin
2. Fill product form
3. Try to upload a non-image file:
   - [ ] Select a PDF or TXT file
   - Expected: "Please select a valid image file"
4. Try to upload a huge image (if you have one):
   - [ ] File > 10MB
   - Expected: "Image size must be less than 10MB"
5. Try to upload without selecting image:
   - [ ] Click Add Product without image
   - Expected: "Image is required"

**Expected Result:**
- [ ] Validation errors show immediately
- [ ] Upload doesn't proceed with invalid files

---

### Test 10: Category Page Filtering
**Objective:** Verify category page works with Supabase data

**Steps:**
1. Go to http://localhost:8080/category/necklaces
2. Wait for products to load
3. Check console for:
   - [ ] "Category: Loading products..." log
   - [ ] Products array shown

**Expected Result:**
- [ ] Page loads with filtered products
- [ ] No errors in console
- [ ] Can apply filters and sort

---

## 📊 FINAL VERIFICATION CHECKLIST

All tests should pass before considering integration complete:

- [ ] Test 1: Customer page loads products
- [ ] Test 2: Admin can add product
- [ ] Test 3: Product appears on customer page
- [ ] Test 4: Real-time sync works (cross-device)
- [ ] Test 5: Edit product works
- [ ] Test 6: Delete product works
- [ ] Test 7: Product detail page works
- [ ] Test 8: Error handling works
- [ ] Test 9: File validation works
- [ ] Test 10: Category filtering works

---

## 🔍 CONSOLE LOG REFERENCE

### Expected Success Logs:
```
✅ "ProductCarousel: Loading products..."
✅ "Fetched products: Array(5)" - shows product data
✅ "ProductCarousel: Real-time subscription active"
✅ "Image uploaded successfully: https://..."
✅ "Inserting product into Supabase: {...}"
✅ "Product updated!"
✅ "Product deleted"
```

### Error Logs to Investigate:
```
❌ "Supabase fetch error:"           - Credentials wrong
❌ "Error loading products"          - Network or credentials
❌ "Storage bucket not found"        - Create product-images bucket
❌ "Upload failed"                   - File validation or storage issue
❌ "Insert failed"                   - RLS policy issue
❌ "supabase is not defined"         - Import issue
```

---

## 🐛 TROUBLESHOOTING

| Symptom | Check |
|---------|-------|
| Nothing loads | Credentials correct? Supabase project running? |
| Products don't appear | Any errors in console? Products in admin? |
| Images broken | Image URL exists in Supabase? Bucket is public? |
| Real-time not working | Refresh page - data there? Realtime enabled in Supabase? |
| Upload fails | File < 10MB? Is image? Bucket exists? |
| Delete/Edit fails | RLS policies set? Console shows error? |

---

## 📞 IF TESTS FAIL

1. **Always check browser console first (F12)**
2. **Copy exact error message**
3. **Check SUPABASE_SETUP.md for solution**
4. **Verify each credential step by step**
5. **Try clearing cache (Ctrl+Shift+Delete) and reloading**
6. **Restart dev server (Ctrl+C, then npm run dev)**

---

**Last Updated:** March 22, 2026  
**Status:** Ready for Testing
