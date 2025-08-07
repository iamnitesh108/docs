---
layout: default
title: Getting Started
description: Learn how to get started with our project
---

### 1 Define a Schema & Model 
```js
```js 
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, required: true, unique: true },
  age: Number,
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);

```
```
