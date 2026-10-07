---
title: "State Management Across Frameworks with Nano Stores"
description: "Astro প্রজেক্টে React, Vue বা Svelte আইল্যান্ডের মাঝে গ্লোবাল স্টেট শেয়ার এবং সিঙ্ক করার জন্য Nano Stores-এর বিস্তারিত গাইড।"
pubDate: "2026-10-07"
author: "Your Name"
tags: ["Astro", "Nano Stores", "State Management", "React", "Vue", "Frontend"]
category: "Web Development"
coverImage: "/assets/blog/astro-nanostores.jpg"
draft: false
---

Astro-এর **Islands Architecture** নিয়ে কাজ করার সময় ডেভেলপাররা সাধারণত একটি চমৎকার সমস্যার সম্মুখীন হন। আমরা জানি যে Astro-তে একই পেজে React, Vue, বা Svelte-এর মতো বিভিন্ন ফ্রেমওয়ার্কের কম্পোনেন্ট একসাথে ব্যবহার করা যায় (Bring Your Own Framework)। 

কিন্তু সমস্যা হলো, **এই ভিন্ন ভিন্ন ফ্রেমওয়ার্কের কম্পোনেন্টগুলো একে অপরের সাথে ডেটা শেয়ার করবে কীভাবে?**

ধরে নিন, আপনার ওয়েবসাইটের নেভিগেশন বারে একটি **Vue** কম্পোনেন্ট আছে যা শপিং কার্টের আইটেম সংখ্যা (Cart Counter) দেখায়। আর পেজের নিচের দিকে একটি প্রোডাক্ট কার্ডে **React** দিয়ে তৈরি "Add to Cart" বাটন আছে। React বাটনে ক্লিক করলে Vue কাউন্টারটি আপডেট হবে কীভাবে? React Context বা Vue-এর Vuex/Pinia তো একে অপরের ডেটা পড়তে পারে না!

এই সমস্যারই নিখুঁত সমাধান হলো **Nano Stores**।

## ১. Nano Stores কী এবং কেন?

**Nano Stores** হলো একটি ফ্রেমওয়ার্ক-অ্যাগনস্টিক (Framework-agnostic) গ্লোবাল স্টেট ম্যানেজমেন্ট লাইব্রেরি। 
* এটি অত্যন্ত লাইটওয়েট (মাত্র কয়েকশো বাইট)।
* এটি React, Vue, Svelte, Solid—সবার সাথেই কাজ করে।
* Astro টিমের রেকমেন্ডেড স্টেট ম্যানেজমেন্ট টুল হলো এই Nano Stores।

যেহেতু এটি কোনো নির্দিষ্ট ফ্রেমওয়ার্কের ওপর নির্ভরশীল নয়, তাই এটি ব্রাউজারের উইন্ডো লেভেলে স্টেট ধরে রাখে। ফলে যেকোনো ফ্রেমওয়ার্কের কম্পোনেন্ট সেই স্টেট পড়তে বা পরিবর্তন করতে পারে।

## ২. প্রজেক্টে Nano Stores ইনস্টল করা

প্রথমে আমাদের মেইন লাইব্রেরি এবং আমরা যেসব ফ্রেমওয়ার্ক ব্যবহার করবো, তাদের জন্য নির্দিষ্ট ইন্টিগ্রেশন প্যাকেজ ইনস্টল করতে হবে। আমাদের উদাহরণে আমরা React এবং Vue ব্যবহার করবো:

```bash
npm install nanostores @nanostores/react @nanostores/vue
```

## ৩. একটি Global Store তৈরি করা

প্রজেক্টের `src` ফোল্ডারের ভেতরে `store` নামে একটি ডিরেক্টরি তৈরি করে সেখানে `cartStore.js` (বা `.ts`) নামে একটি ফাইল তৈরি করুন। 

এখানে আমরা `atom` ব্যবহার করে একটি সিম্পল স্টেট তৈরি করবো:

```javascript
// src/store/cartStore.js
import { atom } from 'nanostores';

// কার্টের আইটেম সংখ্যা ট্র্যাক করার জন্য একটি atom তৈরি করা হলো। ডিফল্ট ভ্যালু 0।
export const cartCount = atom(0);

// কার্টে আইটেম যোগ করার জন্য একটি হেল্পার ফাংশন
export function addToCart() {
  const currentCount = cartCount.get();
  cartCount.set(currentCount + 1);
}
```

