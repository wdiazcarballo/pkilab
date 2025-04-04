# คพ.234 / ทนด.132 - Week 11 Lab
# การระบุตัวตนและการควบคุมการเข้าถึง (Identity and Access Management)
# สถานการณ์จำลอง: ระบบความปลอดภัยของ TechSecure Solutions

**วัตถุประสงค์การเรียนรู้:**
1. เข้าใจหลักการทำงานของระบบยืนยันตัวตนหลายปัจจัย (MFA) และ TOTP
2. สามารถพัฒนาระบบ AAA (Authentication, Authorization, Accounting) ที่มีความปลอดภัยสูง
3. วิเคราะห์ความเสี่ยงและประเมินภัยคุกคามด้านความมั่นคงไซเบอร์
4. ประยุกต์ใช้มาตรฐาน ISO/IEC 27001 และ PDPA ในการพัฒนาระบบ
5. พัฒนาทักษะการทำงานเป็นทีมในการแก้ไขปัญหาด้านความมั่นคงปลอดภัย

## ภาพรวมสถานการณ์จำลอง

นักศึกษาได้รับการว่าจ้างเป็นผู้เชี่ยวชาญด้านความปลอดภัยไซเบอร์ที่บริษัท TechSecure Solutions ซึ่งเป็นบริษัทสตาร์ทอัพที่กำลังเติบโตอย่างรวดเร็ว บริษัทนี้ให้บริการระบบคลาวด์สำหรับเก็บข้อมูลทางการแพทย์ที่มีความอ่อนไหวสูงและต้องปฏิบัติตามกฎหมาย PDPA อย่างเคร่งครัด

**เหตุการณ์ล่าสุด:** เมื่อเดือนที่แล้ว บริษัทคู่แข่งในอุตสาหกรรมเดียวกันถูกแฮกและข้อมูลส่วนตัวของผู้ป่วยถูกขโมยไป ส่งผลให้มีค่าปรับตามกฎหมายกว่า 15 ล้านบาท และเสียชื่อเสียงอย่างมาก ทีม CTI ของนักศึกษาได้วิเคราะห์ว่าการโจมตีเกิดจากช่องโหว่ในระบบยืนยันตัวตนที่ใช้เพียงรหัสผ่านเพียงชั้นเดียว โดยผู้โจมตีได้ใช้การโจมตีแบบ Credential Stuffing และ Account Takeover (ATO)

**ภารกิจของนักศึกษา:** CEO ของ TechSecure Solutions ได้มอบหมายให้ทีมของนักศึกษาพัฒนาระบบ AAA (Authentication, Authorization, and Accounting) ที่มีความปลอดภัยสูงสำหรับพนักงานและลูกค้าเพื่อป้องกันไม่ให้เกิดเหตุการณ์แบบเดียวกัน โดยระบบนี้จะต้องสอดคล้องกับมาตรฐาน ISO/IEC 27001

**ความเร่งด่วน:** ทีมของนักศึกษามีเวลาเพียง 2 สัปดาห์ในการพัฒนาระบบต้นแบบ เนื่องจาก TechSecure Solutions กำลังจะเปิดตัวผลิตภัณฑ์ใหม่ที่จะมีผู้ใช้งานเพิ่มขึ้นอีก 10,000 ราย ทีมบริหารไม่ต้องการให้เกิดปัญหาด้านความปลอดภัยที่จะทำลายความน่าเชื่อถือของบริษัท

**ความท้าทาย:** ระบบต้องใช้งานง่าย ไม่ซับซ้อนเกินไปสำหรับผู้ใช้ แต่ต้องมีความปลอดภัยสูง ทีมของนักศึกษาต้องสร้างสมดุลระหว่างความปลอดภัยและความสะดวกในการใช้งาน (Security vs. Usability)

## บทบาทของนักศึกษา

นักศึกษาและเพื่อนร่วมทีมอีกหนึ่งคนเป็นผู้เชี่ยวชาญด้านความมั่นคงไซเบอร์ที่ได้รับมอบหมายให้:

