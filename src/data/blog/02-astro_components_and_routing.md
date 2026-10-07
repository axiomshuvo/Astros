---
title: "Astro Components, Layouts, and File-Based Routing"
description: "Astro প্রজেক্ট ফোল্ডার স্ট্রাকচার, .astro কম্পোনেন্ট তৈরি, Reusable Layouts এবং File-based Routing কীভাবে কাজ করে তার বিস্তারিত গাইড।"
pubDate: "2026-10-02"
author: "Your Name"
tags: ["Astro", "Components", "Layouts", "Routing", "Frontend"]
category: "Web Development"
coverImage: "/assets/blog/astro-components-routing.jpg"
draft: false
---

আমাদের আগের ব্লগে আমরা দেখেছি Astro-এর "Zero-JS" আর্কিটেকচার কীভাবে কাজ করে এবং কীভাবে একটি প্রজেক্ট সেটআপ করতে হয়। আজ আমরা জানবো Astro-এর মূল ভিত্তি নিয়ে—কীভাবে **Astro Components** তৈরি করতে হয়, **Layouts** দিয়ে কীভাবে কোড রিইউজ করা যায় এবং **File-Based Routing** সিস্টেম কীভাবে কাজ করে।

যেকোনো মডার্ন ওয়েব ফ্রেমওয়ার্কের মতো Astro-তেও সবকিছুই কম্পোনেন্ট ভিত্তিক। চলুন শুরু থেকে দেখা যাক।

## ১. The Astro Component (`.astro` Files)

Astro-এর নিজস্ব কম্পোনেন্ট ফাইলের এক্সটেনশন হলো `.astro`। এটি দেখতে অনেকটা সাধারণ HTML ফাইলের মতোই, তবে এর ভেতরে দারুণ কিছু সুপারপাওয়ার রয়েছে। একটি `.astro` ফাইলকে প্রধানত দুটি অংশে ভাগ করা যায়:

### Component Script (Frontmatter)

ফাইলের একেবারে শুরুতে তিনটি ড্যাশ (`---`) দিয়ে যে কোড ব্লকটি লেখা হয়, তাকে বলা হয় **Component Script**। এটি মূলত সার্ভার-সাইড জাভাস্ক্রিপ্ট (বা টাইপস্ক্রিপ্ট)। এখানে আপনি ভেরিয়েবল ডিক্লেয়ার করতে পারেন, এপিআই থেকে ডেটা ফেচ করতে পারেন এবং অন্যান্য কম্পোনেন্ট ইম্পোর্ট করতে পারেন। এখানকার কোনো কোডই ইউজারের ব্রাউজারে যায় না।

### Component Template

স্ক্রিপ্ট ব্লকের ঠিক নিচেই থাকে **Component Template**। এটি মূলত HTML-এর মতোই, যেখানে আপনি JSX-এর মতো সিনট্যাক্স ব্যবহার করে ডাইনামিক ডেটা রেন্ডার করতে পারেন।

**উদাহরণ:**
চলুন একটি সিম্পল `Greeting.astro` কম্পোনেন্ট তৈরি করি:

```astro
---
// 1. Component Script (Runs on the Server/Build-time)
const greeting = "Hello, Astro!";
const skills = ["HTML", "CSS", "JavaScript", "Astro"];
---

<!-- 2. Component Template (Renders to Static HTML) -->
<div class="greeting-card">
  <h1>{greeting}</h1>
  <p>My top skills are:</p>
  <ul>
    {skills.map((skill) => (
      <li>{skill}</li>
    ))}
  </ul>
</div>
```

## ২. Component Props ব্যবহার করা

একটি কম্পোনেন্টকে ডাইনামিক এবং রিইউজেবল (Reusable) করতে হলে আমাদের এক কম্পোনেন্ট থেকে অন্য কম্পোনেন্টে ডেটা পাস করতে হয়। Astro-তে এটি `Astro.props` এর মাধ্যমে খুব সহজেই করা যায়।

ধরুন, আমরা একটি `Card.astro` কম্পোনেন্ট বানাবো:

```astro
---
// Card.astro
const { title, description } = Astro.props;
---

<article class="card">
  <h2>{title}</h2>
  <p>{description}</p>
</article>
```

এবার আমরা অন্য যেকোনো পেজে এই `Card` কম্পোনেন্টটি ইম্পোর্ট করে আলাদা আলাদা ডেটা পাস করতে পারবো:

```astro
---
import Card from '../components/Card.astro';
---

<Card title="Astro is Fast" description="It ships zero JavaScript by default." />
<Card title="Easy to Learn" description="Feels just like standard HTML." />
```

