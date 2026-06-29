# Auto-Paste

ব্রাউজারের টেকনিক্যাল নিয়মানুযায়ী, একটি বুকমার্কলেট শুধুমাত্র বর্তমান পেজের ওপর কাজ করতে পারে। যখনই আপনি পেজ রিফ্রেশ করেন বা "Next" পেজে যান, ব্রাউজার আগের পেজের সব জাভাস্ক্রিপ্ট মেমোরি থেকে মুছে ফেলে। এই কারণে শুধু বুকমার্ক দিয়ে এটি রিফ্রেশ বা নেক্সট পেজে অটো-রান করানো সম্ভব নয়।

তবে আপনি ঠিক যেমনটা চাচ্ছেন—"একবার অন করলে পেজ রিফ্রেশ বা নেক্সট পেজে গেলেও অফ হবে না, যতক্ষণ না আপনি নিজে অফ করছেন"—সেটি করার একটি দারুণ সমাধান আছে। এজন্য আপনাকে ব্রাউজারে একটি ফ্রি এক্সটেনশন ব্যবহার করতে হবে, যার নাম Tampermonkey (ট্যাম্পারমাঙ্কি)।

এটি একটি ইউজার-স্ক্রিপ্ট ম্যানেজার। এর ভেতর নিচের কোডটি একবার সেভ করে রাখলে, এটি আপনার পছন্দের ওয়েবসাইটে স্থায়ীভাবে কাজ করবে।

যেভাবে এটি সেটআপ করবেন (খুবই সহজ):
১. প্রথমে আপনার ব্রাউজারের (Chrome/Edge/Firefox) এক্সটেনশন স্টোরে গিয়ে Tampermonkey লিখে সার্চ করে সেটি ইনস্টল করে নিন।
২. ইনস্টল হওয়ার পর ব্রাউজারের ওপরের ডানদিকের Tampermonkey আইকনে ক্লিক করে "Create a new script..." অপশনে যান।
৩. সেখানে আগে থেকে লেখা সব কোড মুছে দিয়ে নিচের কোডটি পুরোটা কপি করে পেস্ট করুন:
// ==UserScript==
// @name         Universal Auto Paste Mode (Small & Draggable)
// @namespace    http://tampermonkey.net/
// @version      3.0
// @description  Smaller, draggable auto paste button on any website
// @author       Gemini
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function() {
'use strict';
var btn = document.createElement('button');
btn.id = 'auto-paste-toggle-btn';
btn.style.position = 'fixed';

// বুকমার্কলেট: ছোট সাইজ
btn.style.top = '10px';
btn.style.right = '10px';
btn.style.zIndex = '1000000'; // অনেক বেশি, যেন সবকিছুর ওপর থাকে
btn.style.padding = '5px 10px'; // ছোট প্যাডিং
btn.style.borderRadius = '15px'; // ছোট রেডিয়াস
btn.style.border = 'none';
btn.style.color = '#fff';
btn.style.fontWeight = 'bold';
btn.style.cursor = 'grab'; // ড্র্যাগ করার জন্য হ্যান্ড আইকন
btn.style.boxShadow = '0 2px 4px rgba(0,0,0,0.3)';
btn.style.fontFamily = 'Arial, sans-serif';
btn.style.fontSize = '11px'; // ছোট ফন্ট সাইজ
btn.style.opacity = '0.7'; // একটু স্বচ্ছ, যেন পেজের লেখা দেখা যায়

var isEnabled = localStorage.getItem('autoPasteEnabled') === 'true';

function updateUI() {
if (isEnabled) {
btn.innerText = 'Paste: ON';
btn.style.backgroundColor = '#28a745'; // সবুজ
} else {
btn.innerText = 'Paste: OFF';
btn.style.backgroundColor = '#dc3545'; // লাল
}
}

updateUI();
document.body.appendChild(btn);

// ড্র্যাগেবল করার জন্য ফাংশন
var isDragging = false;
var offsetX, offsetY;

btn.addEventListener('mousedown', function(e) {
isDragging = true;
offsetX = e.clientX - btn.getBoundingClientRect().left;
offsetY = e.clientY - btn.getBoundingClientRect().top;
btn.style.cursor = 'grabbing';
btn.style.opacity = '1'; // ড্র্যাগ করার সময় পুরোপুরি দেখা যাবে
btn.style.transition = 'none'; // ড্র্যাগিং স্মুথ করার জন্য ট্রানজিশন বন্ধ
});

document.addEventListener('mousemove', function(e) {
if (isDragging) {
btn.style.left = (e.clientX - offsetX) + 'px';
btn.style.top = (e.clientY - offsetY) + 'px';
btn.style.right = 'auto'; // আগের রাইট পজিশন বাতিল
}
});

document.addEventListener('mouseup', function() {
if (isDragging) {
isDragging = false;
btn.style.cursor = 'grab';
btn.style.opacity = '0.7'; // ড্র্যাগ শেষ হলে আবার স্বচ্ছ হয়ে যাবে
btn.style.transition = 'opacity 0.3s'; // ট্রানজিশন আবার চালু

// নতুন পজিশন সেভ করে রাখা যেন রিফ্রেশ দিলেও সেখানেই থাকে
localStorage.setItem('autoPasteBtnLeft', btn.style.left);
localStorage.setItem('autoPasteBtnTop', btn.style.top);
}
});

// পেজ লোড হলে আগের সেভ করা পজিশন সেট করা
window.addEventListener('load', function() {
var savedLeft = localStorage.getItem('autoPasteBtnLeft');
var savedTop = localStorage.getItem('autoPasteBtnTop');
if (savedLeft && savedTop) {
btn.style.left = savedLeft;
btn.style.top = savedTop;
btn.style.right = 'auto';
}
});

// অটো-পেস্টিং লজিক
var autoPasteHandler = async function(e) {
if (!isEnabled || isDragging) return; // ড্র্যাগ করার সময় যেন পেস্ট না হয়
if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') {
try {
// Clipboard API শুধুমাত্র সুরক্ষিত কন্টেনস্টে কাজ করে (HTTPS)
const text = await navigator.clipboard.readText();
if (text) {
e.target.value = text;
e.target.dispatchEvent(new Event('input', { bubbles: true }));
e.target.dispatchEvent(new Event('change', { bubbles: true }));
}
} catch (err) {
// Clipboard API ব্যবহারের জন্য প্রথমবার অনুমতি চাইতে পারে
console.error('Failed to read from clipboard:', err);
// বিকল্প পদ্ধতি
// document.execCommand('paste'); // এটি সবসময় সব ব্রাউজারে কাজ নাও করতে পারে
}
}
};

