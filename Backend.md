# การสร้างแอปพลิเคชัน Flip-Card ด้วย MERN Stack

ยินดีต้อนรับสู่บทเรียนการพัฒนาแอปพลิเคชัน Flip-Card ส่วนของสัปดาห์ที่ 12 ในรายวิชา ทนด.132 พื้นฐานการใช้งานเครือข่ายคอมพิวเตอร์และความมั่นคงไซเบอร์ ในบทเรียนนี้เราจะเรียนรู้การสร้างแอปพลิเคชัน Flip-Card ที่มีความปลอดภัยโดยใช้ MERN Stack (MongoDB, Express, React, Node.js)

## ภาพรวมของโปรเจค

เราจะพัฒนาแอปพลิเคชันที่มี 2 โหมด:

1. **โหมดสร้าง (Creator Mode)**: ผู้ใช้สามารถสร้างหมวดหมู่และเพิ่มข้อมูลการ์ดโดยใช้ RESTful APIs
2. **โหมดเล่น (Player Mode)**: ผู้ใช้สามารถใช้แอปพลิเคชันเพื่อเรียนรู้ผ่านการพลิกการ์ด

## โครงสร้างโปรเจค

เริ่มต้นด้วยการสร้างโครงสร้างโปรเจคดังนี้:

```
flipcard-app/
  ├── backend/           # โค้ดฝั่งเซิร์ฟเวอร์
  │   ├── config/        # การตั้งค่าต่างๆ
  │   ├── controllers/   # ตัวควบคุมสำหรับจัดการคำขอ API
  │   ├── models/        # โมเดลข้อมูล mongoose
  │   ├── routes/        # เส้นทาง API
  │   ├── middleware/    # middleware ต่างๆ
  │   ├── utils/         # ฟังก์ชันยูทิลิตี้
  │   ├── server.js      # จุดเริ่มต้นของแอพ
  │   └── package.json   # การพึ่งพาของฝั่งเซิร์ฟเวอร์
  │
  └── frontend/          # โค้ดฝั่งไคลเอนต์
      ├── public/        # ไฟล์สาธารณะ
      ├── src/           # โค้ดต้นฉบับ React
      │   ├── components/  # คอมโพเนนต์ที่ใช้ซ้ำได้
      │   ├── pages/       # หน้าเว็บต่างๆ 
      │   ├── context/     # context API
      │   ├── api/         # การเรียก API
      │   ├── styles/      # CSS, styled components
      │   ├── utils/       # ฟังก์ชันยูทิลิตี้
      │   ├── App.js       # คอมโพเนนต์หลัก
      │   └── index.js     # จุดเริ่มต้นของ React
      └── package.json   # การพึ่งพาของฝั่งไคลเอนต์
```

## ขั้นตอนที่ 1: การตั้งค่าโปรเจค

### 1.1 สร้างโฟลเดอร์หลักของโปรเจค

```bash
mkdir flipcard-app
cd flipcard-app
```

### 1.2 ตั้งค่า Backend

```bash
mkdir backend
cd backend
npm init -y
```

เปิดไฟล์ `package.json` และอัปเดตดังนี้:

```json
{
  "name": "flipcard-backend",
  "version": "1.0.0",
  "description": "Backend for Flipcard Application",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "keywords": ["MERN", "flipcard", "education"],
  "author": "Your Name",
  "license": "MIT"
}
```

### 1.3 ติดตั้ง Dependencies สำหรับ Backend

```bash
npm install express mongoose dotenv cors jsonwebtoken bcryptjs helmet express-rate-limit express-validator
npm install nodemon --save-dev
```

รายละเอียดแพ็กเกจ:

- `express`: เฟรมเวิร์กสำหรับสร้าง REST API
- `mongoose`: ODM สำหรับติดต่อกับ MongoDB
- `dotenv`: จัดการตัวแปรสภาพแวดล้อม
- `cors`: จัดการการเข้าถึงข้ามโดเมน
- `jsonwebtoken`: การยืนยันตัวตนด้วย JWT
- `bcryptjs`: เข้ารหัสรหัสผ่าน
- `helmet`: เพิ่มความปลอดภัยให้กับ HTTP headers
- `express-rate-limit`: จำกัดจำนวนคำขอ
- `express-validator`: ตรวจสอบข้อมูลที่รับเข้ามา
- `nodemon`: รีสตาร์ทเซิร์ฟเวอร์อัตโนมัติเมื่อมีการเปลี่ยนแปลงโค้ด