1. **ออกแบบและพัฒนา:** ระบบยืนยันตัวตนและการเข้าถึงที่ปลอดภัย โดยใช้การยืนยันตัวตนหลายปัจจัย (Multi-factor Authentication) พร้อมกับการใช้โทเคนแบบใช้ครั้งเดียว (One-Time Password) เพื่อเสริมความปลอดภัยให้กับระบบ

2. **วิเคราะห์ความเสี่ยง:** ประเมินภัยคุกคามที่อาจเกิดขึ้นกับระบบและเสนอแนวทางการลดความเสี่ยง

3. **จัดทำระบบตรวจสอบ:** สร้างระบบบันทึกและตรวจสอบกิจกรรมที่ครอบคลุม เพื่อให้สามารถตรวจจับพฤติกรรมที่น่าสงสัยได้อย่างทันท่วงที

4. **นำเสนอผลงาน:** ต่อทีมผู้บริหารระดับสูงที่มีความรู้ด้านเทคนิคจำกัด แต่สนใจในเรื่องความเสี่ยงและการลงทุน

5. **จัดทำเอกสาร:** คู่มือการใช้งานและเอกสารทางเทคนิคสำหรับทีมพัฒนาและทีมปฏิบัติการ

## ความต้องการของระบบ

1. ระบบต้องรองรับการพิสูจน์ตัวตนด้วยหลายปัจจัย (MFA) อย่างน้อย 2 ปัจจัย
2. ต้องใช้โทเคนแบบใช้ครั้งเดียว (OTP) ที่มีความปลอดภัยสูงและอายุการใช้งานที่จำกัด
3. ต้องมีระบบบันทึกการเข้าถึงและตรวจสอบย้อนหลังได้ (Audit Trail) ครบถ้วนตามหลัก 5W1H
4. ต้องสอดคล้องกับมาตรฐาน ISO/IEC 27001 และ PDPA โดยเฉพาะในเรื่องการเก็บข้อมูลส่วนบุคคล
5. ต้องมีระบบตรวจจับความผิดปกติและแจ้งเตือน รวมถึงการป้องกันการโจมตีแบบ Brute Force
6. ต้องมีระบบจัดการสิทธิ์การเข้าถึงตามหลักการให้สิทธิ์น้อยที่สุดเท่าที่จำเป็น (Principle of Least Privilege)
7. ต้องสามารถรองรับการขยายตัวในอนาคตและการบูรณาการกับระบบอื่นได้

### การบูรณาการกับ ACSC Essential 8

ระบบนี้ยังต้องสอดคล้องกับมาตรการความปลอดภัยของ ACSC Essential 8 ในหัวข้อต่อไปนี้:
- การควบคุมการใช้งานแอปพลิเคชัน (Application Control)
- การแก้ไขช่องโหว่ของแอปพลิเคชัน (Patch Applications)
- การกำหนดค่าของแอปพลิเคชัน (Configure Applications)
- การมีระบบป้องกันการรั่วไหลของข้อมูล (Data Loss Prevention)
- การจำกัดสิทธิ์ผู้ดูแลระบบ (Restrict Administrative Privileges)

## การประเมินผล

ผลงานของนักศึกษาจะได้รับการประเมินตามเกณฑ์ดังต่อไปนี้:

| เกณฑ์การประเมิน | คะแนนเต็ม | รายละเอียด |
|----------------|-----------|-----------|
| ความเข้าใจในแนวคิด TOTP/MFA | 20% | ความถูกต้องในการอธิบายหลักการทำงาน, การระบุจุดแข็ง/จุดอ่อน, การประยุกต์ใช้ |
| คุณภาพของโค้ดที่พัฒนา | 25% | ความถูกต้องของการทำงาน, โครงสร้างโค้ด, ความสะอาดของโค้ด, การจัดการข้อผิดพลาด |
| ฟีเจอร์ความปลอดภัยที่เพิ่มเติม | 25% | ความคิดสร้างสรรค์, ความเป็นไปได้ในการนำไปใช้จริง, ระดับความปลอดภัยที่เพิ่มขึ้น |
| รายงานและการวิเคราะห์ | 20% | ความครบถ้วนของการวิเคราะห์, การอ้างอิงมาตรฐาน, ข้อเสนอแนะที่มีคุณค่า |
| การนำเสนอ | 10% | ความชัดเจน, การตอบคำถาม, การรักษาเวลา, ความน่าสนใจของการนำเสนอ |

