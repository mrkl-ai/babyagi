<h1 align="center">
 babyagi
</h1>

# উদ্দেশ্য

এই পাইথন স্ক্রিপ্টটি একটি এআই-শক্তিমত্তা সম্পন্ন টাস্ক ম্যানেজমেন্ট সিস্টেমের উদাহরণ। সিস্টেমটি OpenAI এবং vector ডেটাবেইজ ব্যবহার করে, যেমন ক্রোমা বা উইভিয়েট ব্যবহার করে কাজগুলি তৈরি করতে, অগ্রাধিকার দিতে এবং কার্যকর করতে। এই সিস্টেমের মূল ধারণা হল, এটি পূর্ববর্তী কাজের ফলাফল এবং একটি পূর্বনির্ধারিত উদ্দেশ্যের উপর ভিত্তি করে কাজ তৈরি করে থাকে। স্ক্রিপ্টটি তখন উদ্দেশ্যের উপর ভিত্তি করে নতুন কাজ তৈরি করতে ওপেনএআই-এর ন্যাচারাল ল্যাংগুয়েজ প্রসেসিং (NLP) ক্ষমতা ব্যবহার করে, এবং প্রেক্ষাপটের জন্য টাস্ক ফলাফল সংরক্ষণ এবং পুনরুদ্ধারে ক্রোমা/উইভিয়েট ব্যবহার করে থাকে। এটি [Task-Driven Autonomous Agent](https://twitter.com/yoheinakajima/status/1640934493489070080?s=20) (Mar 28, 2023) এর একটি পেয়ার্ড ডাউন ভার্সন।


README নিম্নলিখিত বিষয়গুলো কাভার করবে:

- [স্ক্রিপট যেভাবে কাজ করে](#how-it-works)

- [যেভাবে ব্যবহার করা যাবে](#how-to-use)

- [সাপোর্টেড মডেলসমূহ](#supported-models)

- [ক্রমাগত স্ক্রিপ্ট চালাতে সতর্কতা](#continous-script-warning)

# স্ক্রিপ্ট যেভাবে কাজ করে<a name="how-it-works"></a>

স্ক্রিপ্টটি একটি অসীম লুপ চালানোর মাধ্যমে নিম্নলিখিত পদক্ষেপগুলি সম্পন্ন করে:

1. টাস্ক লিস্ট থেকে প্রথম টাস্ক টানে।
2. এক্সিকিউশন এজেন্টের কাছে টাস্কটি পাঠায়, যেটি প্রেক্ষাপটের উপর ভিত্তি করে কাজটি সম্পূর্ণ করতে OpenAI এর API ব্যবহার করে থাকে।
3. ফলাফলটি সমৃদ্ধ করে এবং এখানে  [Chroma](https://docs.trychroma.com)/[Weaviate](https://weaviate.io/) সংরক্ষণ করে।
4. নতুন টাস্ক তৈরি করে এবং আগের টাস্কের উদ্দেশ্য এবং ফলাফলের উপর ভিত্তি করে টাস্ক লিস্টকে পুনরায় প্রাধান্য দেয়।
   </br>

![image](https://user-images.githubusercontent.com/21254008/235015461-543a897f-70cc-4b63-941a-2ae3c9172b11.png)

`execution_agent()` ফাংশনে OpenAI এর API ব্যবহার করা হয়। এটির দুইটি প্যারামিটার লাগে: উদ্দেশ্য এবং টাস্ক। এরপর এটি OpenAI API এ একটি প্রম্পট পাঠায়, যা টাস্কের ফলাফল প্রদান করে। প্রম্পটে AI সিস্টেমের টাস্ক, উদ্দেশ্য এবং টাস্কের বিবরণ থাকে। এরপর ফলাফলটি একটি স্ট্রিং হিসাবে ফিরে আসে।

`task_creation_agent()` ফাংশন হল যেখানে OpenAI এর API উদ্দেশ্য এবং পূর্ববর্তী টাস্কের ফলাফলের উপর ভিত্তি করে নতুন টাস্ক তৈরি করতে ব্যবহার করা হয়। ফাংশনটি চারটি প্যারামিটার গ্রহন করে: উদ্দেশ্য, পূর্ববর্তী টাস্কের ফলাফল, টাস্কের বিবরণ এবং বর্তমান টাস্ক লিস্ট। তারপরে এটি OpenAI এর API-তে একটি প্রম্পট পাঠায়, যা স্ট্রিং হিসাবে নতুন কাজের একটি তালিকা প্রদান করে। ফাংশনটি এরপর নতুন টাস্কগুলিকে একটি ডিকশনারির লিস্ট হিসাবে ফিরিয়ে দেয়, যেখানে প্রতিটি ডিকশনারিতে টাস্কের নাম থাকে।

`prioritization_agent()` ফাংশন OpenAI এর API টাস্ক লিস্টকে পুনঃঅগ্রাধিকার দেয়ার জন্য ব্যবহার করা হয়। ফাংশনটি একটি প্যারামিটার নেয়, বর্তমান টাস্কের আইডি। এটি OpenAI এর API-তে একটি প্রম্পট পাঠায়, যা পুনঃঅগ্রাধিকার টাস্ক লিস্টকে একটি সংখ্যাযুক্ত লিস্ট হিসাবে প্রদান করে।

অবশেষে, স্ক্রিপ্টটি প্রেক্ষাপটের জন্য টাস্কের ফলাফলগুলি সংরক্ষণ এবং পুনরুদ্ধার করতে Chroma/Weaviate ব্যবহার করে। স্ক্রিপ্টটি TABLE_NAME ভেরিয়েবলে নির্দিষ্ট করা টেবিল নামের উপর ভিত্তি করে একটি Chroma/Weaviate সংগ্রহ তৈরি করে। এরপরে ক্রোমা/উইভিয়েট ব্যবহার করে টাস্কের নাম এবং অতিরিক্ত মেটাডেটা সহ টাস্কের ফলাফল সংরক্ষণ করতে ব্যবহৃত হয়।

# যেভাবে ব্যবহার করা যাবে<a name="how-to-use"></a>

স্ক্রিপ্ট ব্যবহার করতে, নিম্নলিখিত পদক্ষেপ অনুসরণ করতে হবে:

1. রিপোজিটরি ক্লোন `git clone https://github.com/yoheinakajima/babyagi.git` এবং `cd` এর মাধ্যমে ডিরেক্টোরিতে প্রবেশ করুন।
2. প্রয়োজনীয় প্যাকেজ ইনস্টল করতে: `pip install -r requirements.txt`
3. .env.example ফাইলটি .env তে কপি করুন: `cp .env.example .env` এখানে আপনি নিম্নলিখিত ভেরিয়েবল সেট করবেন।
4. OPENAI_API_KEY এবং OPENAPI_API_MODEL ভেরিয়েবলে আপনার OpenAI API কী সেট করুন। Weaviate এর সাথে ব্যবহার করার জন্য আপনাকে  অতিরিক্ত ভেরিয়েবল সেটআপ করতে হবে [এখানে](docs/weaviate.md)।
5. টেবিলের নাম সেট করুন যেখানে টাস্ক ফলাফলগুলি TABLE_NAME ভেরিয়েবলে সংরক্ষণ করা হবে৷
6. (ঐচ্ছিক) BABY_NAME ভেরিয়েবলে BabyAGI উদাহরণের নাম সেট করুন।
7. (ঐচ্ছিক) Objective ভেরিয়েবলে টাস্ক ম্যানেজমেন্ট সিস্টেমের উদ্দেশ্য সেট করুন।
8. (ঐচ্ছিক) সিস্টেমের প্রথম কাজটি INITIAL_TASK ভেরিয়েবলে সেট করুন।
9. স্ক্রিপ্টটি চালান: `python babyagi.py`

উপরের সকল ঐচ্ছিক মানগুলো কমান্ড লাইনেও সেট করা যাবে।

# Docker container এ চালতে হলে

পূর্বশর্ত হিসাবে, আপনার docker এবং docker-compose ইনস্টল থাকতে হবে। docker desktop হল সবচেয়ে সহজ বিকল্প https://www.docker.com/products/docker-desktop/

docker কন্টেইনারে সিস্টেম চালানোর জন্য উপরের ধাপ অনুযায়ী আপনার .env ফাইল সেটআপ করুন এবং তারপরে নিম্নলিখিত কমান্ড রান করুন:

```
docker-compose up
```

# সাপোর্টেড মডেলসমূহ<a name="supported-models"></a>

এই স্ক্রিপ্টটি সকল OpenAI মডেলের সাথে কাজ করে, একই সাথে Llama এবং Llama.cpp এর সাহায্যে এর অন্যান্য প্রকরনের সাথওে কাজ করে। ডিফল্ট মডেল হল **gpt-3.5-টার্বো**। ভিন্ন মডেল ব্যবহার করতে, LLM_MODEL এর মাধ্যমে এটি নির্দিষ্ট করুন বা কমান্ড লাইন ব্যবহার করুন৷

## Llama

Llama ইন্টিগ্রেশনের জন্য llama-cpp প্যাকেজ প্রয়োজন। আপনার Llama model weights ও প্রয়োজন হবে৷

- **কোন অবস্থাতেই আইপিএফএস, ম্যাগনেট লিঙ্ক বা মডেল ডাউনলোডের অন্য কোনও লিঙ্ক এই রিপোজিটরির কোথাও শেয়ার করবেন না, যার মধ্যে ইস্যু, ডিসকাশন বা পুল রিকুয়েষ্ট অন্তর্ভুক্ত। সেগুলো অবিলম্বে মুছে ফেলা হবে।**

যখন আপনার কাছে প্রয়োজনীয় সবকিছু থাকবে, তখন ব্যবহার করার জন্য নির্দিষ্ট মডেলের পাথে LLAMA_MODEL_PATH সেট করুন৷ আপনার সুবিধার জন্য, আপনি BabyAGI রেপোতে `models` লিঙ্ক করতে পারেন সেই ফোল্ডারে যেখানে আপনার Llama model weights আছে। তারপর `LLM_MODEL=llama` বা `-l` আর্গুমেন্ট দিয়ে স্ক্রিপ্টটি চালান।

# সতর্কতা<a name="continous-script-warning"></a>

এই স্ক্রিপ্টটি টাস্ক ম্যানেজমেন্ট সিস্টেমের অংশ হিসাবে ক্রমাগত চালানোর জন্য ডিজাইন করা হয়েছে। এই স্ক্রিপ্টটি ক্রমাগত চালানোর ফলে উচ্চ API ব্যবহার হতে পারে, তাই দয়া করে দায়িত্বের সাথে এটি ব্যবহার করুন। উপরন্তু, স্ক্রিপ্টের জন্য OpenAI API সঠিকভাবে সেট আপ করা প্রয়োজন, তাই স্ক্রিপ্ট চালানোর আগে নিশ্চিত করুন যে আপনি API সেট আপ করেছেন।

# Contribution

বলা বাহুল্য, BabyAGI এখনও তার শৈশবকালে রয়েছে এবং এইভাবে আমরা এখনও এর লক্ষ্য এবং সেখানে পৌছানোর পদক্ষেপগুলি নির্ধারণ করছি। বর্তমানে, BabyAGI-এর জন্য একটি মূল ডিজাইন লক্ষ্য হল _simple_ হওয়া যাতে এটি সহজে বোঝা এবং তৈরি করা যায়। এই সরলতা বজায় রাখার জন্য, আমরা অনুরোধ করছি যে আপনি PR জমা দেওয়ার সময় নিম্নলিখিত নির্দেশিকাগুলি মেনে চলুন:

- ব্যাপক রিফ্যাক্টরিংয়ের পরিবর্তে ছোট, মডুলার পরিবর্তনগুলিতে ফোকাস করুন।
- নতুন বৈশিষ্ট্যগুলি প্রবর্তন করার ক্ষেত্রে, আপনি যে নির্দিষ্ট ব্যবহারের ক্ষেত্রে সম্বোধন করছেন তার একটি বিশদ বিবরণ প্রদান করুন৷

@yoheinakajima'র মন্তব্য (Apr 5th, 2023):

> জানি এখন PR ক্রমবর্ধমান, আমি আপনার ধৈর্যের প্রশংসা করি - যেহেতু আমি GitHub এবং OpenSource-এ নতুন, এবং এই সপ্তাহে আমার সময় প্রাপ্যতার পরিকল্পনা করিনি। পুনঃনির্দেশ, আমি এটিকে সরল একইসাথে প্রসারিত রাখার বিষয়ে হিমশিম খাচ্ছি - বর্তমানে একটি মূল বেবি এজিআইকে সরল রাখার দিকে ঝুঁকছি, এবং এটিকে প্রসারিত করার জন্য বিভিন্ন পদ্ধতির সমর্থন ও প্রচার করার জন্য এটিকে একটি প্ল্যাটফর্ম হিসাবে ব্যবহার করছি (যেমন: BabyAGIxLangchain একটি উদাহরণ)। আমি বিশ্বাস করি যে বিভিন্ন মতামতযুক্ত পন্থা রয়েছে যা গবেষণার যোগ্য, এবং আমি তুলনা এবং আলোচনা করার জন্য একটি কেন্দ্রীয় স্থানের মূল্য বুঝি। আরো আপডেট শীঘ্রই আসছে।

আমি গিটহাব এবং ওপেন সোর্সে নতুন, তাই অনুগ্রহ করে ধৈর্য ধরুন কারণ আমি এই প্রকল্পটি সঠিকভাবে পরিচালনা করা শিখছি। আমি দিনে একটি ভিসি ফার্ম চালাই, তাই সাধারণত আমার বাচ্চারা ঘুমানোর পরে রাতে PR এবং সমস্যাগুলি পরীক্ষা করব - যা প্রতি রাতে নাও হতে পারে। যেকোনো ধরনের সহযোগিতার সমর্থন করছি, শীঘ্রই এই বিভাগটি আপডেট করা হবে (প্রত্যাশা, দৃষ্টিভঙ্গি, ইত্যাদি)। অনেক মানুষের সাথে কথা বলছি এবং শিখছি - আপডেটের জন্য অপেক্ষা করুন!

# BabyAGI কার্যকলাপ রিপোর্ট

BabyAGI কমিউনিটিকে প্রকল্পের অগ্রগতি সম্পর্কে অবগত থাকতে সাহায্য করার জন্য, Blueprint AI BabyAGI-এর জন্য একটি Github কার্যকলাপ সংক্ষিপ্তসার তৈরি করেছে৷ এই সংক্ষিপ্ত প্রতিবেদনটি গত 7 দিনে BabyAGI সংগ্রহস্থলে সমস্ত অবদানের সংক্ষিপ্তসার প্রদর্শন করে (প্রতিনিয়ত আপডেট করা হচ্ছে), এটি আপনার জন্য বেবি এজিআইএর সাম্প্রতিক বিকাশের উপর নজর রাখা সহজ করে তোলে। BabyAGI এর 7-দিনের কার্যকলাপ রিপোর্ট দেখতে, ক্লিক করুনঃ [https://app.blueprint.ai/github/yoheinakajima/babyagi](https://app.blueprint.ai/github/yoheinakajima/babyagi)

[<img width="293" alt="image" src="https://user-images.githubusercontent.com/334530/235789974-f49d3cbe-f4df-4c3d-89e9-bfb60eea6308.png">](https://app.blueprint.ai/github/yoheinakajima/babyagi)


# Inspired projects

BabyAGI প্রকাশের পর থেকে অল্প সময়ের মধ্যে অনেক প্রকল্পকে অনুপ্রাণিত করেছে। আপনি সবগুলো দেখতে পারেন [এখানে](docs/inspired-projects.md).

# Backstory

BabyAGI [Task-Driven Autonomous Agent] এর একটি সরলীকৃত ভার্সন (https://twitter.com/yoheinakajima/status/1640934493489070080?s=20) (Mar 28, 2023) টুইটারে শেয়ার করা হয়েছে। এই সংস্করণটি 140 লাইনের: 13টি মন্তব্য, 22টি শূন্যস্থান এবং 105 লাইন কোড। আসল স্বায়ত্তশাসিত এজেন্টের প্রতিক্রিয়ায় রেপোর নামটি এসেছে - লেখক বোঝাতে চাননা যে এটি AGI।

যত্ন ও ভালবাসা দিয়ে তৈরি করেছেন [@yoheinakajima](https://twitter.com/yoheinakajima), অনাকাঙ্ক্ষিতভাবে যিনি একজন ভিসি (আপনার সৃজনশীলতার বহিঃপ্রকাশ দেখতে আগ্রহী)