## ขั้นตอนที่ 2: การสร้าง Backend

### 2.1 สร้างโครงสร้างโฟลเดอร์สำหรับ Backend

```bash
mkdir config controllers models routes middleware utils
```

### 2.2 สร้างไฟล์การตั้งค่าสภาพแวดล้อม

สร้างไฟล์ `.env` ในโฟลเดอร์ `backend`:

```
PORT=5000
MONGO_URI=mongodb://localhost:27017/flipcard
JWT_SECRET=your_jwt_secret_key_here
NODE_ENV=development
```

### 2.3 สร้างไฟล์ `config/db.js` สำหรับการเชื่อมต่อฐานข้อมูล

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 2.4 สร้างโมเดลข้อมูล

#### 2.4.1 โมเดลผู้ใช้ (`models/User.js`)

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema(
  {
    username: {
      type: String,
      required: [true, 'กรุณาระบุชื่อผู้ใช้'],
      unique: true,
      trim: true,
    },
    email: {
      type: String,
      required: [true, 'กรุณาระบุอีเมล'],
      unique: true,
      match: [
        /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/,
        'กรุณาระบุอีเมลที่ถูกต้อง',
      ],
    },
    password: {
      type: String,
      required: [true, 'กรุณาระบุรหัสผ่าน'],
      minlength: [8, 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร'],
      select: false,
    },
    role: {
      type: String,
      enum: ['user', 'admin'],
      default: 'user',
    },
  },
  {
    timestamps: true,
  }
);

// เข้ารหัสรหัสผ่านก่อนบันทึก
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) {
    next();
  }
  
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
});

// เปรียบเทียบรหัสผ่าน
userSchema.methods.matchPassword = async function (enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

const User = mongoose.model('User', userSchema);
module.exports = User;
```

#### 2.4.2 โมเดลหมวดหมู่ (`models/Category.js`)

```javascript
const mongoose = require('mongoose');

const categorySchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'กรุณาระบุชื่อหมวดหมู่'],
      trim: true,
    },
    description: {
      type: String,
      trim: true,
    },
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    isPublic: {
      type: Boolean,
      default: false,
    },
  },
  {
    timestamps: true,
  }
);

const Category = mongoose.model('Category', categorySchema);
module.exports = Category;
```

#### 2.4.3 โมเดลการ์ด (`models/Card.js`)

```javascript
const mongoose = require('mongoose');

const cardSchema = new mongoose.Schema(
  {
    category: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Category',
      required: true,
    },
    front: {
      type: String,
      required: [true, 'กรุณาระบุข้อความด้านหน้าการ์ด'],
      trim: true,
    },
    back: {
      type: Array,
      required: [true, 'กรุณาระบุข้อความด้านหลังการ์ด'],
    },
  },
  {
    timestamps: true,
  }
);

const Card = mongoose.model('Card', cardSchema);
module.exports = Card;
```

### 2.5 สร้าง Middleware

#### 2.5.1 สร้าง Middleware สำหรับการยืนยันตัวตน (`middleware/auth.js`)

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// Middleware สำหรับป้องกันเส้นทางที่ต้องการการยืนยันตัวตน
exports.protect = async (req, res, next) => {
  let token;
  
  // ตรวจสอบว่ามี token ใน header หรือไม่
  if (
    req.headers.authorization &&
    req.headers.authorization.startsWith('Bearer')
  ) {
    // แยก token จาก Bearer token
    token = req.headers.authorization.split(' ')[1];
  }
  
  // ตรวจสอบว่ามี token หรือไม่
  if (!token) {
    return res.status(401).json({ 
      success: false, 
      message: 'ไม่มีสิทธิ์เข้าถึง กรุณาล็อกอินก่อน' 
    });
  }
  
  try {
    // ตรวจสอบ token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // ค้นหาผู้ใช้จาก id ใน token และเพิ่มเข้าไปใน req
    req.user = await User.findById(decoded.id);
    next();
  } catch (error) {
    return res.status(401).json({ 
      success: false, 
      message: 'ไม่มีสิทธิ์เข้าถึง' 
    });
  }
};

// Middleware สำหรับตรวจสอบบทบาท (role) ผู้ใช้
exports.authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ 
        success: false, 
        message: 'ไม่มีสิทธิ์เข้าถึงสำหรับบทบาทนี้' 
      });
    }
    next();
  };
};
```