## ขั้นตอนการทำงาน

### ส่วนที่ 1: การตั้งค่าโครงการ Node.js

1. **เริ่มต้นโครงการ Node.js**
   ```bash
   mkdir totp-auth-lab
   cd totp-auth-lab
   npm init -y
   ```

2. **ติดตั้งแพ็คเกจที่จำเป็น**
   ```bash
   npm install express speakeasy qrcode
   ```

3. **สร้างไฟล์ server.js**
   เปิดโปรแกรมแก้ไขข้อความและสร้างไฟล์ server.js โดยใช้โค้ดต่อไปนี้:

```javascript
const express = require('express');
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');
const app = express();

app.use(express.json());
app.use(express.static('public'));

// เก็บข้อมูล secret ชั่วคราว (ในระบบจริงควรใช้ฐานข้อมูล)
let userSecrets = {};
let loginAttempts = {};
const MAX_ATTEMPTS = 5;
const LOCKOUT_TIME = 15 * 60 * 1000; // 15 นาที

// ระบบการล็อกบัญชีเมื่อมีการเข้าสู่ระบบที่ผิดพลาดเกินกำหนด
function checkLoginAttempts(userId) {
  if (!loginAttempts[userId]) {
    loginAttempts[userId] = {
      count: 0,
      lockUntil: 0
    };
  }
  
  const now = Date.now();
  if (loginAttempts[userId].lockUntil > now) {
    const remainingTime = Math.ceil((loginAttempts[userId].lockUntil - now) / 1000 / 60);
    return {
      allowed: false,
      message: `บัญชีถูกล็อก กรุณาลองอีกครั้งในอีก ${remainingTime} นาที`
    };
  }
  
  return { allowed: true };
}

function incrementLoginAttempts(userId) {
  loginAttempts[userId].count += 1;
  
  if (loginAttempts[userId].count >= MAX_ATTEMPTS) {
    loginAttempts[userId].lockUntil = Date.now() + LOCKOUT_TIME;
    loginAttempts[userId].count = 0;
    return false;
  }
  
  return true;
}

function resetLoginAttempts(userId) {
  if (loginAttempts[userId]) {
    loginAttempts[userId].count = 0;
  }
}

// สร้าง Secret และ QR Code
app.post('/api/generate-secret', (req, res) => {
  const { userId } = req.body;
  
  if (!userId) {
    return res.status(400).json({ error: 'กรุณาระบุ User ID' });
  }
  
  // สร้างข้อมูลลับสำหรับ TOTP
  const secret = speakeasy.generateSecret({
    length: 20,
    name: `TechSecure:${userId}`
  });
  
  // เก็บข้อมูลลับไว้ (ในระบบจริงควรเก็บในฐานข้อมูล)
  userSecrets[userId] = {
    base32: secret.base32,
    registrationTime: new Date().toISOString(),
    lastUsed: null
  };
  
  // สร้าง QR Code
  QRCode.toDataURL(secret.otpauth_url, (err, dataUrl) => {
    if (err) {
      return res.status(500).json({ error: 'ไม่สามารถสร้าง QR Code ได้' });
    }
    
    // บันทึกกิจกรรม (Audit log)
    console.log(`[${new Date().toISOString()}] User ${userId} ได้ลงทะเบียน TOTP`);
    
    res.json({ 
      success: true, 
      secret: secret.base32, 
      qrCode: dataUrl 
    });
  });
});

// ตรวจสอบความถูกต้องของ TOTP
app.post('/api/verify', (req, res) => {
  const { userId, token } = req.body;
  
  if (!userId || !token) {
    return res.status(400).json({ error: 'กรุณาระบุ User ID และ Token' });
  }
  
  // ตรวจสอบว่ามีข้อมูลลับของผู้ใช้หรือไม่
  if (!userSecrets[userId]) {
    return res.status(404).json({ error: 'ไม่พบข้อมูลผู้ใช้' });
  }
  
  // ตรวจสอบการล็อก
  const loginCheck = checkLoginAttempts(userId);
  if (!loginCheck.allowed) {
    return res.status(403).json({ error: loginCheck.message });
  }
  
  // ตรวจสอบความถูกต้องของ TOTP
  const verified = speakeasy.totp.verify({
    secret: userSecrets[userId].base32,
    encoding: 'base32',
    token: token,
    window: 1 // อนุญาตให้มีความคลาดเคลื่อนได้ 1 ช่วงเวลา (±30 วินาที)
  });
  
  if (verified) {
    // บันทึกเวลาที่ใช้งานล่าสุด
    userSecrets[userId].lastUsed = new Date().toISOString();
    
    // บันทึกกิจกรรม (Audit log)
    console.log(`[${new Date().toISOString()}] User ${userId} ยืนยันตัวตนสำเร็จ`);
    
    // รีเซ็ตการนับจำนวนครั้งที่ล็อกอินผิดพลาด
    resetLoginAttempts(userId);
    
    res.json({ 
      success: true, 
      message: 'การยืนยันตัวตนสำเร็จ',
      timestamp: new Date().toISOString()
    });
  } else {
    // บันทึกความล้มเหลวในการยืนยันตัวตน
    console.log(`[${new Date().toISOString()}] User ${userId} ยืนยันตัวตนล้มเหลว`);
    
    // เพิ่มจำนวนครั้งที่ล็อกอินผิดพลาด
    const canContinue = incrementLoginAttempts(userId);
    
    if (!canContinue) {
      return res.status(403).json({ 
        success: false, 
        error: 'มีการพยายามเข้าสู่ระบบที่ผิดพลาดเกินกำหนด บัญชีถูกล็อกชั่วคราว' 
      });
    }
    
    res.status(401).json({ 
      success: false, 
      error: 'รหัส OTP ไม่ถูกต้อง' 
    });
  }
});

// แสดงประวัติการใช้งาน (Audit Trail)
app.get('/api/audit-trail/:userId', (req, res) => {
  const { userId } = req.params;
  
  if (!userSecrets[userId]) {
    return res.status(404).json({ error: 'ไม่พบข้อมูลผู้ใช้' });
  }
  
  res.json({
    userId,
    registrationTime: userSecrets[userId].registrationTime,
    lastUsed: userSecrets[userId].lastUsed
  });
});

// เริ่มต้นเซิร์ฟเวอร์
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`เซิร์ฟเวอร์ทำงานที่พอร์ต ${PORT}`);
  console.log(`http://localhost:${PORT}`);
});
```

### ส่วนที่ 2: การพัฒนาส่วนติดต่อผู้ใช้

1. **สร้างโฟลเดอร์ public และไฟล์ HTML**

   สร้างโฟลเดอร์ชื่อ `public` ในโฟลเดอร์โครงการและสร้างไฟล์ `index.html` ในโฟลเดอร์นั้น:

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TechSecure Solutions - ระบบยืนยันตัวตนและควบคุมการเข้าถึง</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <div class="container">
    <header>
      <h1>TechSecure Solutions</h1>
      <h2>ระบบยืนยันตัวตนและควบคุมการเข้าถึง</h2>
    </header>

    <div class="card">
      <div id="register-section">
        <h3>ลงทะเบียนระบบยืนยันตัวตนแบบ TOTP</h3>
        <div class="form-group">
          <label for="userId">รหัสผู้ใช้:</label>
          <input type="text" id="userId" placeholder="กรอกรหัสผู้ใช้">
        </div>
        <button id="generateBtn">ลงทะเบียน TOTP</button>
        <div id="qrcode-container" class="hidden">
          <p>สแกน QR Code ด้วยแอปพลิเคชัน Authenticator (เช่น Google Authenticator, Microsoft Authenticator)</p>
          <div id="qrcode"></div>
          <div class="secret-container">
            <p>หรือป้อนรหัสลับนี้ในแอปพลิเคชันของคุณ:</p>
            <code id="secret-key"></code>
          </div>
        </div>
      </div>

      <div id="verify-section" class="hidden">
        <h3>ยืนยันตัวตนด้วย TOTP</h3>
        <div class="form-group">
          <label for="token">รหัส TOTP:</label>
          <input type="text" id="token" placeholder="กรอกรหัส 6 หลักจากแอปพลิเคชัน Authenticator">
        </div>
        <button id="verifyBtn">ยืนยันตัวตน</button>
        <div id="result-container" class="hidden">
          <p id="result-message"></p>
          <p id="timestamp"></p>
        </div>
      </div>

      <div id="audit-section" class="hidden">
        <h3>ประวัติการใช้งาน</h3>
        <button id="auditBtn">แสดงประวัติการใช้งาน</button>
        <div id="audit-result" class="hidden">
          <p id="registration-time"></p>
          <p id="last-used"></p>
        </div>
      </div>
    </div>

    <div class="info-box">
      <h3>ข้อมูลเกี่ยวกับความปลอดภัยของระบบ</h3>
      <ul>
        <li>ระบบนี้ใช้ Time-based One-Time Password (TOTP) ซึ่งเป็นมาตรฐานการยืนยันตัวตนที่ปลอดภัย</li>
        <li>รหัส TOTP จะเปลี่ยนทุก 30 วินาที และใช้ได้เพียงครั้งเดียว</li>
        <li>การพยายามเข้าสู่ระบบที่ผิดพลาด 5 ครั้งติดต่อกันจะทำให้บัญชีถูกล็อกชั่วคราว</li>
        <li>ระบบมีการบันทึกกิจกรรมทั้งหมดเพื่อการตรวจสอบย้อนหลัง</li>
        <li>มาตรฐานที่ใช้: ISO/IEC 27001 และสอดคล้องกับ PDPA</li>
      </ul>
    </div>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

2. **สร้างไฟล์ CSS**

   สร้างไฟล์ `styles.css` ในโฟลเดอร์ `public`:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body {
  background-color: #f5f5f5;
  padding: 20px;
  color: #333;
}

.container {
  max-width: 800px;
  margin: 0 auto;
}

header {
  text-align: center;
  margin-bottom: 30px;
}

header h1 {
  color: #007bff;
  margin-bottom: 10px;
}

.card {
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

h3 {
  margin-bottom: 15px;
  color: #333;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input[type="text"] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #0056b3;
}

.hidden {
  display: none;
}

#qrcode-container {
  margin-top: 20px;
  text-align: center;
}

#qrcode img {
  margin: 15px auto;
  display: block;
}

.secret-container {
  margin-top: 15px;
  padding: 10px;
  background-color: #f8f9fa;
  border-radius: 4px;
}

code {
  font-family: monospace;
  background-color: #e9ecef;
  padding: 3px 5px;
  border-radius: 3px;
}

#result-container, #audit-result {
  margin-top: 15px;
  padding: 10px;
  border-radius: 4px;
}

.success {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.error {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.info-box {
  background-color: #e9f5fe;
  border-left: 5px solid #007bff;
  padding: 15px;
  border-radius: 4px;
}

.info-box h3 {
  border-bottom: none;
  color: #007bff;
  margin-bottom: 10px;
}

.info-box ul {
  margin-left: 20px;
}

.info-box li {
  margin-bottom: 8px;
}
```