## ৪. React কম্পোনেন্ট থেকে State আপডেট করা

এবার আমরা একটি React বাটন তৈরি করবো, যেটিতে ক্লিক করলে `addToCart` ফাংশনটি ট্রিগার হবে।

```jsx
// src/components/ReactAddToCartButton.jsx
import React from 'react';
import { addToCart } from '../store/cartStore';

export default function ReactAddToCartButton() {
  return (
    <div className="react-island" style={{ border: '2px solid blue', padding: '10px' }}>
      <h3>React Component</h3>
      <button onClick={addToCart}>
        Add to Cart (React)
      </button>
    </div>
  );
}
```

খেয়াল করুন, এখানে আমরা শুধু স্টোর থেকে ফাংশনটি কল করে স্টেট আপডেট করছি।

## ৫. Vue কম্পোনেন্ট থেকে State রিড (Read) করা

এবার আমরা একটি Vue কম্পোনেন্ট বানাবো, যা সব সময় রিয়েল-টাইমে কার্টের ভ্যালু লিসেন (listen) করবে। এর জন্য আমরা `@nanostores/vue` এর `useStore` হুক ব্যবহার করবো।

```vue
<!-- src/components/VueCartCounter.vue -->
<template>
  <div class="vue-island" style="border: 2px solid green; padding: 10px;">
    <h3>Vue Component</h3>
    <p>🛒 Items in Cart: {{ count }}</p>
  </div>
</template>

<script setup>
import { useStore } from '@nanostores/vue';
import { cartCount } from '../store/cartStore';

// store-এর ভ্যালু সাবস্ক্রাইব করা হলো
const count = useStore(cartCount);
</script>
```

## ৬. Astro পেজে Islands রেন্ডার করা

সবশেষে, আমাদের মেইন `.astro` ফাইলে এই দুটি আলাদা ফ্রেমওয়ার্কের কম্পোনেন্ট ইম্পোর্ট করে রেন্ডার করবো।

```astro
---
// src/pages/index.astro
import VueCartCounter from '../components/VueCartCounter.vue';
import ReactAddToCartButton from '../components/ReactAddToCartButton.jsx';
---

<html>
  <head>
    <title>Nano Stores Demo</title>
  </head>
  <body>
    <h1>Astro Cross-Framework State Management</h1>
    
    <header>
      <!-- Vue Island -->
      <VueCartCounter client:load />
    </header>

    <main style="margin-top: 50px;">
      <!-- React Island -->
      <ReactAddToCartButton client:load />
    </main>
    
  </body>
</html>
```

*বিঃদ্রঃ যেহেতু কম্পোনেন্টগুলো ইউজার ইন্টারঅ্যাকশনের ওপর নির্ভর করে, তাই `client:load` বা `client:idle` ডিরেকটিভ ব্যবহার করতে ভুলবেন না!*

### ম্যাজিকটি দেখুন!
এখন আপনার ব্রাউজারে পেজটি রিলোড করে React বাটনে ক্লিক করুন। আপনি দেখবেন, React থেকে ফায়ার করা ইভেন্ট সাথে সাথেই আপনার Vue কম্পোনেন্টের ডেটা আপডেট করে দিচ্ছে! কোনো প্রপস ড্রিলিং (Props Drilling) নেই, কোনো জটিল Context Setup নেই। 

## উপসংহার

Astro-এর Islands Architecture-এর সাথে **Nano Stores**-এর কম্বিনেশন ডেভেলপারদের মাইক্রো-ফ্রন্টএন্ড (Micro-frontend) ডেভেলপমেন্টে এক নতুন স্বাধীনতা দেয়। আপনি আপনার প্রয়োজনমতো যেকোনো ফ্রেমওয়ার্ক ব্যবহার করতে পারবেন, আর তাদের মাঝে ডেটা সিঙ্ক করার দায়িত্ব নেবে Nano Stores।

পরবর্তী ব্লগে (Blog 8) আমরা জানবো **Building AI-Powered API Endpoints in Astro** নিয়ে, যেখানে আমরা Astro-এর সার্ভার রুট (API Routes) ব্যবহার করে Large Language Models (LLM) এবং AI API ইন্টিগ্রেট করা শিখবো!