#### 2.5.2 สร้าง Middleware สำหรับจัดการข้อผิดพลาด (`middleware/error.js`)

```javascript
const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // บันทึกข้อผิดพลาดไว้ในคอนโซลสำหรับการพัฒนา
  console.error(err);
  
  // Mongoose: ข้อผิดพลาด ObjectId ไม่ถูกต้อง
  if (err.name === 'CastError') {
    const message = 'ไม่พบข้อมูลที่ต้องการ';
    return res.status(404).json({ success: false, message });
  }
  
  // Mongoose: ข้อผิดพลาดเมื่อมีฟิลด์ซ้ำ
  if (err.code === 11000) {
    const message = 'มีข้อมูลนี้อยู่ในระบบแล้ว';
    return res.status(400).json({ success: false, message });
  }
  
  // Mongoose: ข้อผิดพลาดเมื่อการตรวจสอบล้มเหลว
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message);
    return res.status(400).json({ success: false, message });
  }
  
  // ส่งข้อผิดพลาดทั่วไป
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || 'เกิดข้อผิดพลาดในเซิร์ฟเวอร์',
  });
};

module.exports = errorHandler;
```

#### 2.5.3 สร้าง Middleware สำหรับจำกัดอัตราการขอ (`middleware/rateLimiter.js`)

```javascript
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 นาที
  max: 100, // จำกัดคำขอสูงสุด 100 ครั้งต่อ IP ใน 15 นาที
  message: {
    success: false,
    message: 'มีคำขอมากเกินไป กรุณาลองใหม่ในอีก 15 นาที',
  },
});

module.exports = apiLimiter;
```

### 2.6 สร้าง Controllers

#### 2.6.1 สร้าง Controller สำหรับผู้ใช้ (`controllers/userController.js`)

```javascript
const User = require('../models/User');
const jwt = require('jsonwebtoken');
const { validationResult } = require('express-validator');

// สร้าง JWT token
const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: '30d',
  });
};

// @desc    ลงทะเบียนผู้ใช้ใหม่
// @route   POST /api/users/register
// @access  Public
exports.registerUser = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ success: false, errors: errors.array() });
  }

  try {
    const { username, email, password } = req.body;

    // ตรวจสอบว่ามีผู้ใช้อยู่แล้วหรือไม่
    const userExists = await User.findOne({ email });
    if (userExists) {
      return res.status(400).json({ success: false, message: 'อีเมลนี้ถูกใช้ไปแล้ว' });
    }

    // สร้างผู้ใช้ใหม่
    const user = await User.create({
      username,
      email,
      password,
    });

    if (user) {
      res.status(201).json({
        success: true,
        user: {
          _id: user._id,
          username: user.username,
          email: user.email,
          role: user.role,
        },
        token: generateToken(user._id),
      });
    }
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    ล็อกอินผู้ใช้
// @route   POST /api/users/login
// @access  Public
exports.loginUser = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ success: false, errors: errors.array() });
  }

  try {
    const { email, password } = req.body;

    // ตรวจสอบผู้ใช้
    const user = await User.findOne({ email }).select('+password');
    if (!user) {
      return res.status(401).json({ success: false, message: 'อีเมลหรือรหัสผ่านไม่ถูกต้อง' });
    }

    // ตรวจสอบรหัสผ่าน
    const isMatch = await user.matchPassword(password);
    if (!isMatch) {
      return res.status(401).json({ success: false, message: 'อีเมลหรือรหัสผ่านไม่ถูกต้อง' });
    }

    res.json({
      success: true,
      user: {
        _id: user._id,
        username: user.username,
        email: user.email,
        role: user.role,
      },
      token: generateToken(user._id),
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    รับข้อมูลผู้ใช้ปัจจุบัน
// @route   GET /api/users/me
// @access  Private
exports.getUserProfile = async (req, res) => {
  try {
    const user = await User.findById(req.user._id);
    if (!user) {
      return res.status(404).json({ success: false, message: 'ไม่พบผู้ใช้' });
    }

    res.json({
      success: true,
      user: {
        _id: user._id,
        username: user.username,
        email: user.email,
        role: user.role,
      },
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};
```

