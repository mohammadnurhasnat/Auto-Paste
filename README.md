# Auto-Paste

ব্রাউজারের টেকনিক্যাল নিয়মানুযায়ী, একটি বুকমার্কলেট শুধুমাত্র বর্তমান পেজের ওপর কাজ করতে পারে। যখনই আপনি পেজ রিফ্রেশ করেন বা "Next" পেজে যান, ব্রাউজার আগের পেজের সব জাভাস্ক্রিপ্ট মেমোরি থেকে মুছে ফেলে। এই কারণে শুধু বুকমার্ক দিয়ে এটি রিফ্রেশ বা নেক্সট পেজে অটো-রান করানো সম্ভব নয়।

তবে আপনি ঠিক যেমনটা চাচ্ছেন—"একবার অন করলে পেজ রিফ্রেশ বা নেক্সট পেজে গেলেও অফ হবে না, যতক্ষণ না আপনি নিজে অফ করছেন"—সেটি করার একটি দারুণ সমাধান আছে। এজন্য আপনাকে ব্রাউজারে একটি ফ্রি এক্সটেনশন ব্যবহার করতে হবে, যার নাম Tampermonkey (ট্যাম্পারমাঙ্কি)।

এটি একটি ইউজার-স্ক্রিপ্ট ম্যানেজার। এর ভেতর নিচের কোডটি একবার সেভ করে রাখলে, এটি আপনার পছন্দের ওয়েবসাইটে স্থায়ীভাবে কাজ করবে।

যেভাবে এটি সেটআপ করবেন (খুবই সহজ):
১. প্রথমে আপনার ব্রাউজারের (Chrome/Edge/Firefox) এক্সটেনশন স্টোরে গিয়ে Tampermonkey লিখে সার্চ করে সেটি ইনস্টল করে নিন।
২. ইনস্টল হওয়ার পর ব্রাউজারের ওপরের ডানদিকের Tampermonkey আইকনে ক্লিক করে "Create a new script..." অপশনে যান।
৩. সেখানে আগে থেকে লেখা সব কোড মুছে দিয়ে নিচের কোডটি পুরোটা কপি করে পেস্ট করুন:
// ==UserScript==
// @name         Visa Auto-Paste Persistent Mode
// @namespace    http://tampermonkey.net/
// @version      1.2
// @description  Persistently auto-paste text on click across page refreshes
// @author       Gemini
// @match        *://*.indianvisaonline.gov.in/*
// @grant        none
// ==UserScript==

(function() {
    'use strict';

    // ১. স্ক্রিনে একটি ফ্লোটিং বাটন তৈরি করা
    var btn = document.createElement('button');
    btn.id = 'auto-paste-toggle-btn';
    btn.style.position = 'fixed';
    btn.style.top = '20px';
    btn.style.right = '20px';
    btn.style.zIndex = '100000';
    btn.style.padding = '8px 15px';
    btn.style.borderRadius = '20px';
    btn.style.border = 'none';
    btn.style.color = '#fff';
    btn.style.fontWeight = 'bold';
    btn.style.cursor = 'pointer';
    btn.style.boxShadow = '0 4px 6px rgba(0,0,0,0.2)';
    btn.style.fontFamily = 'Arial, sans-serif';
    btn.style.fontSize = '13px';

    // ২. আগের সেভ করা স্টেট (On/Off) চেক করা
    var isEnabled = localStorage.getItem('autoPasteEnabled') === 'true';

    // ৩. বাটনের কালার ও টেক্সট আপডেট করার ফাংশন
    function updateUI() {
        if (isEnabled) {
            btn.innerText = 'Auto-Paste: ON';
            btn.style.backgroundColor = '#28a745'; // সবুজ কালার
        } else {
            btn.innerText = 'Auto-Paste: OFF';
            btn.style.backgroundColor = '#dc3545'; // লাল কালার
        }
    }

    updateUI();
    document.body.appendChild(btn);

    // ৪. অটো-পেস্টিং লজিক
    var autoPasteHandler = async function(e) {
        if (!isEnabled) return;
        if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') {
            try {
                const text = await navigator.clipboard.readText();
                e.target.value = text;
                e.target.dispatchEvent(new Event('input', { bubbles: true }));
                e.target.dispatchEvent(new Event('change', { bubbles: true }));
            } catch (err) {
                console.error('Clipboard error:', err);
            }
        }
    };

    // ক্লিক ইভেন্ট লিসেনার (ক্যাপচারিং মোড)
    document.addEventListener('click', autoPasteHandler, true);

    // ৫. বাটনে ক্লিক করলে অন/অফ টগল হওয়া
    btn.addEventListener('click', function(e) {
        e.stopPropagation(); // বাটনে ক্লিক করলে যেন পেস্ট না হয়
        isEnabled = !isEnabled;
        localStorage.setItem('autoPasteEnabled', isEnabled); // ব্রাউজার মেমোরিতে সেভ রাখা
        updateUI();
    });
})();

৪. কোড পেস্ট করার পর কীবোর্ডের Ctrl + S চেপে অথবা Tampermonkey-এর File -> Save মেনুতে গিয়ে এটি সেভ করুন।

এটি এখন যেভাবে কাজ করবে:
স্থায়ী সবুজ/লাল বাটন: এখন থেকে আপনি যখনই ইন্ডিয়ান ভিসার ওয়েবসাইটে (indianvisaonline.gov.in) ঢুকবেন, স্ক্রিনের ডানদিকের ওপরের কোণায় একটি বাটন দেখতে পাবেন।

রিফ্রেশ করলেও মেমোরি থাকবে: আপনি যদি বাটনটিতে ক্লিক করে Auto-Paste: ON (সবুজ) করে দেন, তবে পেজ রিফ্রেশ করলে, পরের পেজে গেলে কিংবা কম্পিউটার বন্ধ করে পরদিন আসলেও এটি সবুজ (ON) অবস্থাতেই থাকবে। ব্রাউজার মনে রাখবে যে আপনি এটি অন করে রেখেছিলেন।

বন্ধ করার নিয়ম: আপনার কাজ পুরোপুরি শেষ হয়ে গেলে জাস্ট ওই বাটনে আরেকবার ক্লিক করবেন, সেটি Auto-Paste: OFF (লাল) হয়ে যাবে এবং অটো-পেস্ট বন্ধ হয়ে যাবে।

(নোট: কোডের ৪ নম্বর লাইনে @match শুধু ইন্ডিয়ান ভিসা ওয়েবসাইটের জন্য সেট করা আছে। আপনি যদি চান এটি পৃথিবীর সব ওয়েবসাইটে কাজ করুক, তবে ওই লাইনটি পরিবর্তন করে // @match *://*/* লিখে সেভ করলেই হবে।)
