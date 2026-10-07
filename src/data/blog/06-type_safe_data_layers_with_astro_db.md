---
title: "Building Type-safe Data Layers with Astro DB"
description: "Astro DB সেটআপ, স্কিমা ডিফাইন করা এবং libSQL দিয়ে লোকাল ও রিমোট ডেটাবেস ম্যানেজ করার অ্যাডভান্সড গাইড।"
pubDate: "2026-10-06"
author: "Your Name"
tags: ["Astro", "Astro DB", "Database", "libSQL", "TypeScript", "Backend"]
category: "Web Development"
coverImage: "/assets/blog/astro-db.jpg"
draft: false
---

Astro মূলত শুরু হয়েছিল স্ট্যাটিক সাইট জেনারেটর (SSG) হিসেবে। এরপর তারা SSR (Server-Side Rendering) নিয়ে আসে। কিন্তু ২০২৪ সালের শুরুতে Astro এমন একটি ফিচার লঞ্চ করেছে যা ওয়েব ডেভেলপমেন্ট কমিউনিটিতে রীতিমতো আলোড়ন তুলেছে—আর তা হলো **Astro DB**।

আজকের ব্লগে আমরা জানবো Astro DB কী, এটি কীভাবে কাজ করে এবং কীভাবে আপনি আপনার প্রজেক্টে একটি সম্পূর্ণ Type-safe রিলেশনাল ডেটাবেস যুক্ত করতে পারেন।

## ১. Astro DB আসলে কী?

**Astro DB** হলো Astro-এর জন্য বিশেষভাবে ডিজাইন করা একটি সম্পূর্ণ ম্যানেজড SQL ডেটাবেস (Managed SQL Database)। 

এর পেছনের মূল প্রযুক্তিগুলো হলো:
*   **libSQL:** এটি মূলত জনপ্রিয় SQLite-এর একটি ফর্ক (fork), যা Edge network-এ দারুণ পারফর্ম করে।
*   **Drizzle ORM:** ডেটাবেস কুয়েরি করার জন্য Astro DB হুডের নিচে Drizzle ORM ব্যবহার করে, যা ডেভেলপারদের ১০০% Type-safe ডেটাবেস এক্সপেরিয়েন্স দেয়।

এর সবচেয়ে বড় সুবিধা হলো, ডেটাবেস সেটআপ করার জন্য আপনাকে কোনো থার্ড-পার্টি সার্ভিস (যেমন- Supabase, Firebase বা MongoDB) এর উপর নির্ভর করতে হবে না। সবকিছু Astro প্রজেক্টের ভেতরেই লোকালি এবং প্রোডাকশনে কাজ করবে!

## ২. প্রজেক্টে Astro DB সেটআপ করা

আপনার বিদ্যমান Astro প্রজেক্টে Astro DB যোগ করা জাদুর মতোই সহজ। টার্মিনালে শুধু রান করুন:

```bash
npx astro add db
```

এই কমান্ডটি আপনার প্রজেক্টে প্রয়োজনীয় প্যাকেজ ইনস্টল করবে এবং রুট ডিরেক্টরিতে `db/` নামে একটি ফোল্ডার তৈরি করবে। এই ফোল্ডারের ভেতর দুটি গুরুত্বপূর্ণ ফাইল থাকবে: `config.ts` এবং `seed.ts`।

## ৩. Database Schema তৈরি করা (`config.ts`)

যেকোনো রিলেশনাল ডেটাবেসের প্রথম কাজ হলো টেবিলের স্কিমা (Schema) ডিফাইন করা। Astro DB-তে এটি TypeScript দিয়ে করা হয়।

ধরে নিন, আমরা একটি কমেন্ট সিস্টেম (Comment System) বানাতে চাই। এর জন্য `db/config.ts` ফাইলে আমরা `Comment` নামের একটি টেবিল ডিফাইন করবো:

```typescript
// db/config.ts
import { defineDb, defineTable, column, NOW } from 'astro:db';

const Comment = defineTable({
  columns: {
    id: column.number({ primaryKey: true }), // অটো-ইনক্রিমেন্ট প্রাইমারি কি
    author: column.text(),                   // ইউজারের নাম
    content: column.text(),                  // কমেন্টের মূল টেক্সট
    publishedAt: column.date({ default: NOW }), // পাবলিশ হওয়ার সময়
  }
});

// ডেটাবেস এক্সপোর্ট করা
export default defineDb({
  tables: { Comment },
});
```