#### 2.6.2 สร้าง Controller สำหรับหมวดหมู่ (`controllers/categoryController.js`)

```javascript
const Category = require('../models/Category');
const { validationResult } = require('express-validator');

// @desc    สร้างหมวดหมู่
// @route   POST /api/categories
// @access  Private
exports.createCategory = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ success: false, errors: errors.array() });
  }

  try {
    const { name, description, isPublic } = req.body;

    // สร้างหมวดหมู่ใหม่
    const category = await Category.create({
      name,
      description,
      user: req.user._id,
      isPublic,
    });

    res.status(201).json({
      success: true,
      data: category,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    รับหมวดหมู่ทั้งหมดของผู้ใช้
// @route   GET /api/categories
// @access  Private
exports.getCategories = async (req, res) => {
  try {
    // ค้นหาหมวดหมู่ที่เป็นของผู้ใช้หรือหมวดหมู่สาธารณะ
    const categories = await Category.find({
      $or: [
        { user: req.user._id },
        { isPublic: true },
      ],
    });

    res.json({
      success: true,
      count: categories.length,
      data: categories,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    รับหมวดหมู่ตาม ID
// @route   GET /api/categories/:id
// @access  Private
exports.getCategoryById = async (req, res) => {
  try {
    const category = await Category.findById(req.params.id);

    if (!category) {
      return res.status(404).json({ success: false, message: 'ไม่พบหมวดหมู่' });
    }

    // ตรวจสอบความเป็นเจ้าของหรือสาธารณะ
    if (category.user.toString() !== req.user._id.toString() && !category.isPublic) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์เข้าถึง' });
    }

    res.json({
      success: true,
      data: category,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    อัปเดตหมวดหมู่
// @route   PUT /api/categories/:id
// @access  Private
exports.updateCategory = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ success: false, errors: errors.array() });
  }

  try {
    let category = await Category.findById(req.params.id);

    if (!category) {
      return res.status(404).json({ success: false, message: 'ไม่พบหมวดหมู่' });
    }

    // ตรวจสอบความเป็นเจ้าของ
    if (category.user.toString() !== req.user._id.toString()) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์แก้ไข' });
    }

    const { name, description, isPublic } = req.body;

    // อัปเดตหมวดหมู่
    category = await Category.findByIdAndUpdate(
      req.params.id,
      { name, description, isPublic },
      { new: true, runValidators: true }
    );

    res.json({
      success: true,
      data: category,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    ลบหมวดหมู่
// @route   DELETE /api/categories/:id
// @access  Private
exports.deleteCategory = async (req, res) => {
  try {
    const category = await Category.findById(req.params.id);

    if (!category) {
      return res.status(404).json({ success: false, message: 'ไม่พบหมวดหมู่' });
    }

    // ตรวจสอบความเป็นเจ้าของ
    if (category.user.toString() !== req.user._id.toString()) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์ลบ' });
    }

    await category.remove();

    res.json({
      success: true,
      data: {},
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};
```

#### 2.6.3 สร้าง Controller สำหรับการ์ด (`controllers/cardController.js`)

