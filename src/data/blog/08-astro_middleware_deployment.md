---
title: "Astro Middleware, Edge Authentication, and Deployment Caching"
description: "Astro Middleware দিয়ে রিকোয়েস্ট ইন্টারসেপ্ট করা, Edge Network-এ ইউজার অথেন্টিকেশন এবং Vercel বা Cloudflare-এ Caching অপ্টিমাইজেশন।"
pubDate: "2026-10-08"
author: "Pradipta Sarker"
tags: ["Astro", "Middleware", "Edge", "Authentication", "Deployment"]
---

আমাদের Astro ব্লগ সিরিজের এটি শেষ পোস্ট। আমরা বেসিক আর্কিটেকচার থেকে শুরু করে ডেটাবেস, AI এবং UX ইমপ্লিমেন্ট করেছি। এবার সময় প্রজেক্টটিকে প্রোডাকশনে (Production) নিয়ে যাওয়ার। 

এই ব্লগে আমরা শিখব কীভাবে **Astro Middleware** ব্যবহার করে ইউজার অথেন্টিকেশন (Authentication) কন্ট্রোল করতে হয় এবং সাইটটিকে **Vercel** বা **Cloudflare Edge**-এ ডিপ্লয় করে রেসপন্স টাইম একদম কমিয়ে আনতে হয়।

## ১. Astro Middleware কী?

**Middleware** হলো এমন একটি কোড যা সার্ভারের কাছে আসা যেকোনো রিকোয়েস্ট (Request) এবং ক্লায়েন্টকে দেওয়া রেসপন্স (Response)-এর মাঝে কাজ করে। এর মাধ্যমে আপনি যেকোনো পেজ বা API রেন্ডার হওয়ার আগে রিকোয়েস্টটিকে ইন্টারসেপ্ট করে চেক করতে পারেন।

Middleware তৈরি করার জন্য `src` ফোল্ডারের ঠিক রুটে (pages-এর ভেতর নয়) `middleware.ts` বা `middleware.js` নামে একটি ফাইল তৈরি করতে হয়।

### বেসিক Middleware স্ট্রাকচার:

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware((context, next) => {
  console.log("Incoming request to:", context.url.pathname);
  
  // পরবর্তী প্রসেসে রিকোয়েস্টটি পাঠিয়ে দিন
  return next(); 
});
```

## ২. Edge Authentication (Route Protection)

Middleware-এর সবচেয়ে বড় ইউস-কেস হলো **Authentication**। ধরুন, আপনার ওয়েবসাইটে একটি `/dashboard` পেজ আছে, যেখানে লগ-ইন ছাড়া কেউ ঢুকতে পারবে না। আমরা Middleware দিয়ে চেক করব ইউজারের কুকিতে (Cookie) ভ্যালিড টোকেন আছে কি না।

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async ({ cookies, redirect, url }, next) => {
  // যদি ইউজার ড্যাশবোর্ডে যাওয়ার চেষ্টা করে
  if (url.pathname.startsWith('/dashboard')) {
    const sessionToken = cookies.get('auth_token')?.value;

    // টোকেন না থাকলে লগ-ইন পেজে রিডাইরেক্ট করুন
    if (!sessionToken) {
      return redirect('/login');
    }
    
    // প্রয়োজনে টোকেন ভ্যালিডেশন লজিক এখানে অ্যাড করতে পারেন
  }

  // সব ঠিক থাকলে পেজ রেন্ডার হতে দিন
  return next();
});
```
এই কোডটি **Edge Network**-এ রান করবে, যার মানে হলো ইউজার কোনো পেজ রেন্ডার হওয়ার আগেই খুব দ্রুত আনঅথোরাইজড এক্সেস ব্লক হয়ে যাবে।

## ৩. Edge Rendering এবং Adapters

Astro প্রজেক্টকে ডাইনামিক বা SSR মোডে রান করাতে হলে হোস্টিং প্রোভাইডারের সার্ভারে তা এক্সিকিউট করতে হয়। এর জন্য প্রয়োজন **Adapter**।

আপনি যদি প্রজেক্টটি Vercel-এ ডিপ্লয় করতে চান, তবে টার্মিনালে Vercel Adapter ইনস্টল করতে হবে:

```bash
npx astro add vercel
```

এটি স্বয়ংক্রিয়ভাবে `astro.config.mjs` ফাইল আপডেট করে দেবে:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel/serverless'; // অথবা edge

export default defineConfig({
  output: 'server', // বা 'hybrid'
  adapter: vercel(),
});
```

Cloudflare-এর জন্য `npx astro add cloudflare` ব্যবহার করতে হবে। Edge-এ ডিপ্লয় করলে আপনার কোড ইউজারের সবচেয়ে কাছের সার্ভার (CDN) থেকে রান করবে, ফলে পারফরম্যান্স হবে অসাধারণ।

## ৪. Deployment Caching Strategies

SSR সাইটগুলো প্রতিটি রিকোয়েস্টে নতুন করে HTML জেনারেট করে, যা সার্ভারের উপর চাপ ফেলে এবং স্পিড কমিয়ে দেয়। তাই **Cache-Control Headers** ব্যবহার করা খুবই গুরুত্বপূর্ণ। 

Middleware বা সরাসরি API/Page থেকে আপনি Caching Header সেট করতে পারেন:

```typescript
// src/pages/api/data.ts
export async function GET() {
  const data = await fetchSomeData();
  
  return new Response(JSON.stringify(data), {
    headers: {
      'Content-Type': 'application/json',
      // CDN কে ১ ঘণ্টার জন্য ডেটা ক্যাশ করতে বলা হচ্ছে
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400'
    }
  });
}
```

## উপসংহার

Astro দিয়ে ডেভেলপমেন্ট জার্নি শুরু করা যেমন সহজ, ঠিক তেমনি প্রোডাকশন স্কেলে এটিকে অপ্টিমাইজ করাও দারুণ ফ্লেক্সিবল। আশা করি, বেসিক থেকে শুরু করে অ্যাডভান্সড টপিকের এই ১০টি ব্লগ সিরিজের মাধ্যমে আপনি Astro-এর কোর আর্কিটেকচার সম্পর্কে স্পষ্ট ধারণা পেয়েছেন। 

Happy coding with Astro! 🚀