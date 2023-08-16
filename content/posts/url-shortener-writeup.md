---
title: "My URL Shortener (thr.fi) Project Writeup"
date: 2023-08-16T18:21:43+07:00
description: "A writeup of my URL Shortener project"
categories:
    - Project
tags:
    - Serverless
    - Cloudflare
---
![Landing](
    /img/thrfi-landing.png
)

This project idea came up when I needed something cool to display on my Discord profile that was simple enough to make. I think an URL shortener would look cool to display on my profile. Then, as I work on it, I read one of Cloudflare's official [tutorial projects](https://developers.cloudflare.com/workers/tutorials/build-a-qr-code-generator/) using Cloudflare Workers and decides to then a QR Code generator feature in the process.

# Overview
This project is a URL shortener with a QR Code generator feature. It is hosted on Cloudflare Workers and uses Cloudflare KV to store the data. The project is written in TypeScript and uses Webpack to bundle the code. With a privacy-first mindset, all data from the user is encrypted using AES-256-GCM or hashed using SHA256 before being stored in the KV. The project is also open-source and can be found on GitHub ([Frontend](
    https://github.com/reverseon/thrfi-fe
) and [Backend](
    https://github.com/reverseon/thrfi-be
)).

# Process
## Tech Stack

### Frontend
The frontend is a simple React app that uses vanilla CSS for the UI. It is bundled using Webpack and deployed to Cloudflare Workers using [Wrangler](
https://developers.cloudflare.com/workers/wrangler/
). I challenge myself to implement the frontend design I found on [Dribbble](
https://dribbble.com/shots/21462071-FlexFit-Web-Site-Design-Landing-Page-Home-Page-UI
).

### Backend
The backend is a Cloudflare Worker that uses Cloudflare KV to store the data. Why? because I want to challenge myself to build a completely serverless project and Cloudflare Workers is a familiar choice for me since I already use Cloudflare for my DNS.

## Flowchart
In a bird's view, this is the simplified process of the project, represented using boxes and arrows.

![Flowchart](/img/thrfi_flowchart.jpg)

There are four endpoints available in the backend:
- `/shorten` to shorten a URL
- `/qr` to generate a QR Code
- `/fetch` to get the original URL from the shortened URL
- `/unlock` to unlock a shortened URL (if it is password-protected)

I use `itty-router` to handle the routing in the backend. The backend also uses `Web Crypto API` to encrypt the data before storing it in the KV.

The general flow of the project is:
1. User enters a URL to be shortened
2. The frontend sends a request to the backend to shorten the URL
3. The backend generates a random string as the shortened URL and stores it in the KV (or if back-half is used, it will be stored in the KV instead)
4. The backend returns the shortened URL to the frontend
5. The frontend displays the shortened URL to the user
6. If the URL is password-protected, the user can enter the password to unlock the URL.

## Encryption
There are three user inputs that need to be stored:
- The original URL
- The back-half
- The password (if the URL is password-protected)

User back-half and password is hashed using SHA256 before being stored in the KV. The original URL on the other hand, can't be hashed because it needs to be decrypted later. So, I use AES-256-GCM to encrypt the original URL before storing it in the KV. I did this because, while technically I can decrypt the URL, in an event where the KV is compromised, the attacker will need to brute-force the encryption key to decrypt the URL.

## QR Code
The QR Code generator is a simple feature that uses [qr-image](
https://www.npmjs.com/package/qr-image
) to generate the QR Code. The QR Code is generated in the backend and then sent to the frontend as a base64 string. The frontend then displays the QR Code to the user.

# Challenges
## Cloudflare Workers
This is my first time using Cloudflare Workers and I have to say, it is a bit confusing at first. I have to learn how to use `wrangler` and how to deploy the project to Cloudflare Workers. I also have to learn how to use `Web Crypto API` to encrypt the data before storing it in the KV. I also have to learn how to use `itty-router` to handle the routing in the backend.

# Conclusion
This project is a fun project to work on. I learned a lot of new things and I also get to challenge myself to build a completely serverless project. I also get to learn how to use `Web Crypto API` and `itty-router`. I also get to learn how to use `wrangler` and how to deploy the project to Cloudflare Workers. I also get to learn how to use `qr-image` to generate a QR Code. Overall, this is a fun project to work on and I'm happy with the result.