// ==UserScript==
// @name         JeetDelete X.com - Hide Indian, Arabic, Japanese, and Thai Language Posts
// @namespace    http://tampermonkey.net/
// @version      1.4.3
// @description  Debugs and hides posts on X written in Hindi, other Indian languages, Arabic, Japanese, or Thai
// @author       xechostormx, hearing_echoes
// @match        https://x.com/*
// @match        https://www.x.com/*
// @match        https://*.x.com/* // Covers regional subdomains
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function() {
    'use strict';

    // Detect if text contains Indian scripts, Arabic, Japanese, or Thai
    function isTargetLanguage(text) {
        if (!text || typeof text !== 'string') return false;
        const ranges = [
            /[\u0900-\u097F]/, // Devanagari (Hindi, Marathi, etc.)
            /[\u0B80-\u0BFF]/, // Tamil
            /[\u0C00-\u0C7F]/, // Telugu
            /[\u0980-\u09FF]/, // Bengali
            /[\u0A80-\u0AFF]/, // Gujarati
            /[\u0B00-\u0B7F]/, // Odia
            /[\u0A00-\u0A7F]/, // Gurmukhi (Punjabi)
            /[\u0C80-\u0CFF]/, // Kannada
            /[\u0D00-\u0D7F]/, // Malayalam
            /[\u0600-\u06FF]/, // Arabic
            /[\u0750-\u077F]/, // Arabic Supplement
            /[\uFB50-\uFDFF]/, // Arabic Forms-A
            /[\uFE70-\uFEFF]/, // Arabic Forms-B
            /[\u3040-\u309F]/, // Hiragana
            /[\u30A0-\u30FF]/, // Katakana
            /[\u4E00-\u9FFF]/, // CJK Unified Ideographs (Kanji)
            /[\u0E00-\u0E7F]/  // Thai
        ];
        return ranges.some(rx => rx.test(text));
    }

    // Scan & hide matching tweets
    function hideTargetLanguagePosts() {
        const selector = 'article[data-testid="tweet"], div[data-testid="cellInnerDiv"]';
        const tweets = document.querySelectorAll(selector);
        console.log(`Scanning ${tweets.length} tweets`); // Debug log
        tweets.forEach((tweet) => {
            const txt = tweet.textContent || '';
            if (isTargetLanguage(txt)) {
                tweet.style.display = 'none'; // Or 'style.opacity = "0.3"' for fade
                console.log('Hid tweet:', txt.trim().slice(0, 50));
            }
        });
    }

    console.log('X-HideLang script initialized @', new Date().toLocaleTimeString());

    // Initial run after a short delay
    setTimeout(hideTargetLanguagePosts, 500);

    // Observe the feed for new tweets with debounce
    let debounceTimer;
    const root = document.querySelector('main') || document.body;
    if (root) {
        const observer = new MutationObserver(() => {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(hideTargetLanguagePosts, 200);
        });
        observer.observe(root, { childList: true, subtree: true });
        console.log('Observer attached to', root.tagName);
    } else {
        console.error('X-HideLang: couldn’t find a root element to observe');
    }

})();
