---
title: "Enhancing UX: View Transitions and Advanced i18n Routing"
description: "Astro View Transitions API এবং i18n (Internationalization) রাউটিং ব্যবহার করে কীভাবে ওয়েবসাইটকে SPA-এর মতো ফাস্ট এবং মাল্টি-ল্যাঙ্গুয়েজ সাপোর্টেড করা যায়।"
pubDate: "2026-10-09"
author: "Pradipta Sarker"
tags: ["Astro", "View Transitions", "i18n", "UX", "Routing"]
---

Astro মূলত একটি **Multi-Page Application (MPA)** ফ্রেমওয়ার্ক। এর মানে হলো, ইউজার যখন এক পেজ থেকে অন্য পেজে যায়, তখন ব্রাউজার পুরো পেজটি নতুন করে রিলোড করে। কিন্তু মডার্ন ওয়েব ডেভেলপমেন্টে ইউজাররা **Single Page Application (SPA)**-এর মতো স্মুথ এক্সপেরিয়েন্স পছন্দ করে। 

Astro-তে এই স্মুথ এক্সপেরিয়েন্স আনার জন্যই ব্যবহার করা হয় **View Transitions API**। এর পাশাপাশি, গ্লোবাল অডিয়েন্সের জন্য ওয়েবসাইটকে একাধিক ভাষায় (Multilingual) রূপান্তর করতে **i18n (Internationalization)** রাউটিং অত্যন্ত জরুরি। এই ব্লগে আমরা এই দুটি অ্যাডভান্সড ফিচার নিয়ে আলোচনা করব।

## ১. View Transitions কী এবং কেন?

ব্রাউজারের নেটিভ **View Transitions API** ব্যবহার করে Astro পেজ রিলোড হওয়ার সময় চমৎকার অ্যানিমেশন বা ফেড-ইন/ফেড-আউট ইফেক্ট তৈরি করতে পারে। এর ফলে মনেই হবে্বা না যে আপনি একটি MPA ব্রাউজ করছেন; পুরো সাইটটি React বা Next.js-এর তৈরি একটি SPA-এর মতো রেসপন্সিভ মনে হবে।

### View Transitions ইমপ্লিমেন্ট করা

এটি সেটআপ করা জাদুর মতো সহজ। আপনার গ্লোবাল লেআউট ফাইলে (যেমন `src/layouts/Layout.astro`) শুধু `<ViewTransitions />` কম্পোনেন্টটি ইম্পোর্ট করে `head` ট্যাগের ভেতর বসিয়ে দিন:

```html
---
// src/layouts/Layout.astro
import { ViewTransitions } from 'astro:transitions';
---
<html lang="en">
  <head>
    <title>My Astro Site</title>
    <!-- এখানে ViewTransitions অ্যাড করা হলো -->
    <ViewTransitions/>
  </head>
  <body>
    <slot />
  </body>
</html>
```
ব্যাস! এখন আপনার সাইটের যেকোনো লিঙ্কে ক্লিক করলে পেজ রিলোড হওয়ার বদলে স্মুথলি ট্রানজিশন হবে। 

### কাস্টম অ্যানিমেশন (transition:name)

আপনি চাইলে একটি নির্দিষ্ট এলিমেন্ট (যেমন ব্লগের কভার ইমেজ) এক পেজ থেকে অন্য পেজে যাওয়ার সময় অ্যানিমেট করে পজিশন চেঞ্জ করাতে পারেন। এর জন্য এলিমেন্টটিতে `transition:name` ডিরেক্টিভ ব্যবহার করতে হয়:

```html
<!-- Page 1: Blog List -->
<img src="/images/hero.jpg" transition:name="hero-image" />

<!-- Page 2: Blog Details -->
<img src="/images/hero.jpg" transition:name="hero-image" />
```
Astro নিজে থেকেই এই দুটি ইমেজের মাঝে একটি চমৎকার মরফিং (Morphing) অ্যানিমেশন তৈরি করে দেবে।

## ২. Advanced i18n (Internationalization) Routing

আপনার প্রজেক্ট যদি একাধিক ভাষায় (যেমন ইংরেজি এবং বাংলা) সাপোর্ট করে, তবে Astro-এর বিল্ট-ইন i18n ফিচার ব্যবহার করে খুব সহজেই সাব-ডিরেক্টরি রাউটিং (Sub-directory Routing) তৈরি করতে পারবেন (যেমন: `/en/about` এবং `/bn/about`)।

### i18n কনফিগারেশন

প্রথমে `astro.config.mjs` ফাইলে গিয়ে আপনার ওয়েবসাইটের ভাষাগুলো ডিফাইন করে দিন:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'bn'], // ইংরেজি এবং বাংলা
    routing: {
      prefixDefaultLocale: false // 'en' এর জন্য /en/ দেখাবে না, শুধু / দেখাবে।
    }
  }
});
```

### ফোল্ডার স্ট্রাকচার

i18n কাজ করানোর জন্য আপনার `src/pages` ডিরেক্টরিকে সেভাবে সাজাতে হবে:

```text
src/
 └── pages/
      ├── index.astro       (ডিফল্ট ইংরেজি হোমপেজ -> /)
      ├── about.astro       (ডিফল্ট ইংরেজি এবাউট পেজ -> /about)
      └── bn/
           ├── index.astro  (বাংলা হোমপেজ -> /bn/)
           └── about.astro  (বাংলা এবাউট পেজ -> /bn/about)
```

### Language Switcher তৈরি করা

ইউজার যাতে সহজেই ভাষা পরিবর্তন করতে পারে, তার জন্য Astro-এর `getRelativeLocaleUrl` হেল্পার ফাংশন ব্যবহার করতে পারেন:

```html
---
// LanguageSwitcher.astro
import { getRelativeLocaleUrl } from 'astro:i18n';
---
<nav>
  <!-- বর্তমান পেজের লিঙ্কের উপর ভিত্তি করে ল্যাঙ্গুয়েজ সুইচ করুন -->
  <a href={getRelativeLocaleUrl('en', 'about')}>English</a>
  <a href={getRelativeLocaleUrl('bn', 'about')}>বাংলা</a>
</nav>
```

View Transitions এবং i18n-এর এই কম্বিনেশন আপনার Astro সাইটকে করে তুলবে সুপার ফাস্ট, স্মুথ এবং গ্লোবাল ইউজারদের জন্য ফ্রেন্ডলি।