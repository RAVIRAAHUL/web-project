# Subscription Management Backend (Node.js + Express)

This backend handles **user and subscription management**. It follows the MVC architecture (Models, Controllers, Routers) for clean separation of concerns.

---

## 📂 Folder Structure
```
nodeapp/
│
├── controllers/
│   ├── userController.js
│   └── subscriptionController.js
│
├── models/
│   ├── userModel.js
│   └── subscriptionModel.js
│
├── routers/
│   ├── userRouter.js
│   └── subscriptionRouter.js
│
├── authUtils.js
├── index.js
├── package.json
└── package-lock.json
```

---

## 🧠 index.js
```js
import express from 'express';
import mongoose from 'mongoose';
import bodyParser from 'body-parser';
import cors from 'cors';

import userRouter from './routers/userRouter.js';
import subscriptionRouter from './routers/subscriptionRouter.js';

const app = express();

app.use(cors());
app.use(bodyParser.json());

mongoose.connect('mongodb://localhost:27017/appdb', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => console.log('✅ MongoDB connected'))
  .catch(err => console.error('❌ DB Connection Failed:', err));

app.use('/api/users', userRouter);
app.use('/api/subscriptions', subscriptionRouter);

app.get('/', (req, res) => {
  res.send('Welcome to Subscription Management API');
});

const PORT = 5000;
app.listen(PORT, () => console.log(`🚀 Server running on http://localhost:${PORT}`));
```

---

## 👤 userModel.js
```js
import mongoose from 'mongoose';

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
}, { timestamps: true });

export default mongoose.model('User', userSchema);
```

---

## 💼 subscriptionModel.js
```js
import mongoose from 'mongoose';

const subscriptionSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  plan: { type: String, required: true },
  startDate: { type: Date, default: Date.now },
  endDate: { type: Date },
}, { timestamps: true });

export default mongoose.model('Subscription', subscriptionSchema);
```

---

## ⚙️ userController.js
```js
import User from '../models/userModel.js';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';

export const registerUser = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    const existingUser = await User.findOne({ email });
    if (existingUser) return res.status(400).json({ message: 'User already exists' });

    const hashedPassword = await bcrypt.hash(password, 10);
    const newUser = await User.create({ name, email, password: hashedPassword });

    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const loginUser = async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (!user) return res.status(404).json({ message: 'User not found' });

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json({ message: 'Invalid credentials' });

    const token = jwt.sign({ id: user._id }, 'secretkey', { expiresIn: '1d' });

    res.json({ message: 'Login successful', token });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

---

## 💳 subscriptionController.js
```js
import Subscription from '../models/subscriptionModel.js';

export const createSubscription = async (req, res) => {
  try {
    const { userId, plan, endDate } = req.body;
    const subscription = await Subscription.create({ userId, plan, endDate });
    res.status(201).json(subscription);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getSubscriptions = async (req, res) => {
  try {
    const subscriptions = await Subscription.find().populate('userId');
    res.json(subscriptions);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

---

## 🧭 userRouter.js
```js
import express from 'express';
import { registerUser, loginUser } from '../controllers/userController.js';

const router = express.Router();

router.post('/register', registerUser);
router.post('/login', loginUser);

export default router;
```

---

## 📦 subscriptionRouter.js
```js
import express from 'express';
import { createSubscription, getSubscriptions } from '../controllers/subscriptionController.js';

const router = express.Router();

router.post('/', createSubscription);
router.get('/', getSubscriptions);

export default router;
```

---

## 🔐 authUtils.js
```js
import jwt from 'jsonwebtoken';

export const verifyToken = (req, res, next) => {
  const token = req.headers['authorization'];
  if (!token) return res.status(403).json({ message: 'No token provided' });

  jwt.verify(token, 'secretkey', (err, decoded) => {
    if (err) return res.status(401).json({ message: 'Unauthorized' });
    req.userId = decoded.id;
    next();
  });
};
```

---

✅ **You can copy-paste this entire backend structure directly into VS Code.** 
This will give you a working Node.js + Express + MongoDB backend for user and subscription management.
