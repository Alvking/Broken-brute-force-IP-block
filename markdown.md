# Broken Brute-Force Protection, IP Block

This repository documents the step-by-step walkthrough for solving PortSwigger Web Security Academy's lab on broken brute-force protection with IP blocking.

## Overview

The vulnerability occurs because the failed-login counter is reset after a valid login succeeds before the rate-limit threshold is reached. This allows an attacker to alternate between a valid account and the target account in order to evade the IP block and brute-force the password.

The goal is to use Burp Suite Intruder to send login attempts in a pattern that keeps the IP block from triggering while still checking candidate passwords for the target user.

---

## Step 1: Initial reconnaissance and intercepting the login request

1. Open the lab in the browser and attempt a failed login.
2. In Burp Suite, go to the Proxy tab and inspect the HTTP history.
3. Find the `POST /login` request and send it to Burp Intruder.

<img src="Screenshot_2026-09-21_21-21-48.png" alt="Initial POST /login request in Burp Proxy" width="900" />

---

## Step 2: Configure the Burp Intruder attack

1. Send the `POST /login` request to Burp Intruder.
2. Change the attack type to `Pitchfork`.
3. Set the payload positions so that:
   - `username=§wiener§`
   - `password=§peter§`

<img src="Screenshot_2026-09-21_21-20-15.png" alt="Configuring Burp Intruder with Pitchfork and payload positions" width="900" />

---

## Step 3: Configure the payload lists

Go to the Payloads tab and configure both positions.

### Username payload list
- Use a simple list.
- Start with the valid account username, `wiener`.
- Then alternate with the target username, `carlos`, repeated many times.
- Ensure the valid user is sent first so the failed attempt counter resets without triggering the block.

### Password payload list
- Use a simple list.
- Place the correct password for `wiener` (`peter`) before each candidate password for `carlos`.
- This pattern keeps the valid login sequence aligned with the target login attempts.

---

## Step 4: Run the attack and identify the password

1. Click `Start attack`.
2. Review the results and look for a response indicating a successful login for `carlos`.
3. The successful request will usually return a `302` redirect or have a noticeably different response length.
4. Note the password from the corresponding payload value.

<img src="Screenshot_2026-09-21_21-23-52.png" alt="Burp Intruder attack results showing the valid password for carlos" width="900" />

---

## Step 5: Log in and solve the lab

1. Return to the browser and log in as `carlos` using the recovered password.
2. Access Carlos's account page.

<img src="Screenshot_2026-09-21_21-25-45.png" alt="Logged in as Carlos in the target account" width="900" />

3. The lab is solved once the account page loads successfully.

<img src="Screenshot_2026-09-21_21-24-27.png" alt="Lab solved after logging in as Carlos" width="900" />

---

## Result

By alternating a valid login with attempts against the target account, the failed-attempt counter resets before the IP block is triggered. This bypasses the brute-force protection and exposes the password for the target user.
