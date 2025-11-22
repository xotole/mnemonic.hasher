# mnemonic.locker

# Password-Based Seed Shuffle

Deterministic shuffle of a seed list using a password → SHA-256 → Fisher–Yates process.

---

## 📌 Overview

This algorithm takes a user-provided password and uses its SHA-256 hash bytes to deterministically shuffle a list of seeds.

* Same password → same shuffle.
* Different password → different shuffle.

Mnemonic: **Password → Hash → Shuffle → Table**

---

# 🚀 Steps

## **1. Read Password (with validation)**

```js
password = document.getElementById("password").value;

if (!password || password.length > 10) {
    alert("Password must be max of 10 char.");
    return;
}
```

---

## **2. Hash Password Using SHA-256**

```js
const encoder = new TextEncoder();
const data = encoder.encode(password);
const hashBuffer = await crypto.subtle.digest("SHA-256", data);
const hashArray = Array.from(new Uint8Array(hashBuffer)); // 32 bytes
```

---

## **3. Shuffle Seeds Using Modified Fisher–Yates**

* Each iteration uses one byte of the hash.
* `i % hashArray.length` cycles through hash bytes.
* `hashByte % (i + 1)` produces the deterministic swap index.

```js
let shuffledSeeds = [...seeds];

for (let i = shuffledSeeds.length - 1; i > 0; i--) {
    const hashByte = hashArray[i % hashArray.length];
    const j = hashByte % (i + 1);
    [shuffledSeeds[i], shuffledSeeds[j]] = [shuffledSeeds[j], shuffledSeeds[i]];
}
```

---

## **4. Render the Shuffled Seeds in a Table (10 per row)**

```js
const table = document
  .getElementById("seedsTable")
  .getElementsByTagName("tbody")[0];

table.innerHTML = "";

for (let i = 0; i < shuffledSeeds.length; i += 10) {
    let row = "<tr>";
    for (let j = 0; j < 10; j++) {
        if (i + j < shuffledSeeds.length) {
            const label = generateLabel(i + j);
            row += `<td>${label}</td><td>${shuffledSeeds[i + j]}</td>`;
        }
    }
    row += "</tr>";
    table.innerHTML += row;
}
```

---

# 📄 Minimal Reusable Function

```js
async function shuffleWithPassword(password, seeds) {
  const hash = new Uint8Array(
    await crypto.subtle.digest("SHA-256", new TextEncoder().encode(password))
  );

  const out = [...seeds];
  for (let i = out.length - 1; i > 0; i--) {
    const j = hash[i % hash.length] % (i + 1);
    [out[i], out[j]] = [out[j], out[i]];
  }
  return out;
}
```