খেয়াল করুন, আমরা সরাসরি TypeScript কোড লিখে টেবিল ডিফাইন করছি। এর মানে হলো, আমরা যখন ডেটা ফেচ করবো, তখন আমাদের কোড এডিটর (VS Code) স্বয়ংক্রিয়ভাবে অটোকমপ্লিট সাজেশন দেবে!

## ৪. Seed Data ইনসার্ট করা (`seed.ts`)

ডেভেলপমেন্টের সময় টেস্ট করার জন্য কিছু ডামি ডেটা (Dummy Data) দরকার হয়। Astro DB-তে `seed.ts` ফাইলটি ঠিক এই কাজের জন্যই ব্যবহৃত হয়।

```typescript
// db/seed.ts
import { db, Comment } from 'astro:db';

export default async function seed() {
  await db.insert(Comment).values([
    { author: 'Rahim', content: 'Astro DB is mind-blowing!' },
    { author: 'Karim', content: 'I love how type-safe it is.' },
  ]);
  
  console.log('Seed data inserted successfully!');
}
```

ডেভেলপমেন্ট সার্ভার (`npm run dev`) চালু থাকলে Astro প্রতিবার স্টার্ট হওয়ার সময় এই লোকাল ডেটাবেসটি স্বয়ংক্রিয়ভাবে রিফ্রেশ করে এবং ডামি ডেটাগুলো ইনসার্ট করে দেয়।

## ৫. Astro ফাইলে ডেটা Query করা

ডেটাবেস সেটআপ এবং ডেটা ইনসার্ট করা শেষ। এবার চলুন আমাদের পেজে এই ডেটাগুলো দেখাই। 

যেহেতু Astro DB সার্ভার-সাইডে কাজ করে, তাই আমরা সরাসরি আমাদের `.astro` ফাইলের Component Script-এ ডেটাবেস কুয়েরি করতে পারবো:

```astro
---
// src/pages/comments.astro
import { db, Comment } from 'astro:db';

// ডেটাবেস থেকে সব কমেন্ট ফেচ করা
const comments = await db.select().from(Comment);
---

<html>
  <body>
    <h1>User Comments</h1>
    
    <div class="comments-list">
      {comments.map((comment) => (
        <article class="comment">
          <h3>{comment.author}</h3>
          <p>{comment.content}</p>
          <small>Posted at: {comment.publishedAt.toLocaleDateString()}</small>
        </article>
      ))}
    </div>
  </body>
</html>
```

এখানে `db.select().from(Comment)` লেখার সময় আপনি খেয়াল করবেন যে TypeScript আপনাকে ১০০% Type-safety দিচ্ছে। আপনি যদি `comment.content` এর জায়গায় ভুল করে `comment.body` লেখেন, তবে বিল্ড টাইমেই Astro এরর ধরিয়ে দেবে।

## ৬. প্রোডাকশন এবং Astro Studio

লোকাল ডেভেলপমেন্ট তো হলো, কিন্তু সাইট যখন লাইভ করবেন তখন ডেটাবেস কোথায় থাকবে? 

এর জন্য Astro নিয়ে এসেছে **Astro Studio** (studio.astro.build)। এটি হলো Astro-এর নিজস্ব ক্লাউড ডেটাবেস হোস্টিং। মাত্র একটি কমান্ড (`npx astro db push`) দিয়ে আপনি আপনার লোকাল স্কিমা এবং ডেটাবেস ক্লাউডে ডিপ্লয় করতে পারবেন।

## উপসংহার

Astro DB ফ্রন্টএন্ড ডেভেলপারদের জন্য ফুল-স্ট্যাক অ্যাপ্লিকেশন তৈরি করা অবিশ্বাস্য রকম সহজ করে দিয়েছে। আপনাকে জটিল SQL কুয়েরি লিখতে হবে না, আবার Type-safety নিয়েও ভাবতে হবে না।

সিরিজের পরবর্তী ব্লগে (Blog 6) আমরা জানবো Astro-এর রেন্ডারিং মোডগুলো সম্পর্কে, বিশেষ করে **Hybrid Rendering এবং Server Islands** নিয়ে, যা ডাইনামিক কন্টেন্ট ডেলিভারিতে এক নতুন যুগের সূচনা করেছে!