```javascript
const Card = require('../models/Card');
const Category = require('../models/Category');
const { validationResult } = require('express-validator');

// @desc    สร้างการ์ด
// @route   POST /api/categories/:categoryId/cards
// @access  Private
exports.createCard = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ success: false, errors: errors.array() });
  }

  try {
    const { front, back } = req.body;
    const categoryId = req.params.categoryId;

    // ตรวจสอบว่าหมวดหมู่มีอยู่จริงและเป็นของผู้ใช้
    const category = await Category.findById(categoryId);
    if (!category) {
      return res.status(404).json({ success: false, message: 'ไม่พบหมวดหมู่' });
    }

    if (category.user.toString() !== req.user._id.toString()) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์เพิ่มการ์ดในหมวดหมู่นี้' });
    }

    // สร้างการ์ดใหม่
    const card = await Card.create({
      category: categoryId,
      front,
      back,
    });

    res.status(201).json({
      success: true,
      data: card,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    รับการ์ดทั้งหมดในหมวดหมู่
// @route   GET /api/categories/:categoryId/cards
// @access  Private
exports.getCards = async (req, res) => {
  try {
    const categoryId = req.params.categoryId;

    // ตรวจสอบว่าหมวดหมู่มีอยู่จริง
    const category = await Category.findById(categoryId);
    if (!category) {
      return res.status(404).json({ success: false, message: 'ไม่พบหมวดหมู่' });
    }

    // ตรวจสอบความเป็นเจ้าของหรือสาธารณะ
    if (category.user.toString() !== req.user._id.toString() && !category.isPublic) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์เข้าถึง' });
    }

    // รับการ์ดทั้งหมดในหมวดหมู่
    const cards = await Card.find({ category: categoryId });

    res.json({
      success: true,
      count: cards.length,
      data: cards,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    รับการ์ดตาม ID
// @route   GET /api/cards/:id
// @access  Private
exports.getCardById = async (req, res) => {
  try {
    const card = await Card.findById(req.params.id).populate('category');

    if (!card) {
      return res.status(404).json({ success: false, message: 'ไม่พบการ์ด' });
    }

    // ตรวจสอบความเป็นเจ้าของหรือสาธารณะ
    if (
      card.category.user.toString() !== req.user._id.toString() &&
      !card.category.isPublic
    ) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์เข้าถึง' });
    }

    res.json({
      success: true,
      data: card,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    อัปเดตการ์ด
// @route   PUT /api/cards/:id
// @access  Private
exports.updateCard = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ success: false, errors: errors.array() });
  }

  try {
    let card = await Card.findById(req.params.id).populate('category');

    if (!card) {
      return res.status(404).json({ success: false, message: 'ไม่พบการ์ด' });
    }

    // ตรวจสอบความเป็นเจ้าของ
    if (card.category.user.toString() !== req.user._id.toString()) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์แก้ไข' });
    }

    const { front, back } = req.body;

    // อัปเดตการ์ด
    card = await Card.findByIdAndUpdate(
      req.params.id,
      { front, back },
      { new: true, runValidators: true }
    );

    res.json({
      success: true,
      data: card,
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};

// @desc    ลบการ์ด
// @route   DELETE /api/cards/:id
// @access  Private
exports.deleteCard = async (req, res) => {
  try {
    const card = await Card.findById(req.params.id).populate('category');

    if (!card) {
      return res.status(404).json({ success: false, message: 'ไม่พบการ์ด' });
    }

    // ตรวจสอบความเป็นเจ้าของ
    if (card.category.user.toString() !== req.user._id.toString()) {
      return res.status(403).json({ success: false, message: 'ไม่มีสิทธิ์ลบ' });
    }

    await card.remove();

    res.json({
      success: true,
      data: {},
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
};
```

### 2.7 สร้าง Routes

#### 2.7.1 สร้าง Routes สำหรับผู้ใช้ (`routes/userRoutes.js`)

```javascript
const express = require('express');
const { check } = require('express-validator');
const router = express.Router();
const {
  registerUser,
  loginUser,
  getUserProfile,
} = require('../controllers/userController');
const { protect } = require('../middleware/auth');

// @route   POST /api/users/register
router.post(
  '/register',
  [
    check('username', 'กรุณาระบุชื่อผู้ใช้').not().isEmpty(),
    check('email', 'กรุณาระบุอีเมลที่ถูกต้อง').isEmail(),
    check('password', 'กรุณาระบุรหัสผ่านที่มีความยาวอย่างน้อย 8 ตัวอักษร').isLength({ min: 8 }),
  ],
  registerUser
);

// @route   POST /api/users/login
router.post(
  '/login',
  [
    check('email', 'กรุณาระบุอีเมลที่ถูกต้อง').isEmail(),
    check('password', 'กรุณาระบุรหัสผ่าน').exists(),
  ],
  loginUser
);

// @route   GET /api/users/me
router.get('/me', protect, getUserProfile);

module.exports = router;
```