## ৩. Layouts এবং `<slot />` এর ব্যবহার

একটি ওয়েবসাইটের প্রতিটি পেজে সাধারণত কিছু কমন অংশ থাকে, যেমন- `<head>`, Navbar, এবং Footer। প্রতিটি পেজে এগুলো বারবার না লিখে আমরা একটি **Layout Component** তৈরি করতে পারি।

Astro-তে Layout তৈরি করা অন্য যেকোনো কম্পোনেন্ট তৈরির মতোই। তবে এখানে মূল জাদুটি দেখায় `<slot />` ট্যাগ। `<slot />` হলো এমন একটি প্লেসহোল্ডার, যেখানে আপনি Layout-এর ভেতরে চাইল্ড (Child) কন্টেন্ট ইনজেক্ট করতে পারেন।

চলুন একটি `BaseLayout.astro` তৈরি করি:

```astro
---
// src/layouts/BaseLayout.astro
import Navbar from '../components/Navbar.astro';
import Footer from '../components/Footer.astro';

const { pageTitle } = Astro.props;
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>{pageTitle} | My Astro Blog</title>
  </head>
  <body>
    <Navbar />
    
    <main>
      <!-- এখানে আমাদের স্পেসিফিক পেজের কন্টেন্ট রেন্ডার হবে -->
      <slot />
    </main>

    <Footer />
  </body>
</html>
```

এবার এই Layout-টি আমরা আমাদের পেজে ব্যবহার করতে পারি:

```astro
---
// src/pages/about.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout pageTitle="About Me">
  <h1>Welcome to the About Page</h1>
  <p>This content will be injected exactly where the slot tag is in the layout!</p>
</BaseLayout>
```

## ৪. File-Based Routing

Astro-তে রাউটিং (Routing) সিস্টেমটি অত্যন্ত সহজ এবং ইনটুইটিভ। এটি **File-based Routing** ব্যবহার করে। অর্থাৎ, আপনাকে আলাদা করে কোনো রাউটার কনফিগার করতে হবে না। 

আপনার প্রজেক্টের `src/pages/` ডিরেক্টরির ভেতরে আপনি যে স্ট্রাকচারে ফাইল রাখবেন, আপনার ওয়েবসাইটের URL-ও ঠিক সেভাবেই তৈরি হবে।

*   `src/pages/index.astro` ➔ `yoursite.com/` (Home page)
*   `src/pages/about.astro` ➔ `yoursite.com/about`
*   `src/pages/blog/index.astro` ➔ `yoursite.com/blog/`
*   `src/pages/blog/post-1.md` ➔ `yoursite.com/blog/post-1`

খেয়াল করুন, Astro পেজ হিসেবে শুধু `.astro` ফাইলই নয়, বরং সরাসরি `.md` (Markdown) ফাইলও সাপোর্ট করে। এটি কন্টেন্ট-ফোকাসড সাইটের জন্য দারুণ একটি ফিচার।

### Dynamic Routing (এক নজরে)

কখনো কখনো আমাদের ডাইনামিক URL প্রয়োজন হয়, যেমন `yoursite.com/blog/1`, `yoursite.com/blog/2` ইত্যাদি। এর জন্য Astro-তে `[id].astro` বা `[slug].astro` এর মতো ব্র্যাকেট সিনট্যাক্স ব্যবহার করে ডাইনামিক রুট তৈরি করা যায়। এ সম্পর্কে আমরা ডেটা ফেচিং এবং কন্টেন্ট কালেকশন ব্লগে আরও বিস্তারিত জানবো।

## উপসংহার

Astro Components অত্যন্ত লাইটওয়েট এবং ডেভেলপার-ফ্রেন্ডলি। JSX-এর মতো শক্তিশালী টেমপ্লেটিং ল্যাঙ্গুয়েজের সাথে সাধারণ HTML-এর কম্বিনেশন একে শেখার জন্য খুব সহজ করে তুলেছে। আর Layouts ও File-based Routing দিয়ে দ্রুত একটি ওয়েবসাইটের স্কেলিটন দাঁড় করিয়ে ফেলা যায়।

পরবর্তী ব্লগে (Blog 3) আমরা Astro-এর সবচেয়ে আলোচিত ফিচার—**Islands Architecture** এবং **Bring Your Own Framework (BYOF)** নিয়ে কথা বলবো, যেখানে আমরা দেখবো কীভাবে এই স্ট্যাটিক সাইটের ভেতরে React বা Vue-এর মতো ফ্রেমওয়ার্ক দিয়ে ইন্টারঅ্যাক্টিভিটি যুক্ত করা যায়।