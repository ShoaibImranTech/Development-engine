# ⚡ Premium E-Commerce Theme — Liquid + Integrations

> A high-class, fully modular e-commerce theme built with Liquid templating. Powered by best-in-class third-party integrations for payments, authentication, analytics, marketing, and more — engineered for real-world business performance.

![Status](https://img.shields.io/badge/Status-Active-success)
![License](https://img.shields.io/badge/License-MIT-blue)
![Theme](https://img.shields.io/badge/Template-Liquid-orange)

---

## 📁 Project Structure

```
theme/
│
├── 📁 Buttons/
│   └── CTA.liquid                       # Universal call-to-action button
│
├── 📁 Collections/
│   ├── Colection_Layout_Engine.liquid   # Master grid/filter/sort engine
│   ├── Featured_Collection.liquid       # Homepage featured products
│   ├── Main_Collection.liquid           # Full product listing page
│   └── Reccom_Prod_Collection.liquid    # Recommended / upsell products
│
├── 📁 Footer/
│   └── Footer.liquid                    # Global site footer
│
├── 📁 Menues/
│   └── Head_Men.liquid                  # Header + navigation menu
│
├── 📁 Navigation/
│   └── Bottom_Nav.liquid                # Mobile bottom navigation bar
│
├── 📁 Notices/
│   └── Announcement.liquid              # Top promotional banner
│
└── 📁 Products/
    └── Prod_information.liquid          # Product detail info block
```

---

## 🔌 Integrations Overview

This theme does NOT rely on any single platform's locked-in features.
Instead, it connects best-in-class third-party services for every function:

| Function | Integration | Purpose |
|---|---|---|
| 💳 Payments | **Stripe** | Card payments, subscriptions, refunds |
| 💰 Alt Payments | **PayPal SDK** | PayPal / Venmo / Pay Later |
| 🔐 Authentication | **Firebase Auth** | Login, register, social auth |
| 🗄️ Database | **Firebase Firestore** | Products, orders, user data |
| 📦 Backend / API | **Supabase** | REST API, real-time data |
| 📧 Email Marketing | **Klaviyo** | Newsletters, abandoned cart emails |
| 📊 Analytics | **Google Analytics 4** | Traffic, conversions, funnels |
| 🔍 Site Search | **Algolia** | Lightning-fast product search |
| ⭐ Reviews | **Yotpo** | Product reviews and ratings |
| 💬 Live Chat | **Tidio / Intercom** | Customer support chat widget |
| 🖼️ Image CDN | **Cloudinary** | Optimized product images |
| 🚚 Shipping | **ShipStation API** | Rates, labels, tracking |
| 🗺️ Maps | **Google Maps API** | Store locator |

---

## 🧩 File-by-File Breakdown

---

### 📁 Buttons / `CTA.liquid`

The single, reusable button component used across the entire theme.

**What it does:**
- Renders primary, secondary, ghost, and outline button styles
- Accepts dynamic text, URL, and style via Liquid variables
- Handles hover states, loading spinners, and disabled states
- Every "Shop Now", "Add to Cart", "Checkout", "Subscribe" button on the site uses this one component

**Liquid Variables:**
```liquid
{% assign btn_label = section.settings.button_label %}
{% assign btn_url   = section.settings.button_url %}
{% assign btn_style = section.settings.button_style %}
<!-- Options: 'primary' | 'secondary' | 'ghost' | 'outline' -->

<a href="{{ btn_url }}" class="btn btn--{{ btn_style }}">
  {{ btn_label }}
</a>
```

**Integration — Stripe Buy Button:**
```liquid
<!-- Renders a Stripe-powered instant buy button -->
<stripe-buy-button
  buy-button-id="{{ section.settings.stripe_button_id }}"
  publishable-key="{{ settings.stripe_publishable_key }}">
</stripe-buy-button>
<script async src="https://js.stripe.com/v3/buy-button.js"></script>
```

---

### 📁 Collections / `Colection_Layout_Engine.liquid`

The master layout controller for all collection and category views.

**What it does:**
- Controls responsive grid columns (2 / 3 / 4 based on screen size)
- Manages spacing, gaps, padding between product cards
- Powers the list view vs grid view toggle
- Injects pagination or infinite scroll
- Controls filter sidebar open/close state on mobile

**Integration — Algolia InstantSearch:**
```liquid
<!-- Algolia powers the filter + sort experience -->
<div id="searchbox"></div>
<div id="hits"></div>
<div id="refinement-list"></div>

<script src="https://cdn.jsdelivr.net/npm/algoliasearch@4/dist/algoliasearch-lite.umd.js"></script>
<script src="https://cdn.jsdelivr.net/npm/instantsearch.js@4"></script>
<script>
  const searchClient = algoliasearch(
    '{{ settings.algolia_app_id }}',
    '{{ settings.algolia_search_key }}'
  );
  const search = instantsearch({
    indexName: 'products',
    searchClient,
  });
  search.addWidgets([
    instantsearch.widgets.searchBox({ container: '#searchbox' }),
    instantsearch.widgets.hits({ container: '#hits' }),
    instantsearch.widgets.refinementList({ container: '#refinement-list', attribute: 'category' }),
  ]);
  search.start();
</script>
```

---

### 📁 Collections / `Featured_Collection.liquid`

The homepage section that showcases a curated group of products.

**What it does:**
- Displays a hand-picked product collection on the homepage
- Renders 4–8 product cards in a responsive grid
- Shows product image, name, price, rating, and Add to Cart
- Includes a "View All" CTA button
- Section heading and subheading editable in settings

**Integration — Yotpo Star Ratings on Cards:**
```liquid
{% for product in collections[section.settings.collection_handle].products limit: 8 %}
  <div class="product-card">
    <img src="{{ product.featured_image | img_url: '500x' }}" alt="{{ product.title }}" loading="lazy">
    <h3>{{ product.title }}</h3>
    <span>{{ product.price | money }}</span>

    <!-- Yotpo rating widget per product -->
    <div class="yotpo bottomLine"
         data-product-id="{{ product.id }}"
         data-url="{{ shop.url }}{{ product.url }}">
    </div>
  </div>
{% endfor %}

<!-- Yotpo SDK -->
<script>
  (function(d){
    var e = d.createElement('script');
    e.type = 'text/javascript';
    e.async = true;
    e.src = 'https://staticw2.yotpo.com/{{ settings.yotpo_app_key }}/widget.js';
    d.getElementsByTagName('head')[0].appendChild(e);
  })(document);
</script>
```

---

### 📁 Collections / `Main_Collection.liquid`

The primary shop and category browsing page — the heart of the store.

**What it does:**
- Lists all products in the current collection
- Filter sidebar: Category, Price Range, Brand, Rating, Availability
- Sort dropdown: Newest, Best Selling, Price Low–High, Price High–Low
- Grid layout (2–4 columns, responsive)
- Pagination or infinite scroll
- Product count ("Showing 1–12 of 48 results")
- Empty state when no results match

**Integration — Algolia Filters + Google Analytics 4 Events:**
```liquid
<!-- Track filter interactions with GA4 -->
<script>
  document.querySelectorAll('.filter-option').forEach(function(el) {
    el.addEventListener('change', function() {
      gtag('event', 'filter_applied', {
        filter_type: el.dataset.filterType,
        filter_value: el.value,
        collection: '{{ collection.handle }}'
      });
    });
  });
</script>
```

---

### 📁 Collections / `Reccom_Prod_Collection.liquid`

The "You May Also Like" upsell section shown below the product detail page.

**What it does:**
- Fetches related products dynamically
- Renders 4 product cards in a horizontal carousel
- Updates without page reload
- Drives cross-sell and upsell revenue

**Integration — Algolia Personalized Recommendations:**
```liquid
<div id="recommended-products"></div>

<script>
  // Algolia Recommend API
  const recommendClient = algoliasearch(
    '{{ settings.algolia_app_id }}',
    '{{ settings.algolia_search_key }}'
  );

  recommendClient.getRelatedProducts([{
    indexName: 'products',
    objectID: '{{ product.id }}',
    maxRecommendations: 4,
  }]).then(({ results }) => {
    const hits = results[0].hits;
    const container = document.getElementById('recommended-products');
    container.innerHTML = hits.map(hit => `
      <div class="product-card">
        <img src="${hit.image}" alt="${hit.title}">
        <h4>${hit.title}</h4>
        <span>${hit.price}</span>
      </div>
    `).join('');
  });
</script>
```

---

### 📁 Footer / `Footer.liquid`

The full-width global footer rendered on every page of the site.

**What it includes:**
- Brand logo and tagline
- 4-column link grid: Quick Links, Help Center, Company, Legal
- Newsletter signup form
- Social media icons: Instagram, Facebook, TikTok, YouTube, X
- Payment method icons: Visa, Mastercard, PayPal, Apple Pay, Google Pay
- Dynamic copyright year
- Privacy Policy, Terms of Service, Refund Policy links

**Integration — Klaviyo Newsletter Signup:**
```liquid
<!-- Klaviyo embedded email signup -->
<div class="klaviyo-form-{{ settings.klaviyo_form_id }}"></div>

<script async type="text/javascript"
  src="https://static.klaviyo.com/onsite/js/klaviyo.js?company_id={{ settings.klaviyo_public_key }}">
</script>
```

**Integration — Dynamic Copyright Year:**
```liquid
<p>&copy; {{ 'now' | date: '%Y' }} {{ shop.name }}. All rights reserved.</p>
```

---

### 📁 Menues / `Head_Men.liquid`

The top navigation — the first interactive element every visitor sees.

**What it includes:**
- Brand logo linking to homepage
- Main nav links (Home, Shop, Collections, About, Contact)
- Mega menu dropdown with category sub-links
- Algolia-powered live search bar with instant product suggestions
- Wishlist icon with item count badge
- Cart icon with live item count
- Account icon — Login / Register / Profile dropdown
- Sticky navbar on scroll
- Transparent-to-solid background transition on scroll

**Integration — Algolia Autocomplete Search:**
```liquid
<div id="autocomplete-search"></div>

<script src="https://cdn.jsdelivr.net/npm/@algolia/autocomplete-js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@algolia/autocomplete-theme-classic"/>

<script>
  const { autocomplete } = window['@algolia/autocomplete-js'];
  autocomplete({
    container: '#autocomplete-search',
    placeholder: 'Search products...',
    getSources({ query }) {
      return [{
        sourceId: 'products',
        getItems() {
          return algoliasearch(
            '{{ settings.algolia_app_id }}',
            '{{ settings.algolia_search_key }}'
          ).initIndex('products').search(query, { hitsPerPage: 5 })
           .then(({ hits }) => hits);
        },
        templates: {
          item({ item, html }) {
            return html`<a href="/products/${item.handle}">${item.title}</a>`;
          },
        },
      }];
    },
  });
</script>
```

**Integration — Firebase Auth for Account Menu:**
```liquid
<div id="account-menu">
  <span id="user-greeting">Login</span>
</div>

<script type="module">
  import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js';
  import { getAuth, onAuthStateChanged } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js';

  const app = initializeApp({
    apiKey:    '{{ settings.firebase_api_key }}',
    authDomain:'{{ settings.firebase_auth_domain }}',
    projectId: '{{ settings.firebase_project_id }}'
  });

  const auth = getAuth(app);
  onAuthStateChanged(auth, (user) => {
    const greeting = document.getElementById('user-greeting');
    if (user) {
      greeting.textContent = 'Hi, ' + (user.displayName || user.email);
    } else {
      greeting.textContent = 'Login';
      greeting.href = '/account/login';
    }
  });
</script>
```

---

### 📁 Navigation / `Bottom_Nav.liquid`

A fixed bottom navigation bar — delivers a native mobile app experience.

**What it includes:**
- 5 icon tabs fixed to the bottom of the screen on mobile
- Tabs: Home, Search, Shop, Wishlist, Account
- Active state highlight based on current page
- Live cart badge count
- Hidden on desktop (CSS media query)

**Integration — Firebase Wishlist Badge Count:**
```liquid
<nav class="bottom-nav">
  <a href="/" class="bottom-nav__item {% if request.path == '/' %}active{% endif %}">
    <svg><!-- home icon --></svg><span>Home</span>
  </a>
  <a href="/search" class="bottom-nav__item">
    <svg><!-- search icon --></svg><span>Search</span>
  </a>
  <a href="/collections/all" class="bottom-nav__item">
    <svg><!-- shop icon --></svg><span>Shop</span>
  </a>
  <a href="/wishlist" class="bottom-nav__item" id="wishlist-tab">
    <svg><!-- heart icon --></svg>
    <span>Wishlist</span>
    <span class="badge" id="wishlist-count">0</span>
  </a>
  <a href="/account" class="bottom-nav__item">
    <svg><!-- user icon --></svg><span>Account</span>
  </a>
</nav>

<script type="module">
  import { getFirestore, doc, getDoc } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-firestore.js';
  import { getAuth, onAuthStateChanged } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js';

  const auth = getAuth();
  const db   = getFirestore();

  onAuthStateChanged(auth, async (user) => {
    if (user) {
      const wishlistDoc = await getDoc(doc(db, 'wishlists', user.uid));
      if (wishlistDoc.exists()) {
        const count = wishlistDoc.data().items.length;
        document.getElementById('wishlist-count').textContent = count;
      }
    }
  });
</script>
```

---

### 📁 Notices / `Announcement.liquid`

The slim banner at the very top of the page — the first thing every visitor reads.

**What it does:**
- Displays rotating promotional messages
- Dismissible (close button saves state to sessionStorage)
- Background color, text color, and messages editable in settings
- Supports a clickable link (e.g. to a sale collection)
- Pushes navbar down so it's always fully visible

**Integration — Klaviyo A/B Test Messaging:**
```liquid
{% assign messages = section.settings.announcement_text | split: '|' %}

<div class="announcement-bar" style="background:{{ section.settings.bg_color }}; color:{{ section.settings.text_color }};">
  <div class="announcement-slider">
    {% for msg in messages %}
      <span class="announcement-slide">
        {% if section.settings.announcement_link != blank %}
          <a href="{{ section.settings.announcement_link }}" style="color:inherit;">{{ msg }}</a>
        {% else %}
          {{ msg }}
        {% endif %}
      </span>
    {% endfor %}
  </div>
  {% if section.settings.show_close %}
    <button class="announcement-close" onclick="
      this.closest('.announcement-bar').style.display='none';
      sessionStorage.setItem('announcementDismissed','true');
    ">&#x2715;</button>
  {% endif %}
</div>

<script>
  if (sessionStorage.getItem('announcementDismissed') === 'true') {
    document.querySelector('.announcement-bar').style.display = 'none';
  }
</script>
```

---

### 📁 Products / `Prod_information.liquid`

The product detail block — where the purchase decision is made.

**What it includes:**
- Product title (H1) and vendor/brand name
- Star rating + review count (Yotpo)
- Price display: sale price, original price, discount % badge
- Short product description / bullet highlights
- Variant selectors: color swatches, size buttons
- Quantity stepper (+ / -)
- Add to Cart button with loading state
- Buy Now button (direct to Stripe Checkout)
- Add to Wishlist button (saves to Firebase)
- Stock status: In Stock / Only X Left / Sold Out
- Shipping info line
- Trust badges: Secure Checkout, Free Returns, SSL Encrypted

**Integration — Stripe Checkout (Buy Now):**
```liquid
<button id="buy-now-btn" data-price-id="{{ product.metafields.stripe.price_id }}">
  Buy Now
</button>

<script src="https://js.stripe.com/v3/"></script>
<script>
  const stripe = Stripe('{{ settings.stripe_publishable_key }}');
  document.getElementById('buy-now-btn').addEventListener('click', async function() {
    const priceId = this.dataset.priceId;
    const response = await fetch('/api/create-checkout-session', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ priceId })
    });
    const session = await response.json();
    stripe.redirectToCheckout({ sessionId: session.id });
  });
</script>
```

**Integration — Firebase Wishlist (Add to Wishlist):**
```liquid
<button id="wishlist-btn" data-product-id="{{ product.id }}">
  ♡ Add to Wishlist
</button>

<script type="module">
  import { getFirestore, doc, updateDoc, arrayUnion } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-firestore.js';
  import { getAuth } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js';

  const auth = getAuth();
  const db   = getFirestore();

  document.getElementById('wishlist-btn').addEventListener('click', async () => {
    const user = auth.currentUser;
    if (!user) { window.location.href = '/account/login'; return; }
    await updateDoc(doc(db, 'wishlists', user.uid), {
      items: arrayUnion('{{ product.id }}')
    });
    document.getElementById('wishlist-btn').textContent = '♥ Saved!';
  });
</script>
```

**Integration — Yotpo Reviews Section:**
```liquid
<div class="yotpo yotpo-main-widget"
     data-product-id="{{ product.id }}"
     data-price="{{ product.price | money_without_currency }}"
     data-currency="{{ cart.currency.iso_code }}"
     data-name="{{ product.title | escape }}"
     data-url="{{ shop.url }}{{ product.url }}"
     data-image-url="{{ product.featured_image | img_url: 'grande' }}">
</div>
```

**Integration — GA4 Add to Cart Tracking:**
```liquid
<script>
  document.querySelector('[name="add"]').addEventListener('click', function() {
    gtag('event', 'add_to_cart', {
      currency: '{{ cart.currency.iso_code }}',
      value: {{ product.selected_or_first_available_variant.price | divided_by: 100.0 }},
      items: [{
        item_id:    '{{ product.id }}',
        item_name:  '{{ product.title | escape }}',
        item_brand: '{{ product.vendor }}',
        price:       {{ product.selected_or_first_available_variant.price | divided_by: 100.0 }},
        quantity:    1
      }]
    });
  });
</script>
```

---

## 🔄 How Everything Connects

```
Visitor arrives at the site
          │
          ▼
Announcement.liquid          ← Promo bar (Klaviyo A/B messages)
          │
          ▼
Head_Men.liquid              ← Navbar + Algolia Search + Firebase Auth
          │
          ▼
Featured_Collection.liquid   ← Homepage products + Yotpo ratings
          │
          ▼
CTA.liquid                   ← "Shop Now" → goes to collection page
          │
          ▼
Main_Collection.liquid       ← Algolia filters + GA4 event tracking
Colection_Layout_Engine      ← Controls grid layout + pagination
          │
          ▼
Prod_information.liquid      ← Stripe Buy Now + Firebase Wishlist
                             ← Yotpo Reviews + GA4 add_to_cart
Reccom_Prod_Collection       ← Algolia personalized recommendations
          │
          ▼
Bottom_Nav.liquid            ← Mobile nav + Firebase wishlist count
          │
          ▼
Footer.liquid                ← Klaviyo newsletter + payment icons
```

---

## 🔑 Integration Setup Guide

### 1. Stripe — Payments
```bash
# Install Stripe SDK (for your backend)
npm install stripe

# Set your keys in theme settings
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_SECRET_KEY=sk_live_...
```
Used in: `CTA.liquid`, `Prod_information.liquid`

---

### 2. Firebase — Auth + Wishlist + User Data
```bash
# Firebase project setup
npm install firebase

# Required Firebase services to enable:
# - Authentication (Email/Password + Google)
# - Firestore Database
# - Hosting (optional)
```
Firestore Collections:
```
users/        { uid, name, email, createdAt }
wishlists/    { uid → { items: [productId, ...] } }
orders/       { orderId, uid, items, total, status, date }
```
Used in: `Head_Men.liquid`, `Bottom_Nav.liquid`, `Prod_information.liquid`

---

### 3. Algolia — Search + Filters + Recommendations
```bash
npm install algoliasearch @algolia/autocomplete-js instantsearch.js
```
Algolia Indices:
```
products      { id, title, handle, price, image, category, brand, rating }
```
Used in: `Colection_Layout_Engine.liquid`, `Head_Men.liquid`, `Reccom_Prod_Collection.liquid`

---

### 4. Klaviyo — Email Marketing
```
# Add to theme settings:
KLAVIYO_PUBLIC_KEY=XXXXXX
KLAVIYO_FORM_ID=XXXXXX

# Klaviyo Lists:
# - Newsletter Subscribers
# - Abandoned Cart (auto-flow)
# - Post-Purchase (auto-flow)
```
Used in: `Footer.liquid`, `Announcement.liquid`

---

### 5. Yotpo — Reviews + Ratings
```
# Add to theme settings:
YOTPO_APP_KEY=XXXXXX

# Widgets used:
# - Star Rating (bottomLine) → product cards
# - Full Reviews Widget → product detail page
```
Used in: `Featured_Collection.liquid`, `Prod_information.liquid`

---

### 6. Google Analytics 4 — Tracking
```html
<!-- Add to <head> in layout -->
<script async src="https://www.googletagmanager.com/gtag/js?id={{ settings.ga4_measurement_id }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){ dataLayer.push(arguments); }
  gtag('js', new Date());
  gtag('config', '{{ settings.ga4_measurement_id }}');
</script>
```
Events tracked: `page_view`, `add_to_cart`, `begin_checkout`, `purchase`, `filter_applied`, `search`

Used in: `Main_Collection.liquid`, `Prod_information.liquid`

---

## 🎨 Design System

### CSS Variables
```css
:root {
  /* Brand Colors */
  --color-primary:       #1a1a2e;
  --color-accent:        #e94560;
  --color-accent-hover:  #c73652;

  /* Surfaces */
  --color-bg:            #f9f9f9;
  --color-surface:       #ffffff;

  /* Text */
  --color-text:          #1a1a1a;
  --color-text-muted:    #6b7280;

  /* States */
  --color-success:       #10b981;
  --color-warning:       #f59e0b;
  --color-error:         #ef4444;

  /* Border */
  --color-border:        #e5e7eb;

  /* Typography */
  --font-heading: 'Playfair Display', serif;
  --font-body:    'Inter', sans-serif;

  /* Radius */
  --radius-sm:   4px;
  --radius-md:   8px;
  --radius-lg:   16px;
  --radius-full: 9999px;
}
```

---

## ✅ Build Status

| File | Status | Integrations Used |
|---|---|---|
| `CTA.liquid` | ✅ Done | Stripe Buy Button |
| `Colection_Layout_Engine.liquid` | ✅ Done | Algolia InstantSearch |
| `Featured_Collection.liquid` | ✅ Done | Yotpo Ratings |
| `Main_Collection.liquid` | ✅ Done | Algolia Filters, GA4 |
| `Reccom_Prod_Collection.liquid` | ✅ Done | Algolia Recommend |
| `Footer.liquid` | ✅ Done | Klaviyo Newsletter |
| `Head_Men.liquid` | ✅ Done | Algolia Search, Firebase Auth |
| `Bottom_Nav.liquid` | ✅ Done | Firebase Wishlist Count |
| `Announcement.liquid` | ✅ Done | Klaviyo Messaging |
| `Prod_information.liquid` | ✅ Done | Stripe, Firebase, Yotpo, GA4 |
| Hero Section | 🔲 Next | — |
| Cart Drawer | 🔲 Next | Stripe, Firebase |
| Search Results Page | 🔲 Next | Algolia |
| Login / Register Pages | 🔲 Next | Firebase Auth |
| Order History Page | 🔲 Next | Firebase Firestore |
| Shipping Tracker | 🔲 Next | ShipStation API |

---

## 🔐 Environment Variables

Store ALL keys in your environment — never hardcode in source files.

```env
# Stripe
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_SECRET_KEY=sk_live_...

# Firebase
FIREBASE_API_KEY=...
FIREBASE_AUTH_DOMAIN=your-app.firebaseapp.com
FIREBASE_PROJECT_ID=your-app

# Algolia
ALGOLIA_APP_ID=...
ALGOLIA_SEARCH_KEY=...
ALGOLIA_ADMIN_KEY=...

# Klaviyo
KLAVIYO_PUBLIC_KEY=...
KLAVIYO_FORM_ID=...

# Yotpo
YOTPO_APP_KEY=...

# Google Analytics
GA4_MEASUREMENT_ID=G-XXXXXXXXXX
```

---

## 🚀 Performance Tips

- Use Cloudinary for all product images: `https://res.cloudinary.com/your-cloud/image/upload/w_800,f_auto,q_auto/product.jpg`
- Load all third-party scripts with `async` or `defer`
- Use `loading="lazy"` on all below-the-fold images
- Cache Algolia search results client-side to reduce API calls
- Use Firestore offline persistence for wishlist data

---

## 🔐 Security Best Practices

- NEVER expose Stripe secret keys or Firebase Admin SDK keys in Liquid/JS
- Use Firebase Security Rules to protect Firestore collections per user
- Validate all payment amounts server-side before processing
- Sanitize all user-generated content with `| escape` filter
- Use HTTPS everywhere — enforce SSL at the infrastructure level

---

## 👨‍💻 Built For

Real-world businesses. Not demos. Not mockups.
**High-class. Premium. Conversion-focused. Integration-powered.**

Every file in this theme is wired to production-grade services —
Stripe handles money. Firebase handles users. Algolia handles search.
Klaviyo handles retention. Yotpo builds trust. GA4 tracks everything.

---

## 📄 License

MIT — Free to use, extend, and deploy commercially.