#### 2.7.2 สร้าง Routes สำหรับหมวดหมู่ (`routes/categoryRoutes.js`)

```javascript
const express = require('express');
const { check } = require('express-validator');
const router = express.Router();
const {
  createCategory,
  getCategories,
  getCategoryById,
  updateCategory,
  deleteCategory,
} = require('../controllers/categoryController');
const { protect } = require('../middleware/auth');

// @route   POST /api/categories
router.post(
  '/',
  [
    protect,
    check('name', 'กรุณาระบุชื่อหมวดหมู่').not().isEmpty(),
  ],
  createCategory
);

// @route   GET /api/categories
router.get('/', protect, getCategories);

// @route   GET /api/categories/:id
router.get('/:id', protect, getCategoryById);

// @route   PUT /api/categories/:id
router.put(
  '/:id',
  [
    protect,
    check('name', 'กรุณาระบุชื่อหมวดหมู่').not().isEmpty(),
  ],
  updateCategory
);

// @route   DELETE /api/categories/:id
router.delete('/:id', protect, deleteCategory);

module.exports = router;
```

#### 2.7.3 สร้าง Routes สำหรับการ์ด (`routes/cardRoutes.js`)

```javascript
const express = require('express');
const { check } = require('express-validator');
const router = express.Router({ mergeParams: true });
const {
  createCard,
  getCards,
  getCardById,
  updateCard,
  deleteCard,
} = require('../controllers/cardController');
const { protect } = require('../middleware/auth');

// @route   POST /api/categories/:categoryId/cards
router.post(
  '/',
  [
    protect,
    check('front', 'กรุณาระบุข้อความด้านหน้าการ์ด').not().isEmpty(),
    check('back', 'กรุณาระบุข้อความด้านหลังการ์ด').isArray().not().isEmpty(),
  ],
  createCard
);

// @route   GET /api/categories/:categoryId/cards
router.get('/', protect, getCards);

// @route   GET /api/cards/:id (ต้องกำหนดใน server.js)
// @route   PUT /api/cards/:id (ต้องกำหนดใน server.js)
// @route   DELETE /api/cards/:id (ต้องกำหนดใน server.js)

module.exports = router;
```

### 2.8 สร้างไฟล์ `server.js`

```javascript
const express = require('express');
const dotenv = require('dotenv');
const cors = require('cors');
const helmet = require('helmet');
const connectDB = require('./config/db');
const errorHandler = require('./middleware/error');
const apiLimiter = require('./middleware/rateLimiter');
const { protect } = require('./middleware/auth');

// โหลดตัวแปรสภาพแวดล้อม
dotenv.config();

// เชื่อมต่อฐานข้อมูล
connectDB();

// ตั้งค่า Express
const app = express();

// Middleware
app.use(express.json());
app.use(cors());
app.use(helmet());
app.use('/api', apiLimiter);

// Routes
const userRoutes = require('./routes/userRoutes');
const categoryRoutes = require('./routes/categoryRoutes');
const cardRoutes = require('./routes/cardRoutes');
const cardController = require('./controllers/cardController');

app.use('/api/users', userRoutes);
app.use('/api/categories', categoryRoutes);
app.use('/api/categories/:categoryId/cards', cardRoutes);

// Routes สำหรับการ์ดแบบอิสระ (ไม่ผ่าน categoryId)
app.get('/api/cards/:id', protect, cardController.getCardById);
app.put(
  '/api/cards/:id',
  [
    protect,
    require('express-validator').check('front', 'กรุณาระบุข้อความด้านหน้าการ์ด').not().isEmpty(),
    require('express-validator').check('back', 'กรุณาระบุข้อความด้านหลังการ์ด').isArray().not().isEmpty(),
  ],
  cardController.updateCard
);
app.delete('/api/cards/:id', protect, cardController.deleteCard);

// ตั้งค่า API สำหรับตรวจสอบสถานะ
app.get('/', (req, res) => {
  res.json({ message: 'API ของแอปพลิเคชัน Flip-Card' });
});

// จัดการข้อผิดพลาด
app.use(errorHandler);

// กำหนดพอร์ต
const PORT = process.env.PORT || 5000;

// เริ่มเซิร์ฟเวอร์
app.listen(PORT, () => {
  console.log(`เซิร์ฟเวอร์กำลังทำงานที่พอร์ต ${PORT} ในโหมด ${process.env.NODE_ENV}`);
});

// จัดการข้อผิดพลาดที่ไม่ได้จัดการ
process.on('unhandledRejection', (err, promise) => {
  console.error(`ข้อผิดพลาด: ${err.message}`);
  // ปิดเซิร์ฟเวอร์และออกจากกระบวนการ
  // server.close(() => process.exit(1));
});
```