3. **สร้างไฟล์ JavaScript**

   สร้างไฟล์ `app.js` ในโฟลเดอร์ `public`:

```javascript
// เก็บข้อมูลผู้ใช้ปัจจุบัน
let currentUser = null;

// เลือกองค์ประกอบ
const userIdInput = document.getElementById('userId');
const generateBtn = document.getElementById('generateBtn');
const qrcodeContainer = document.getElementById('qrcode-container');
const qrcodeElement = document.getElementById('qrcode');
const secretKeyElement = document.getElementById('secret-key');
const verifySection = document.getElementById('verify-section');
const tokenInput = document.getElementById('token');
const verifyBtn = document.getElementById('verifyBtn');
const resultContainer = document.getElementById('result-container');
const resultMessage = document.getElementById('result-message');
const timestamp = document.getElementById('timestamp');
const auditSection = document.getElementById('audit-section');
const auditBtn = document.getElementById('auditBtn');
const auditResult = document.getElementById('audit-result');
const registrationTime = document.getElementById('registration-time');
const lastUsed = document.getElementById('last-used');

// สร้าง Secret และ QR Code
generateBtn.addEventListener('click', async () => {
  const userId = userIdInput.value.trim();
  
  if (!userId) {
    alert('กรุณากรอกรหัสผู้ใช้');
    return;
  }
  
  try {
    const response = await fetch('/api/generate-secret', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ userId })
    });
    
    const data = await response.json();
    
    if (data.success) {
      // แสดง QR Code และรหัสลับ
      qrcodeElement.innerHTML = `<img src="${data.qrCode}" alt="QR Code">`;
      secretKeyElement.textContent = data.secret;
      qrcodeContainer.classList.remove('hidden');
      
      // แสดงส่วนยืนยันตัวตน
      verifySection.classList.remove('hidden');
      auditSection.classList.remove('hidden');
      
      // บันทึกผู้ใช้ปัจจุบัน
      currentUser = userId;
    } else {
      alert(data.error || 'เกิดข้อผิดพลาด กรุณาลองอีกครั้ง');
    }
  } catch (error) {
    console.error('Error:', error);
    alert('เกิดข้อผิดพลาดในการเชื่อมต่อกับเซิร์ฟเวอร์');
  }
});

// ยืนยันตัวตนด้วย TOTP
verifyBtn.addEventListener('click', async () => {
  const token = tokenInput.value.trim();
  
  if (!currentUser) {
    alert('กรุณาลงทะเบียนก่อน');
    return;
  }
  
  if (!token) {
    alert('กรุณากรอกรหัส TOTP');
    return;
  }
  
  try {
    const response = await fetch('/api/verify', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ userId: currentUser, token })
    });
    
    const data = await response.json();
    
    resultContainer.classList.remove('hidden');
    
    if (response.ok && data.success) {
      resultContainer.className = 'success';
      resultMessage.textContent = 'การยืนยันตัวตนสำเร็จ!';
      timestamp.textContent = `เวลา: ${new Date(data.timestamp).toLocaleString()}`;
    } else {
      resultContainer.className = 'error';
      resultMessage.textContent = data.error || 'การยืนยันตัวตนล้มเหลว';
      timestamp.textContent = '';
    }
  } catch (error) {
    console.error('Error:', error);
    resultContainer.className = 'error';
    resultMessage.textContent = 'เกิดข้อผิดพลาดในการเชื่อมต่อกับเซิร์ฟเวอร์';
    timestamp.textContent = '';
  }
});

// แสดงประวัติการใช้งาน
auditBtn.addEventListener('click', async () => {
  if (!currentUser) {
    alert('กรุณาลงทะเบียนก่อน');
    return;
  }
  
  try {
    const response = await fetch(`/api/audit-trail/${currentUser}`);
    const data = await response.json();
    
    auditResult.classList.remove('hidden');
    
    if (response.ok) {
      registrationTime.textContent = `เวลาลงทะเบียน: ${new Date(data.registrationTime).toLocaleString()}`;
      lastUsed.textContent = data.lastUsed 
        ? `เข้าสู่ระบบล่าสุด: ${new Date(data.lastUsed).toLocaleString()}`
        : 'ยังไม่เคยเข้าสู่ระบบ';
    } else {
      auditResult.innerHTML = `<p class="error">${data.error || 'ไม่สามารถแสดงประวัติการใช้งานได้'}</p>`;
    }
  } catch (error) {
    console.error('Error:', error);
    auditResult.innerHTML = '<p class="error">เกิดข้อผิดพลาดในการเชื่อมต่อกับเซิร์ฟเวอร์</p>';
  }
});
```