// ক্লিক ইভেন্ট লিসেনার (ক্যাপচারিং মোড, যেন ইনপুট ফিল্ডের ওপরের লেয়ার হিসেবে কাজ করে)
document.addEventListener('click', autoPasteHandler, true);

// বাটনে ক্লিক করলে অন/অফ টগল হওয়া
btn.addEventListener('click', function(e) {
if (isDragging) return; // ড্র্যাগ করার সময় ক্লিক ইভেন্ট বন্ধ
e.stopPropagation(); // বাটনে ক্লিক করলে যেন পেস্ট না হয়
isEnabled = !isEnabled;
localStorage.setItem('autoPasteEnabled', isEnabled);
updateUI();
});
})();
৪. কোড পেস্ট করার পর কীবোর্ডের Ctrl + S চেপে অথবা Tampermonkey-এর File -> Save মেনুতে গিয়ে এটি সেভ করুন।

এটি এখন যেভাবে কাজ করবে:
কোডটি আপনার বুকমার্কলেটে ব্যবহার করতে পারবেন। ড্র্যাগ করার জন্য হ্যান্ড আইকন দেওয়া হয়েছে এবং পেজের ওপর একটি স্বচ্ছ আস্তরণ তৈরি করা হয়েছে যেন এটি পেজের লেখার ওপরে থাকে এবং তা সহজে সরিয়ে রাখা যায়। বাটনটির নতুন পজিশন ব্রাউজারে সেভ করে রাখা হয়, যেন পেজ রিফ্রেশ দিলেও এটি আপনার সরিয়ে রাখা পজিশনেই থাকে।
রিফ্রেশ করলেও মেমোরি থাকবে: আপনি যদি বাটনটিতে ক্লিক করে Auto-Paste: ON (সবুজ) করে দেন, তবে পেজ রিফ্রেশ করলে, পরের পেজে গেলে কিংবা কম্পিউটার বন্ধ করে পরদিন আসলেও এটি সবুজ (ON) অবস্থাতেই থাকবে। ব্রাউজার মনে রাখবে যে আপনি এটি অন করে রেখেছিলেন।

বন্ধ করার নিয়ম: আপনার কাজ পুরোপুরি শেষ হয়ে গেলে জাস্ট ওই বাটনে আরেকবার ক্লিক করবেন, সেটি Auto-Paste: OFF (লাল) হয়ে যাবে এবং অটো-পেস্ট বন্ধ হয়ে যাবে।
