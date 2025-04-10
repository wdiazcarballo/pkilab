# บทช่วยสอนการสร้างแอปพลิเคชัน Flip-Card ด้วย MERN Stack

## บทนำ

บทช่วยสอนนี้จะแนะนำวิธีการสร้างแอปพลิเคชัน Flip-Card แบบเต็มรูปแบบโดยใช้ MERN Stack ซึ่งประกอบด้วย:

- **M**ongoDB - ฐานข้อมูล NoSQL
- **E**xpress - เฟรมเวิร์คสำหรับ Node.js
- **R**eact - ไลบรารี JavaScript สำหรับสร้าง UI
- **N**ode.js - สภาพแวดล้อมการทำงานของ JavaScript

แอปพลิเคชันของเราจะมีสองโหมด:
1. **โหมดสร้าง (Creator Mode)** - ผู้ใช้สามารถสร้างหมวดหมู่และเพิ่มข้อมูลการ์ดผ่าน RESTful API
2. **โหมดเล่น (Player Mode)** - ผู้ใช้สามารถเลือกหมวดหมู่และเรียนรู้ผ่านการพลิกการ์ด

## สารบัญ

1. [การตั้งค่าโครงการ](#1-การตั้งค่าโครงการ)
2. [การสร้างฐานข้อมูล MongoDB](#2-การสร้างฐานข้อมูล-mongodb)
3. [การพัฒนา Backend ด้วย Express และ Node.js](#3-การพัฒนา-backend-ด้วย-express-และ-nodejs)
4. [การพัฒนา Frontend ด้วย React](#4-การพัฒนา-frontend-ด้วย-react)
5. [การเชื่อมต่อ Frontend และ Backend](#5-การเชื่อมต่อ-frontend-และ-backend)
6. [การปรับแต่งและทำให้เสร็จสมบูรณ์](#6-การปรับแต่งและทำให้เสร็จสมบูรณ์)
7. [การเผยแพร่แอปพลิเคชัน](#7-การเผยแพร่แอปพลิเคชัน)

---

## 1. การตั้งค่าโครงการ

### 1.1 ติดตั้ง Node.js และ npm

1. ดาวน์โหลดและติดตั้ง Node.js จาก [nodejs.org](https://nodejs.org/)
2. ตรวจสอบการติดตั้งโดยเปิด Terminal หรือ Command Prompt และพิมพ์:

```bash
node -v
npm -v
```

### 1.2 ตั้งค่าโครงสร้างโปรเจค

1. สร้างโฟลเดอร์หลักสำหรับโครงการ:

```bash
mkdir flip-card-app
cd flip-card-app
```

2. จัดโครงสร้างโปรเจคให้แยกส่วน frontend และ backend:

```bash
mkdir backend frontend
```

### 1.3 ตั้งค่า Backend

1. เข้าไปที่โฟลเดอร์ backend:

```bash
cd backend
```

2. สร้างไฟล์ package.json:

```bash
npm init -y
```

3. ติดตั้งแพ็คเกจที่จำเป็น:

```bash
npm install express mongoose cors dotenv body-parser jsonwebtoken bcrypt
npm install nodemon --save-dev
```

4. แก้ไขไฟล์ package.json เพื่อเพิ่ม script สำหรับการพัฒนา:

```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
}
```

### 1.4 ตั้งค่า Frontend

1. กลับไปที่โฟลเดอร์หลักและสร้างโปรเจค React:

```bash
cd ..
cd frontend
npx create-react-app .
```

2. ติดตั้งแพ็คเกจเพิ่มเติมสำหรับ Frontend:

```bash
npm install axios react-router-dom styled-components framer-motion
```

---

## 2. การสร้างฐานข้อมูล MongoDB

### 2.1 สมัครบัญชี MongoDB Atlas

1. ไปที่ [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register) และสร้างบัญชีผู้ใช้
2. สร้าง Cluster ใหม่ (เลือกแผนฟรี)
3. ตั้งค่าการเข้าถึงฐานข้อมูล (Database Access)
   - สร้างผู้ใช้ฐานข้อมูลและรหัสผ่าน
4. ตั้งค่าการเข้าถึงเครือข่าย (Network Access)
   - อนุญาตการเข้าถึงจากทุก IP (0.0.0.0/0) สำหรับการพัฒนา

### 2.2 รับ Connection String

1. จาก Dashboard คลิกที่ "Connect"
2. เลือก "Connect your application"
3. คัดลอก Connection String
4. แทนที่ `<password>` ด้วยรหัสผ่านผู้ใช้ฐานข้อมูลที่สร้างไว้

### 2.3 กำหนดค่าการเชื่อมต่อใน Backend

1. สร้างไฟล์ `.env` ในโฟลเดอร์ backend:

```
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/flipcarddb?retryWrites=true&w=majority
PORT=5000
JWT_SECRET=your_jwt_secret_key
```

---

## 3. การพัฒนา Backend ด้วย Express และ Node.js

### 3.1 สร้างโครงสร้างโฟลเดอร์สำหรับ Backend

```bash
cd backend
mkdir controllers models routes middleware config
```

### 3.2 สร้างไฟล์การเชื่อมต่อฐานข้อมูล

สร้างไฟล์ `config/db.js`:

```javascript
const mongoose = require('mongoose');
require('dotenv').config();

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('MongoDB connected successfully');
  } catch (error) {
    console.error('MongoDB connection error:', error.message);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 3.3 สร้างโมเดลข้อมูล

สร้างไฟล์ `models/User.js`:

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const UserSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    trim: true
  },
  password: {
    type: String,
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Hash password before saving
UserSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Method to compare passwords
UserSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

module.exports = mongoose.model('User', UserSchema);
```

สร้างไฟล์ `models/Category.js`:

```javascript
const mongoose = require('mongoose');

const CategorySchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  description: {
    type: String,
    trim: true
  },
  creator: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Category', CategorySchema);
```

สร้างไฟล์ `models/Card.js`:

```javascript
const mongoose = require('mongoose');

const CardSchema = new mongoose.Schema({
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category',
    required: true
  },
  front: {
    type: String,
    required: true
  },
  back: {
    type: [String],
    required: true
  },
  creator: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Card', CardSchema);
```

### 3.4 สร้าง Middleware สำหรับการยืนยันตัวตน

สร้างไฟล์ `middleware/auth.js`:

```javascript
const jwt = require('jsonwebtoken');
require('dotenv').config();

module.exports = (req, res, next) => {
  // Get token from header
  const token = req.header('x-auth-token');

  // Check if no token
  if (!token) {
    return res.status(401).json({ msg: 'ไม่มีโทเค็น การเข้าถึงถูกปฏิเสธ' });
  }

  // Verify token
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded.user;
    next();
  } catch (err) {
    res.status(401).json({ msg: 'โทเค็นไม่ถูกต้อง' });
  }
};
```

### 3.5 สร้าง Controller

สร้างไฟล์ `controllers/userController.js`:

```javascript
const User = require('../models/User');
const jwt = require('jsonwebtoken');
require('dotenv').config();

// Register User
exports.registerUser = async (req, res) => {
  const { username, email, password } = req.body;

  try {
    // Check if user already exists
    let user = await User.findOne({ email });
    if (user) {
      return res.status(400).json({ msg: 'อีเมลนี้มีผู้ใช้งานแล้ว' });
    }

    // Create new user
    user = new User({
      username,
      email,
      password
    });

    await user.save();

    // Create and return JWT
    const payload = {
      user: {
        id: user.id
      }
    };

    jwt.sign(
      payload,
      process.env.JWT_SECRET,
      { expiresIn: '1h' },
      (err, token) => {
        if (err) throw err;
        res.json({ token });
      }
    );
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Login User
exports.loginUser = async (req, res) => {
  const { email, password } = req.body;

  try {
    // Find user by email
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(400).json({ msg: 'ข้อมูลไม่ถูกต้อง' });
    }

    // Check password
    const isMatch = await user.comparePassword(password);
    if (!isMatch) {
      return res.status(400).json({ msg: 'ข้อมูลไม่ถูกต้อง' });
    }

    // Create and return JWT
    const payload = {
      user: {
        id: user.id
      }
    };

    jwt.sign(
      payload,
      process.env.JWT_SECRET,
      { expiresIn: '1h' },
      (err, token) => {
        if (err) throw err;
        res.json({ token });
      }
    );
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Get User Profile
exports.getProfile = async (req, res) => {
  try {
    const user = await User.findById(req.user.id).select('-password');
    res.json(user);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};
```

สร้างไฟล์ `controllers/categoryController.js`:

```javascript
const Category = require('../models/Category');

// Create category
exports.createCategory = async (req, res) => {
  const { name, description } = req.body;

  try {
    const newCategory = new Category({
      name,
      description,
      creator: req.user.id
    });

    const category = await newCategory.save();
    res.json(category);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Get all categories
exports.getAllCategories = async (req, res) => {
  try {
    const categories = await Category.find().sort({ createdAt: -1 });
    res.json(categories);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Get user categories
exports.getUserCategories = async (req, res) => {
  try {
    const categories = await Category.find({ creator: req.user.id }).sort({ createdAt: -1 });
    res.json(categories);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Get category by ID
exports.getCategoryById = async (req, res) => {
  try {
    const category = await Category.findById(req.params.id);
    
    if (!category) {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }

    res.json(category);
  } catch (err) {
    console.error(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Update category
exports.updateCategory = async (req, res) => {
  const { name, description } = req.body;

  try {
    let category = await Category.findById(req.params.id);
    
    if (!category) {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }

    // Check user
    if (category.creator.toString() !== req.user.id) {
      return res.status(401).json({ msg: 'ไม่มีสิทธิ์แก้ไข' });
    }

    // Update
    category.name = name;
    category.description = description;

    await category.save();
    res.json(category);
  } catch (err) {
    console.error(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Delete category
exports.deleteCategory = async (req, res) => {
  try {
    const category = await Category.findById(req.params.id);
    
    if (!category) {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }

    // Check user
    if (category.creator.toString() !== req.user.id) {
      return res.status(401).json({ msg: 'ไม่มีสิทธิ์ลบ' });
    }

    await category.remove();
    res.json({ msg: 'ลบหมวดหมู่เรียบร้อยแล้ว' });
  } catch (err) {
    console.error(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};
```

สร้างไฟล์ `controllers/cardController.js`:

```javascript
const Card = require('../models/Card');
const Category = require('../models/Category');

// Create card
exports.createCard = async (req, res) => {
  const { category, front, back } = req.body;

  try {
    // Check if category exists
    const categoryExists = await Category.findById(category);
    if (!categoryExists) {
      return res.status(404).json({ msg: 'ไม่พบหมวดหมู่' });
    }

    const newCard = new Card({
      category,
      front,
      back: Array.isArray(back) ? back : [back],
      creator: req.user.id
    });

    const card = await newCard.save();
    res.json(card);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Get cards by category
exports.getCardsByCategory = async (req, res) => {
  try {
    const cards = await Card.find({ category: req.params.categoryId }).sort({ createdAt: -1 });
    res.json(cards);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Get card by ID
exports.getCardById = async (req, res) => {
  try {
    const card = await Card.findById(req.params.id);
    
    if (!card) {
      return res.status(404).json({ msg: 'ไม่พบการ์ด' });
    }

    res.json(card);
  } catch (err) {
    console.error(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'ไม่พบการ์ด' });
    }
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Update card
exports.updateCard = async (req, res) => {
  const { front, back } = req.body;

  try {
    let card = await Card.findById(req.params.id);
    
    if (!card) {
      return res.status(404).json({ msg: 'ไม่พบการ์ด' });
    }

    // Check user
    if (card.creator.toString() !== req.user.id) {
      return res.status(401).json({ msg: 'ไม่มีสิทธิ์แก้ไข' });
    }

    // Update
    card.front = front;
    card.back = Array.isArray(back) ? back : [back];

    await card.save();
    res.json(card);
  } catch (err) {
    console.error(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'ไม่พบการ์ด' });
    }
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};

// Delete card
exports.deleteCard = async (req, res) => {
  try {
    const card = await Card.findById(req.params.id);
    
    if (!card) {
      return res.status(404).json({ msg: 'ไม่พบการ์ด' });
    }

    // Check user
    if (card.creator.toString() !== req.user.id) {
      return res.status(401).json({ msg: 'ไม่มีสิทธิ์ลบ' });
    }

    await card.remove();
    res.json({ msg: 'ลบการ์ดเรียบร้อยแล้ว' });
  } catch (err) {
    console.error(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'ไม่พบการ์ด' });
    }
    res.status(500).send('เซิร์ฟเวอร์ผิดพลาด');
  }
};
```

### 3.6 สร้าง Routes

สร้างไฟล์ `routes/userRoutes.js`:

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const auth = require('../middleware/auth');

// Register User - POST /api/users/register
router.post('/register', userController.registerUser);

// Login User - POST /api/users/login
router.post('/login', userController.loginUser);

// Get User Profile - GET /api/users/profile
router.get('/profile', auth, userController.getProfile);

module.exports = router;
```

สร้างไฟล์ `routes/categoryRoutes.js`:

```javascript
const express = require('express');
const router = express.Router();
const categoryController = require('../controllers/categoryController');
const auth = require('../middleware/auth');

// Create category - POST /api/categories
router.post('/', auth, categoryController.createCategory);

// Get all categories - GET /api/categories
router.get('/', categoryController.getAllCategories);

// Get user categories - GET /api/categories/user
router.get('/user', auth, categoryController.getUserCategories);

// Get category by ID - GET /api/categories/:id
router.get('/:id', categoryController.getCategoryById);

// Update category - PUT /api/categories/:id
router.put('/:id', auth, categoryController.updateCategory);

// Delete category - DELETE /api/categories/:id
router.delete('/:id', auth, categoryController.deleteCategory);

module.exports = router;
```

สร้างไฟล์ `routes/cardRoutes.js`:

```javascript
const express = require('express');
const router = express.Router();
const cardController = require('../controllers/cardController');
const auth = require('../middleware/auth');

// Create card - POST /api/cards
router.post('/', auth, cardController.createCard);

// Get cards by category - GET /api/cards/category/:categoryId
router.get('/category/:categoryId', cardController.getCardsByCategory);

// Get card by ID - GET /api/cards/:id
router.get('/:id', cardController.getCardById);

// Update card - PUT /api/cards/:id
router.put('/:id', auth, cardController.updateCard);

// Delete card - DELETE /api/cards/:id
router.delete('/:id', auth, cardController.deleteCard);

module.exports = router;
```

### 3.7 สร้าง Server

สร้างไฟล์ `server.js` ในโฟลเดอร์ backend:

```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const connectDB = require('./config/db');
require('dotenv').config();

// Connect to database
connectDB();

const app = express();

// Middleware
app.use(cors());
app.use(bodyParser.json());

// Routes
app.use('/api/users', require('./routes/userRoutes'));
app.use('/api/categories', require('./routes/categoryRoutes'));
app.use('/api/cards', require('./routes/cardRoutes'));

// Basic route
app.get('/', (req, res) => {
  res.send('API is running...');
});

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### 3.8 ทดสอบ Backend

เริ่มต้นเซิร์ฟเวอร์:

```bash
cd backend
npm run dev
```

ทดสอบ API ด้วย Postman หรือโปรแกรมทดสอบ API อื่นๆ

---

## 4. การพัฒนา Frontend ด้วย React

### 4.1 โครงสร้างโฟลเดอร์

เข้าไปที่โฟลเดอร์ frontend และสร้างโฟลเดอร์เพิ่มเติม:

```bash
cd ../frontend
cd src
mkdir components context hooks pages utils styles
```

### 4.2 สร้าง Context สำหรับการจัดการ State

สร้างไฟล์ `context/AuthContext.js`:

```javascript
import React, { createContext, useReducer, useEffect } from 'react';
import axios from 'axios';

// Initial state
const initialState = {
  token: localStorage.getItem('token'),
  isAuthenticated: null,
  loading: true,
  user: null,
  error: null
};

// Create context
export const AuthContext = createContext(initialState);

// Reducer
const authReducer = (state, action) => {
  switch (action.type) {
    case 'USER_LOADED':
      return {
        ...state,
        isAuthenticated: true,
        loading: false,
        user: action.payload
      };
    case 'REGISTER_SUCCESS':
    case 'LOGIN_SUCCESS':
      localStorage.setItem('token', action.payload.token);
      return {
        ...state,
        ...action.payload,
        isAuthenticated: true,
        loading: false,
        error: null
      };
    case 'AUTH_ERROR':
    case 'REGISTER_FAIL':
    case 'LOGIN_FAIL':
    case 'LOGOUT':
      localStorage.removeItem('token');
      return {
        ...state,
        token: null,
        isAuthenticated: false,
        loading: false,
        user: null,
        error: action.payload
      };
    case 'CLEAR_ERROR':
      return {
        ...state,
        error: null
      };
    default:
      return state;
  }
};

// Provider component
export const AuthProvider = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  // Set auth token
  const setAuthToken = token => {
    if (token) {
      axios.defaults.headers.common['x-auth-token'] = token;
    } else {
      delete axios.defaults.headers.common['x-auth-token'];
    }
  };

  // Load user
  const loadUser = async () => {
    if (localStorage.token) {
      setAuthToken(localStorage.token);
    }

    try {
      const res = await axios.get('/api/users/profile');
      dispatch({ type: 'USER_LOADED', payload: res.data });
    } catch (err) {
      dispatch({ type: 'AUTH_ERROR', payload: err.response?.data?.msg });
    }
  };

  // Register user
  const register = async formData => {
    const config = {
      headers: {
        'Content-Type': 'application/json'
      }
    };

    try {
      const res = await axios.post('/api/users/register', formData, config);
      dispatch({ type: 'REGISTER_SUCCESS', payload: res.data });
      await loadUser
