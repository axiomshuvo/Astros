---
title: "Building AI-Powered API Endpoints in Astro"
description: "Astro-এর Server (API) Routes ব্যবহার করে কীভাবে কাস্টম ব্যাকএন্ড লজিক এবং LLM (Large Language Models) API ইন্টিগ্রেশন করতে হয় তার গাইড।"
pubDate: "2026-10-10"
author: "Your Name"
tags: ["Astro", "API", "Backend", "AI", "LLM", "Full-stack"]
---

Astro-কে অনেকেই শুধুমাত্র একটি **Static Site Generator (SSG)** হিসেবে চেনে। কিন্তু রিয়েলিটি হলো, Astro একটি অত্যন্ত শক্তিশালী **Full-stack Framework**। এর **Server Routes** বা **API Endpoints** ব্যবহার করে আপনি খুব সহজেই কাস্টম ব্যাকএন্ড লজিক লিখতে পারেন। 

আজকাল ওয়েব ডেভেলপমেন্টে **AI (Artificial Intelligence)** এবং **LLM (Large Language Model)**-এর ব্যবহার দ্রুত বাড়ছে। এই ব্লগে আমরা দেখব কীভাবে Astro-এর API Routes ব্যবহার করে একটি **AI-Powered API Endpoint** তৈরি করা যায়, যা Google Gemini বা OpenAI-এর মতো LLM-এর সাথে কথা বলতে পারবে।

## ১. Astro-তে API Routes কী?

Astro-তে API Endpoint তৈরি করা খুবই সহজ। `src/pages/` ডিরেক্টরির ভেতর আপনি যদি `.js` বা `.ts` এক্সটেনশনের কোনো ফাইল তৈরি করেন, Astro সেটিকে একটি **Server Route** বা **API Endpoint** হিসেবে ধরে নেয়। 

এই ফাইলগুলোতে কোনো `.astro` কম্পোনেন্ট বা HTML থাকে না। বরং, এগুলো স্ট্যান্ডার্ড **Web API**-এর `Request` এবং `Response` অবজেক্ট রিটার্ন করে। ডেটাবেস ফেচ করা, ইউজার অথেনটিকেশন বা এক্সটার্নাল API কল করার জন্য এগুলো পারফেক্ট।

## ২. SSR বা Hybrid Mode এনাবল করা

যেহেতু API Endpoints সার্ভার-সাইডে রান করে, তাই প্রজেক্টে **Server-Side Rendering (SSR)** বা **Hybrid Rendering** এনাবল থাকতে হবে। `astro.config.mjs` ফাইলে গিয়ে `output` মুড চেঞ্জ করে দিন:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'hybrid', // প্রজেক্ট স্ট্যাটিক থাকবে, শুধু নির্দিষ্ট পেজ/এপিআই সার্ভারে রান করবে
  adapter: node({
    mode: 'standalone',
  }),
});
```

## ৩. Environment Variables সেটআপ (API Key সিকিউরিটি)

AI API কল করার জন্য আমাদের একটি **Secret API Key** লাগবে। ভুলেও কখনো API Key ফ্রন্টএন্ডে বা গিটহাবে পুশ করবেন না! প্রজেক্টের রুট ডিরেক্টরিতে একটি `.env` ফাইল তৈরি করুন:

```env
# .env
GEMINI_API_KEY="your_api_key_here"
```

Astro-তে সার্ভার সাইড থেকে এটি এক্সেস করতে আমরা `import.meta.env.GEMINI_API_KEY` ব্যবহার করব। 

## ৪. AI-Powered API Endpoint তৈরি করা

এবার আমরা একটি POST রিকোয়েস্ট হ্যান্ডেল করব, ক্লায়েন্ট থেকে ইউজারের প্রম্পট (Prompt) নেব, সেটি AI-এর কাছে পাঠাব এবং রেসপন্সটি ক্লায়েন্টকে ফেরত দেব।

`src/pages/api/generate.ts` নামে ফাইল তৈরি করুন:

```typescript
// src/pages/api/generate.ts
import type { APIRoute } from 'astro';

export const POST: APIRoute = async ({ request }) => {
  try {
    // ১. Request Body থেকে ইউজারের ডেটা (prompt) রিসিভ করা
    const body = await request.json();
    const userPrompt = body.prompt;

    if (!userPrompt) {
      return new Response(JSON.stringify({ error: "Prompt is required!" }), {
        status: 400,
      });
    }

    // ২. AI API-তে কল করা (Gemini API-এর উদাহরণ)
    const apiKey = import.meta.env.GEMINI_API_KEY;
    const aiResponse = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:generateContent?key=${apiKey}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        contents: [{ parts: [{ text: userPrompt }] }]
      }),
    });

    const aiData = await aiResponse.json();
    const generatedText = aiData.candidates[0].content.parts[0].text;

    // ৩. ক্লায়েন্টকে JSON রেসপন্স পাঠানো
    return new Response(
      JSON.stringify({
        success: true,
        data: generatedText,
      }),
      {
        status: 200,
        headers: { "Content-Type": "application/json" },
      }
    );

  } catch (error) {
    console.error("AI API Error:", error);
    return new Response(
      JSON.stringify({ error: "Failed to generate AI response." }),
      { status: 500 }
    );
  }
};
```

## ৫. Frontend থেকে API Call করা

আমাদের AI API রেডি! এবার আমরা আমাদের যেকোনো `.astro`, `React` বা `Vue` কম্পোনেন্ট থেকে এই API-তে কল করতে পারব। 

ধরা যাক, একটি `Astro` পেজে আমরা ক্লায়েন্ট-সাইড জাভাস্ক্রিপ্ট দিয়ে কল করব:

```html
---
// src/pages/ai-chat.astro
---

<html>
  <body>
    <h1>Ask the AI</h1>
    <input type="text" id="promptInput" placeholder="Ask anything..." />
    <button id="askBtn">Generate</button>
    
    <div id="resultBox" style="margin-top: 20px; padding: 10px; border: 1px solid #ccc;"></div>

    <script>
      const btn = document.getElementById('askBtn');
      const input = document.getElementById('promptInput');
      const resultBox = document.getElementById('resultBox');

      btn.addEventListener('click', async () => {
        const prompt = input.value;
        resultBox.innerHTML = "Thinking...";

        const res = await fetch('/api/generate', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ prompt })
        });

        const json = await res.json();
        
        if (json.success) {
          resultBox.innerHTML = json.data;
        } else {
          resultBox.innerHTML = "Error occurred!";
        }
      });
    </script>
  </body>
</html>
```

## উপসংহার

Astro-এর Server Routes ফ্রেমওয়ার্কটিকে শুধুমাত্র একটি স্ট্যাটিক টুল থেকে ফুল-ফ্লেজেড ব্যাকএন্ড সলিউশনে পরিণত করেছে। কোনো আলাদা Node.js সার্ভার ছাড়াই এখন আপনি সরাসরি Astro-এর ভেতরই AI ইন্টিগ্রেশন বা ফর্ম হ্যান্ডলিং-এর মতো জটিল কাজগুলো সেরে ফেলতে পারবেন।