### 2.9 สร้างไฟล์ `utils/generateToken.js` (แยกฟังก์ชันการสร้าง Token)

```javascript
const jwt = require('jsonwebtoken');

// สร้าง JWT token
const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: '30d',
  });
};

module.exports = generateToken;
```

## ขั้นตอนที่ 3: ทดสอบ Backend

### 3.1 ทดสอบเซิร์ฟเวอร์ว่าทำงานหรือไม่

ในโฟลเดอร์ `backend` ให้รันคำสั่ง:

```bash
npm run dev
```

ถ้าเซิร์ฟเวอร์ทำงานถูกต้อง จะมีข้อความแสดงในคอนโซลว่า:

```
เซิร์ฟเวอร์กำลังทำงานที่พอร์ต 5000 ในโหมด development
MongoDB Connected: <ชื่อโฮสต์ของ MongoDB>
```

### 3.2 ทดสอบ API ด้วย Postman หรือ Insomnia

สามารถทดสอบ API ต่างๆ ด้วย Postman หรือ Insomnia ดังนี้:

#### 3.2.1 ลงทะเบียนผู้ใช้ใหม่

```
POST http://localhost:5000/api/users/register
Content-Type: application/json

{
  "username": "testuser",
  "email": "test@example.com",
  "password": "password123"
}
```

#### 3.2.2 ล็อกอินผู้ใช้

```
POST http://localhost:5000/api/users/login
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "password123"
}
```

#### 3.2.3 สร้างหมวดหมู่ใหม่

```
POST http://localhost:5000/api/categories
Content-Type: application/json
Authorization: Bearer <token_จากการล็อกอิน>

{
  "name": "พื้นฐาน AAA",
  "description": "การยืนยันตัวตนและการควบคุมการเข้าถึง",
  "isPublic": true
}
```

#### 3.2.4 สร้างการ์ดใหม่

```
POST http://localhost:5000/api/categories/<category_id>/cards
Content-Type: application/json
Authorization: Bearer <token_จากการล็อกอิน>

{
  "front": "❓ AAA ย่อมาจากอะไร?<br>และแต่ละตัวหมายถึงอะไร?",
  "back": [
    "A - Authentication (การพิสูจน์ตัวตน): คุณเป็นใคร?",
    "A - Authorization (การตรวจสอบสิทธิ์): คุณทำอะไรได้บ้าง?",
    "A - Accounting (การบันทึกกิจกรรม): คุณทำอะไรไปแล้วบ้าง?"
  ]
}
```

## สรุปขั้นตอนที่ได้ทำไปแล้ว

เราได้สร้าง Backend สำหรับแอปพลิเคชัน Flip-Card โดยใช้ Node.js, Express และ MongoDB ซึ่งมีความสามารถดังนี้:

1. การยืนยันตัวตนและการควบคุมการเข้าถึงด้วย JWT
2. การจัดการผู้ใช้ (ลงทะเบียน, ล็อกอิน, ดูโปรไฟล์)
3. การจัดการหมวดหมู่ (สร้าง, ดู, แก้ไข, ลบ)
4. การจัดการการ์ด (สร้าง, ดู, แก้ไข, ลบ)
5. การรักษาความปลอดภัยด้วย Helmet, Rate Limiting, และการตรวจสอบความถูกต้องของข้อมูล

ในส่วนต่อไปเราจะสร้าง Frontend ด้วย React เพื่อให้ผู้ใช้สามารถโต้ตอบกับแอปพลิเคชันผ่านทางเว็บเบราว์เซอร์ได้