### ส่วนที่ 3: การทดสอบระบบ

1. **เริ่มต้นเซิร์ฟเวอร์**
   ```bash
   node server.js
   ```

2. **เปิดเว็บเบราว์เซอร์** และไปที่ http://localhost:3000

3. **ทดสอบการใช้งาน**:
   - ลงทะเบียนด้วยรหัสผู้ใช้
   - สแกน QR Code ด้วยแอปพลิเคชัน Authenticator (เช่น Google Authenticator, Microsoft Authenticator, หรือ Authy)
   - ป้อนรหัส TOTP ที่ได้จากแอปพลิเคชัน
   - ทดสอบกรณีป้อนรหัสผิด
   - ตรวจสอบระบบล็อกบัญชีหลังจากป้อนรหัสผิด 5 ครั้ง
   - ตรวจสอบประวัติการใช้งาน

4. **การทดสอบความปลอดภัย**:
   - ทดสอบการบุกรุกด้วยการพยายามเดารหัส TOTP
   - ทดสอบการโจมตีแบบ Replay Attack โดยการใช้รหัส TOTP เดิมซ้ำ
   - ทดสอบด้านเวลา โดยการปรับเวลาในเครื่องและสังเกตผล
   - ตรวจสอบความปลอดภัยของการสื่อสารระหว่างไคลเอนต์และเซิร์ฟเวอร์
   - วิเคราะห์ log ในเซิร์ฟเวอร์เพื่อดูประวัติการเข้าใช้งาน

5. **การวิเคราะห์ ISO/IEC 27001 และ PDPA**:
   - ประเมินความสอดคล้องกับมาตรฐาน ISO/IEC 27001
   - ตรวจสอบการปฏิบัติตาม PDPA ในส่วนของการจัดเก็บข้อมูลส่วนบุคคล
   - ระบุช่องว่างที่ต้องปรับปรุงเพื่อให้เป็นไปตามมาตรฐาน

## ทรัพยากรและข้อมูลอ้างอิง

การพัฒนาระบบ AAA ที่มีความปลอดภัยสูงนั้น ควรศึกษาข้อมูลเพิ่มเติมจากแหล่งต่อไปนี้:

1. **มาตรฐานและแนวปฏิบัติ:**
   - ISO/IEC 27001 - ระบบบริหารจัดการความมั่นคงปลอดภัยสารสนเทศ
   - NIST SP 800-63B - Digital Identity Guidelines: Authentication and Lifecycle Management
   - ACSC Essential Eight - มาตรการความปลอดภัยสำคัญ 8 ประการ
   - PDPA - พระราชบัญญัติคุ้มครองข้อมูลส่วนบุคคล พ.ศ. 2562

2. **เอกสารทางเทคนิค:**
   - [RFC 6238](https://tools.ietf.org/html/rfc6238) - TOTP: Time-Based One-Time Password Algorithm
   - [Speakeasy Documentation](https://github.com/speakeasyjs/speakeasy) - เอกสารประกอบการใช้งาน Speakeasy
   - [Express.js Documentation](https://expressjs.com/) - คู่มือการใช้งาน Express.js

3. **แหล่งข้อมูลเพิ่มเติม:**
   - [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
   - [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
   - [ThaiCERT](https://www.thaicert.or.th/) - ศูนย์ประสานการรักษาความมั่นคงปลอดภัยระบบคอมพิวเตอร์ประเทศไทย

## กำหนดการส่งงาน

- นำเสนอความคืบหน้า (Progress Check): สัปดาห์ที่ 12 (10 นาที/กลุ่ม)
- ส่งรายงานและซอร์สโค้ด: ก่อนวันนำเสนองาน
- นำเสนอผลงานสมบูรณ์: สัปดาห์ที่ 13 (15 นาที/กลุ่ม)

## งานที่ต้องทำ

ให้นักศึกษาแต่ละกลุ่ม (2 คน) ดำเนินการดังนี้:

1. **ศึกษาและทดลองโค้ดที่ให้มา** - ทำความเข้าใจกลไกการทำงานของระบบ TOTP และการใช้งาน speakeasy

2. **วิเคราะห์ระบบความปลอดภัย** - ระบุจุดแข็งและจุดอ่อนของระบบที่พัฒนาขึ้น

3. **พัฒนาต่อยอด** - เพิ่มคุณสมบัติความปลอดภัยอย่างน้อย 2 อย่างจากรายการต่อไปนี้:
   - ระบบการยืนยันตัวตนด้วยปัจจัยที่ 3 (เช่น การส่ง SMS, Email)
   - ระบบการตรวจจับความผิดปกติ (เช่น การเข้าสู่ระบบจาก IP ที่ไม่เคยใช้)
   - ระบบ Role-Based Access Control (RBAC)
   - ระบบแบ็คอัพและกู้คืนรหัสลับ (Backup & Recovery)
   - การเข้ารหัสข้อมูลลับในฐานข้อมูล
