# การสร้างส่วน Frontend สำหรับแอปพลิเคชัน Flip-Card ด้วย React

หลังจากที่เราได้สร้างส่วน Backend เสร็จเรียบร้อยแล้ว ในบทเรียนนี้เราจะมาพัฒนาส่วน Frontend ของแอปพลิเคชัน Flip-Card โดยใช้ React จากแบบตัวอย่างที่เราได้เห็นในไฟล์อ้างอิง เพื่อให้ผู้ใช้สามารถสร้างและเล่นกับการ์ดได้อย่างมีประสิทธิภาพ

## ขั้นตอนที่ 1: การตั้งค่า Frontend Project

### 1.1 สร้างโปรเจค React ด้วย Create React App

เริ่มต้นโดยการสร้างโปรเจค React ใหม่ในโฟลเดอร์ `frontend` ของโปรเจค:

```bash
npx create-react-app frontend
cd frontend
```

### 1.2 ติดตั้ง Dependencies

ติดตั้งไลบรารีที่จำเป็นสำหรับโปรเจค:

```bash
npm install axios react-router-dom formik yup styled-components react-icons
```

รายละเอียดแพ็กเกจ:

- `axios`: สำหรับทำการเรียก API
- `react-router-dom`: สำหรับการจัดการเส้นทาง (routing)
- `formik` และ `yup`: สำหรับการจัดการฟอร์มและการตรวจสอบความถูกต้อง
- `styled-components`: สำหรับการเขียน CSS ในรูปแบบ JavaScript
- `react-icons`: สำหรับใช้ไอคอนต่างๆ

### 1.3 สร้างโครงสร้างโฟลเดอร์

สร้างโครงสร้างโฟลเดอร์ภายใน `src` ดังนี้:

```bash
mkdir -p src/components/common
mkdir -p src/components/auth
mkdir -p src/components/creator
mkdir -p src/components/player
mkdir -p src/pages
mkdir -p src/context
mkdir -p src/api
mkdir -p src/styles
mkdir -p src/utils
```

## ขั้นตอนที่ 2: การตั้งค่าพื้นฐาน

### 2.1 สร้างไฟล์สำหรับการตั้งค่า API (`src/api/axios.js`)

```javascript
import axios from 'axios';

// สร้างอินสแตนซ์ Axios
const API = axios.create({
  baseURL: 'http://localhost:5000/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

// เพิ่ม Interceptor สำหรับการแนบ Token ในทุกคำขอ
API.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

export default API;
```

### 2.2 สร้าง Context สำหรับการจัดการสถานะผู้ใช้ (`src/context/AuthContext.js`)

```javascript
import React, { createContext, useState, useEffect } from 'react';
import API from '../api/axios';

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // ตรวจสอบสถานะการล็อกอินเมื่อโหลดแอปพลิเคชัน
  useEffect(() => {
    const checkLoggedIn = async () => {
      setLoading(true);
      try {
        const token = localStorage.getItem('token');
        if (!token) {
          setLoading(false);
          return;
        }

        const response = await API.get('/users/me');
        setUser(response.data.user);
      } catch (error) {
        localStorage.removeItem('token');
        setError('เซสชันหมดอายุ กรุณาล็อกอินใหม่');
      } finally {
        setLoading(false);
      }
    };

    checkLoggedIn();
  }, []);

  // ฟังก์ชันสำหรับการลงทะเบียน
  const register = async (userData) => {
    setLoading(true);
    try {
      const response = await API.post('/users/register', userData);
      localStorage.setItem('token', response.data.token);
      setUser(response.data.user);
      setError(null);
      return true;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการลงทะเบียน'
      );
      return false;
    } finally {
      setLoading(false);
    }
  };

  // ฟังก์ชันสำหรับการล็อกอิน
  const login = async (userData) => {
    setLoading(true);
    try {
      const response = await API.post('/users/login', userData);
      localStorage.setItem('token', response.data.token);
      setUser(response.data.user);
      setError(null);
      return true;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการล็อกอิน'
      );
      return false;
    } finally {
      setLoading(false);
    }
  };

  // ฟังก์ชันสำหรับการล็อกเอาท์
  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        loading,
        error,
        register,
        login,
        logout,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
};
```

### 2.3 สร้าง Context สำหรับการจัดการสถานะการ์ด (`src/context/CardContext.js`)

```javascript
import React, { createContext, useState, useCallback, useContext } from 'react';
import API from '../api/axios';
import { AuthContext } from './AuthContext';

export const CardContext = createContext();

export const CardProvider = ({ children }) => {
  const { user } = useContext(AuthContext);
  const [categories, setCategories] = useState([]);
  const [currentCategory, setCurrentCategory] = useState(null);
  const [cards, setCards] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // ฟังก์ชันสำหรับการดึงหมวดหมู่ทั้งหมด
  const fetchCategories = useCallback(async () => {
    if (!user) return;
    
    setLoading(true);
    try {
      const response = await API.get('/categories');
      setCategories(response.data.data);
      setError(null);
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการดึงข้อมูลหมวดหมู่'
      );
    } finally {
      setLoading(false);
    }
  }, [user]);

  // ฟังก์ชันสำหรับการสร้างหมวดหมู่ใหม่
  const createCategory = useCallback(async (categoryData) => {
    setLoading(true);
    try {
      const response = await API.post('/categories', categoryData);
      setCategories([...categories, response.data.data]);
      setError(null);
      return response.data.data;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการสร้างหมวดหมู่'
      );
      return null;
    } finally {
      setLoading(false);
    }
  }, [categories]);

  // ฟังก์ชันสำหรับการอัปเดตหมวดหมู่
  const updateCategory = useCallback(async (id, categoryData) => {
    setLoading(true);
    try {
      const response = await API.put(`/categories/${id}`, categoryData);
      setCategories(
        categories.map((category) =>
          category._id === id ? response.data.data : category
        )
      );
      setError(null);
      return response.data.data;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการอัปเดตหมวดหมู่'
      );
      return null;
    } finally {
      setLoading(false);
    }
  }, [categories]);

  // ฟังก์ชันสำหรับการลบหมวดหมู่
  const deleteCategory = useCallback(async (id) => {
    setLoading(true);
    try {
      await API.delete(`/categories/${id}`);
      setCategories(categories.filter((category) => category._id !== id));
      setError(null);
      return true;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการลบหมวดหมู่'
      );
      return false;
    } finally {
      setLoading(false);
    }
  }, [categories]);

  // ฟังก์ชันสำหรับการดึงการ์ดในหมวดหมู่
  const fetchCards = useCallback(async (categoryId) => {
    if (!categoryId) return;
    
    setLoading(true);
    try {
      const response = await API.get(`/categories/${categoryId}/cards`);
      setCards(response.data.data);
      
      // ดึงข้อมูลหมวดหมู่เพื่อตั้งค่า currentCategory
      const categoryResponse = await API.get(`/categories/${categoryId}`);
      setCurrentCategory(categoryResponse.data.data);
      
      setError(null);
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการดึงข้อมูลการ์ด'
      );
    } finally {
      setLoading(false);
    }
  }, []);

  // ฟังก์ชันสำหรับการสร้างการ์ดใหม่
  const createCard = useCallback(async (categoryId, cardData) => {
    setLoading(true);
    try {
      const response = await API.post(
        `/categories/${categoryId}/cards`,
        cardData
      );
      setCards([...cards, response.data.data]);
      setError(null);
      return response.data.data;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการสร้างการ์ด'
      );
      return null;
    } finally {
      setLoading(false);
    }
  }, [cards]);

  // ฟังก์ชันสำหรับการอัปเดตการ์ด
  const updateCard = useCallback(async (id, cardData) => {
    setLoading(true);
    try {
      const response = await API.put(`/cards/${id}`, cardData);
      setCards(
        cards.map((card) => (card._id === id ? response.data.data : card))
      );
      setError(null);
      return response.data.data;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการอัปเดตการ์ด'
      );
      return null;
    } finally {
      setLoading(false);
    }
  }, [cards]);

  // ฟังก์ชันสำหรับการลบการ์ด
  const deleteCard = useCallback(async (id) => {
    setLoading(true);
    try {
      await API.delete(`/cards/${id}`);
      setCards(cards.filter((card) => card._id !== id));
      setError(null);
      return true;
    } catch (error) {
      setError(
        error.response?.data?.message || 'เกิดข้อผิดพลาดในการลบการ์ด'
      );
      return false;
    } finally {
      setLoading(false);
    }
  }, [cards]);

  return (
    <CardContext.Provider
      value={{
        categories,
        currentCategory,
        cards,
        loading,
        error,
        fetchCategories,
        createCategory,
        updateCategory,
        deleteCategory,
        fetchCards,
        createCard,
        updateCard,
        deleteCard,
      }}
    >
      {children}
    </CardContext.Provider>
  );
};
```

## ขั้นตอนที่ 3: การสร้างคอมโพเนนต์พื้นฐาน

### 3.1 สร้างคอมโพเนนต์พื้นฐาน (`src/components/common/Button.js`)

```javascript
import styled from 'styled-components';

const Button = styled.button`
  background-color: ${(props) => (props.secondary ? '#2980b9' : '#3498db')};
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s;
  margin: ${(props) => props.margin || '0'};

  &:hover {
    background-color: ${(props) => (props.secondary ? '#2473a6' : '#2980b9')};
  }

  &:disabled {
    background-color: #95a5a6;
    cursor: not-allowed;
  }

  &.active {
    background-color: #2ecc71;
  }

  &.danger {
    background-color: #e74c3c;
    &:hover {
      background-color: #c0392b;
    }
  }
`;

export default Button;
```

### 3.2 สร้างคอมโพเนนต์การ์ด (`src/components/common/Card.js`)

```javascript
import styled from 'styled-components';

export const CardContainer = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
`;

export const Header = styled.header`
  text-align: center;
  margin-bottom: 20px;
  padding: 20px;
  background-color: #3498db;
  color: white;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

  h1 {
    margin: 0;
    font-size: 2rem;
  }

  .subtitle {
    font-size: 1.2rem;
    margin-top: 10px;
    opacity: 0.9;
  }
`;

export const Instructions = styled.div`
  text-align: center;
  margin-bottom: 20px;
  padding: 10px;
  background-color: #ecf0f1;
  border-radius: 5px;
`;

export const Controls = styled.div`
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  gap: 10px;
`;

export const CardsContainer = styled.div`
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
`;

export const CardOuter = styled.div`
  perspective: 1000px;
  width: 500px;
  height: 300px;
  margin-bottom: 20px;

  @media (max-width: 768px) {
    width: 320px;
    height: 250px;
  }
`;

export const CardInner = styled.div`
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  transform-style: preserve-3d;
  cursor: pointer;
  transform: ${(props) => (props.flipped ? 'rotateY(180deg)' : 'rotateY(0)')};
`;

export const CardFace = styled.div`
  position: absolute;
  width: 100%;
  height: 100%;
  -webkit-backface-visibility: hidden;
  backface-visibility: hidden;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  padding: 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: center;
`;

export const CardFront = styled(CardFace)`
  background-color: #3498db;
  color: white;
`;

export const CardBack = styled(CardFace)`
  background-color: white;
  color: #2c3e50;
  transform: rotateY(180deg);
`;

export const CardNumber = styled.div`
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: 0.9rem;
  opacity: 0.8;
`;

export const CardContent = styled.div`
  font-size: 1.2rem;
  text-align: center;

  @media (max-width: 768px) {
    font-size: 1rem;
  }

  ul {
    text-align: left;
    padding-left: 20px;
    list-style-type: none;
  }

  li {
    margin-bottom: 10px;
    position: relative;

    &::before {
      content: "✅";
      margin-right: 8px;
    }
  }
`;

export const ProgressContainer = styled.div`
  text-align: center;
  margin-top: 20px;
`;

export const ProgressText = styled.div`
  font-size: 1.1rem;
  margin-bottom: 10px;
`;

export const ProgressBar = styled.div`
  width: 100%;
  height: 20px;
  background-color: #ddd;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 10px;
`;

export const ProgressFill = styled.div`
  height: 100%;
  background-color: #2ecc71;
  width: ${(props) => props.percentage}%;
  transition: width 0.3s ease;
`;
```

### 3.3 สร้างคอมโพเนนต์ฟอร์ม (`src/components/common/FormElements.js`)

```javascript
import styled from 'styled-components';

export const FormGroup = styled.div`
  margin-bottom: 20px;
`;

export const Label = styled.label`
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
`;

export const Input = styled.input`
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

export const TextArea = styled.textarea`
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  min-height: 100px;
  resize: vertical;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

export const Select = styled.select`
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

export const Checkbox = styled.div`
  display: flex;
  align-items: center;
  
  input {
    margin-right: 10px;
  }
`;

export const ErrorMessage = styled.div`
  color: #e74c3c;
  font-size: 0.875rem;
  margin-top: 5px;
`;

export const Form = styled.form`
  max-width: 600px;
  margin: 0 auto;
`;
```

### 3.4 สร้างคอมโพเนนต์เลย์เอาต์ (`src/components/common/Layout.js`)

```javascript
import React, { useContext } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import styled from 'styled-components';
import { AuthContext } from '../../context/AuthContext';
import Button from './Button';

const LayoutContainer = styled.div`
  display: flex;
  flex-direction: column;
  min-height: 100vh;
`;

const Header = styled.header`
  background-color: #2c3e50;
  color: white;
  padding: 1rem;
`;

const Nav = styled.nav`
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
`;

const Logo = styled(Link)`
  color: white;
  text-decoration: none;
  font-size: 1.5rem;
  font-weight: bold;
`;

const NavLinks = styled.div`
  display: flex;
  gap: 20px;
`;

const StyledLink = styled(Link)`
  color: white;
  text-decoration: none;
  padding: 5px 10px;
  border-radius: 4px;
  
  &:hover {
    background-color: rgba(255, 255, 255, 0.1);
  }

  &.active {
    background-color: #3498db;
  }
`;

const Main = styled.main`
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
`;

const Footer = styled.footer`
  background-color: #2c3e50;
  color: white;
  text-align: center;
  padding: 1rem;
`;

const Layout = ({ children }) => {
  const { user, logout } = useContext(AuthContext);
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/login');
  };

  return (
    <LayoutContainer>
      <Header>
        <Nav>
          <Logo to="/">Flip Card App</Logo>
          <NavLinks>
            {user ? (
              <>
                <StyledLink to="/creator">Creator Mode</StyledLink>
                <StyledLink to="/player">Player Mode</StyledLink>
                <Button onClick={handleLogout}>ออกจากระบบ</Button>
              </>
            ) : (
              <>
                <StyledLink to="/login">เข้าสู่ระบบ</StyledLink>
                <StyledLink to="/register">ลงทะเบียน</StyledLink>
              </>
            )}
          </NavLinks>
        </Nav>
      </Header>
      <Main>{children}</Main>
      <Footer>
        <p>&copy; {new Date().getFullYear()} - Flip Card App | DTI132</p>
      </Footer>
    </LayoutContainer>
  );
};

export default Layout;
```

## ขั้นตอนที่ 4: การสร้างหน้าการยืนยันตัวตน

### 4.1 สร้างหน้าลงทะเบียน (`src/components/auth/Register.js`)

```javascript
import React, { useContext, useState } from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';
import { Link, useNavigate } from 'react-router-dom';
import styled from 'styled-components';
import { AuthContext } from '../../context/AuthContext';
import Button from '../common/Button';
import { FormGroup, Label, ErrorMessage as StyledError } from '../common/FormElements';

const StyledForm = styled.div`
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
`;

const Title = styled.h2`
  text-align: center;
  margin-bottom: 20px;
  color: #2c3e50;
`;

const StyledField = styled(Field)`
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

const LoginLink = styled.div`
  text-align: center;
  margin-top: 20px;
  font-size: 0.9rem;
  
  a {
    color: #3498db;
    text-decoration: none;
    
    &:hover {
      text-decoration: underline;
    }
  }
`;

const Alert = styled.div`
  padding: 10px;
  background-color: ${props => props.success ? '#d4edda' : '#f8d7da'};
  color: ${props => props.success ? '#155724' : '#721c24'};
  border-radius: 4px;
  margin-bottom: 20px;
  text-align: center;
`;

// สคีมาการตรวจสอบความถูกต้อง
const validationSchema = Yup.object({
  username: Yup.string().required('กรุณาระบุชื่อผู้ใช้'),
  email: Yup.string().email('รูปแบบอีเมลไม่ถูกต้อง').required('กรุณาระบุอีเมล'),
  password: Yup.string().min(8, 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร').required('กรุณาระบุรหัสผ่าน'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password'), null], 'รหัสผ่านไม่ตรงกัน')
    .required('กรุณายืนยันรหัสผ่าน'),
});

const Register = () => {
  const { register, error } = useContext(AuthContext);
  const [successMessage, setSuccessMessage] = useState('');
  const navigate = useNavigate();

  const handleSubmit = async (values, { setSubmitting }) => {
    const { username, email, password } = values;
    const success = await register({ username, email, password });
    
    if (success) {
      setSuccessMessage('ลงทะเบียนสำเร็จ กำลังนำคุณไปยังหน้าหลัก...');
      setTimeout(() => {
        navigate('/');
      }, 2000);
    }
    
    setSubmitting(false);
  };

  return (
    <div>
      <Title>ลงทะเบียน</Title>
      
      {error && <Alert>{error}</Alert>}
      {successMessage && <Alert success>{successMessage}</Alert>}
      
      <StyledForm>
        <Formik
          initialValues={{ username: '', email: '', password: '', confirmPassword: '' }}
          validationSchema={validationSchema}
          onSubmit={handleSubmit}
        >
          {({ isSubmitting }) => (
            <Form>
              <FormGroup>
                <Label htmlFor="username">ชื่อผู้ใช้</Label>
                <StyledField type="text" id="username" name="username" />
                <ErrorMessage name="username" component={StyledError} />
              </FormGroup>

              <FormGroup>
                <Label htmlFor="email">อีเมล</Label>
                <StyledField type="email" id="email" name="email" />
                <ErrorMessage name="email" component={StyledError} />
              </FormGroup>

              <FormGroup>
                <Label htmlFor="password">รหัสผ่าน</Label>
                <StyledField type="password" id="password" name="password" />
                <ErrorMessage name="password" component={StyledError} />
              </FormGroup>

              <FormGroup>
                <Label htmlFor="confirmPassword">ยืนยันรหัสผ่าน</Label>
                <StyledField type="password" id="confirmPassword" name="confirmPassword" />
                <ErrorMessage name="confirmPassword" component={StyledError} />
              </FormGroup>


              <Button type="submit" disabled={isSubmitting} style={{ width: '100%' }}>
                {isSubmitting ? 'กำลังลงทะเบียน...' : 'ลงทะเบียน'}
              </Button>
            </Form>
          )}
        </Formik>
        
        <LoginLink>
          มีบัญชีอยู่แล้ว? <Link to="/login">เข้าสู่ระบบ</Link>
        </LoginLink>
      </StyledForm>
    </div>
  );
};

export default Register;
```

### 4.2 สร้างหน้าล็อกอิน (`src/components/auth/Login.js`)

```javascript
import React, { useContext, useState } from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';
import { Link, useNavigate } from 'react-router-dom';
import styled from 'styled-components';
import { AuthContext } from '../../context/AuthContext';
import Button from '../common/Button';
import { FormGroup, Label, ErrorMessage as StyledError } from '../common/FormElements';

const StyledForm = styled.div`
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
`;

const Title = styled.h2`
  text-align: center;
  margin-bottom: 20px;
  color: #2c3e50;
`;

const StyledField = styled(Field)`
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

const RegisterLink = styled.div`
  text-align: center;
  margin-top: 20px;
  font-size: 0.9rem;
  
  a {
    color: #3498db;
    text-decoration: none;
    
    &:hover {
      text-decoration: underline;
    }
  }
`;

const Alert = styled.div`
  padding: 10px;
  background-color: ${props => props.success ? '#d4edda' : '#f8d7da'};
  color: ${props => props.success ? '#155724' : '#721c24'};
  border-radius: 4px;
  margin-bottom: 20px;
  text-align: center;
`;

// สคีมาการตรวจสอบความถูกต้อง
const validationSchema = Yup.object({
  email: Yup.string().email('รูปแบบอีเมลไม่ถูกต้อง').required('กรุณาระบุอีเมล'),
  password: Yup.string().required('กรุณาระบุรหัสผ่าน'),
});

const Login = () => {
  const { login, error } = useContext(AuthContext);
  const [successMessage, setSuccessMessage] = useState('');
  const navigate = useNavigate();

  const handleSubmit = async (values, { setSubmitting }) => {
    const success = await login(values);
    
    if (success) {
      setSuccessMessage('เข้าสู่ระบบสำเร็จ กำลังนำคุณไปยังหน้าหลัก...');
      setTimeout(() => {
        navigate('/');
      }, 2000);
    }
    
    setSubmitting(false);
  };

  return (
    <div>
      <Title>เข้าสู่ระบบ</Title>
      
      {error && <Alert>{error}</Alert>}
      {successMessage && <Alert success>{successMessage}</Alert>}
      
      <StyledForm>
        <Formik
          initialValues={{ email: '', password: '' }}
          validationSchema={validationSchema}
          onSubmit={handleSubmit}
        >
          {({ isSubmitting }) => (
            <Form>
              <FormGroup>
                <Label htmlFor="email">อีเมล</Label>
                <StyledField type="email" id="email" name="email" />
                <ErrorMessage name="email" component={StyledError} />
              </FormGroup>

              <FormGroup>
                <Label htmlFor="password">รหัสผ่าน</Label>
                <StyledField type="password" id="password" name="password" />
                <ErrorMessage name="password" component={StyledError} />
              </FormGroup>

              <Button type="submit" disabled={isSubmitting} style={{ width: '100%' }}>
                {isSubmitting ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
              </Button>
            </Form>
          )}
        </Formik>
        
        <RegisterLink>
          ยังไม่มีบัญชี? <Link to="/register">ลงทะเบียน</Link>
        </RegisterLink>
      </StyledForm>
    </div>
  );
};

export default Login;
```

## ขั้นตอนที่ 5: การสร้างโหมดผู้สร้าง (Creator Mode)

### 5.1 สร้างหน้าผู้สร้าง (`src/components/creator/CreatorDashboard.js`)

```javascript
import React, { useContext, useEffect, useState } from 'react';
import styled from 'styled-components';
import { CardContext } from '../../context/CardContext';
import Button from '../common/Button';
import CategoryForm from './CategoryForm';
import CategoryList from './CategoryList';
import CardForm from './CardForm';
import CardList from './CardList';

const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
`;

const Title = styled.h1`
  color: #2c3e50;
  text-align: center;
  margin-bottom: 20px;
`;

const Subtitle = styled.h2`
  color: #34495e;
  margin: 30px 0 15px;
`;

const TwoColumnLayout = styled.div`
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 20px;
  
  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }
`;

const Panel = styled.div`
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
`;

const Alert = styled.div`
  padding: 10px;
  background-color: ${props => props.success ? '#d4edda' : '#f8d7da'};
  color: ${props => props.success ? '#155724' : '#721c24'};
  border-radius: 4px;
  margin-bottom: 20px;
  text-align: center;
`;

const CreatorDashboard = () => {
  const { 
    categories, 
    currentCategory,
    cards, 
    loading, 
    error, 
    fetchCategories,
    fetchCards
  } = useContext(CardContext);
  
  const [showCategoryForm, setShowCategoryForm] = useState(false);
  const [showCardForm, setShowCardForm] = useState(false);
  const [editingCategory, setEditingCategory] = useState(null);
  const [editingCard, setEditingCard] = useState(null);

  useEffect(() => {
    fetchCategories();
  }, [fetchCategories]);

  const handleCategorySelect = (categoryId) => {
    fetchCards(categoryId);
    setShowCardForm(false);
    setEditingCard(null);
  };

  const handleEditCategory = (category) => {
    setEditingCategory(category);
    setShowCategoryForm(true);
  };

  const handleEditCard = (card) => {
    setEditingCard(card);
    setShowCardForm(true);
  };

  const handleCategoryFormClose = () => {
    setShowCategoryForm(false);
    setEditingCategory(null);
  };

  const handleCardFormClose = () => {
    setShowCardForm(false);
    setEditingCard(null);
  };

  return (
    <Container>
      <Title>โหมดผู้สร้าง (Creator Mode)</Title>
      
      {error && <Alert>{error}</Alert>}
      
      <TwoColumnLayout>
        <Panel>
          <Subtitle>หมวดหมู่การ์ด</Subtitle>
          
          {!showCategoryForm ? (
            <Button 
              onClick={() => setShowCategoryForm(true)} 
              margin="0 0 20px 0"
            >
              + สร้างหมวดหมู่ใหม่
            </Button>
          ) : (
            <CategoryForm 
              onClose={handleCategoryFormClose} 
              editingCategory={editingCategory} 
            />
          )}
          
          <CategoryList 
            categories={categories} 
            onCategorySelect={handleCategorySelect}
            onEditCategory={handleEditCategory}
            currentCategoryId={currentCategory?._id}
          />
        </Panel>
        
        <Panel>
          <Subtitle>
            {currentCategory ? `การ์ดในหมวดหมู่: ${currentCategory.name}` : 'เลือกหมวดหมู่เพื่อดูการ์ด'}
          </Subtitle>
          
          {currentCategory && !showCardForm ? (
            <Button 
              onClick={() => setShowCardForm(true)} 
              margin="0 0 20px 0"
            >
              + สร้างการ์ดใหม่
            </Button>
          ) : null}
          
          {showCardForm && currentCategory ? (
            <CardForm 
              categoryId={currentCategory._id} 
              onClose={handleCardFormClose}
              editingCard={editingCard}
            />
          ) : null}
          
          {currentCategory ? (
            <CardList 
              cards={cards} 
              onEditCard={handleEditCard}
            />
          ) : (
            <p>กรุณาเลือกหมวดหมู่จากรายการด้านซ้าย</p>
          )}
        </Panel>
      </TwoColumnLayout>
    </Container>
  );
};

export default CreatorDashboard;
```

### 5.2 สร้างคอมโพเนนต์ฟอร์มหมวดหมู่ (`src/components/creator/CategoryForm.js`)

```javascript
import React, { useContext } from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';
import styled from 'styled-components';
import { CardContext } from '../../context/CardContext';
import Button from '../common/Button';
import { FormGroup, Label, ErrorMessage as StyledError } from '../common/FormElements';

const FormContainer = styled.div`
  margin-bottom: 20px;
  padding: 15px;
  background-color: #ecf0f1;
  border-radius: 8px;
`;

const FormTitle = styled.h3`
  margin-top: 0;
  margin-bottom: 15px;
  color: #2c3e50;
`;

const ButtonGroup = styled.div`
  display: flex;
  gap: 10px;
  margin-top: 15px;
`;

const StyledField = styled(Field)`
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

// สคีมาการตรวจสอบความถูกต้อง
const validationSchema = Yup.object({
  name: Yup.string().required('กรุณาระบุชื่อหมวดหมู่'),
  description: Yup.string(),
  isPublic: Yup.boolean(),
});

const CategoryForm = ({ onClose, editingCategory }) => {
  const { createCategory, updateCategory } = useContext(CardContext);

  const initialValues = editingCategory 
    ? { 
        name: editingCategory.name, 
        description: editingCategory.description || '', 
        isPublic: editingCategory.isPublic || false 
      }
    : { name: '', description: '', isPublic: false };

  const handleSubmit = async (values, { setSubmitting, resetForm }) => {
    if (editingCategory) {
      await updateCategory(editingCategory._id, values);
    } else {
      await createCategory(values);
    }
    
    setSubmitting(false);
    resetForm();
    onClose();
  };

  return (
    <FormContainer>
      <FormTitle>{editingCategory ? 'แก้ไขหมวดหมู่' : 'สร้างหมวดหมู่ใหม่'}</FormTitle>
      
      <Formik
        initialValues={initialValues}
        validationSchema={validationSchema}
        onSubmit={handleSubmit}
      >
        {({ isSubmitting }) => (
          <Form>
            <FormGroup>
              <Label htmlFor="name">ชื่อหมวดหมู่</Label>
              <StyledField type="text" id="name" name="name" />
              <ErrorMessage name="name" component={StyledError} />
            </FormGroup>

            <FormGroup>
              <Label htmlFor="description">คำอธิบาย</Label>
              <StyledField as="textarea" id="description" name="description" rows="3" />
              <ErrorMessage name="description" component={StyledError} />
            </FormGroup>

            <FormGroup>
              <Label>
                <Field type="checkbox" name="isPublic" />
                {' '}เผยแพร่เป็นสาธารณะ
              </Label>
            </FormGroup>

            <ButtonGroup>
              <Button type="submit" disabled={isSubmitting}>
                {isSubmitting ? 'กำลังบันทึก...' : (editingCategory ? 'บันทึกการแก้ไข' : 'สร้างหมวดหมู่')}
              </Button>
              <Button type="button" secondary onClick={onClose}>
                ยกเลิก
              </Button>
            </ButtonGroup>
          </Form>
        )}
      </Formik>
    </FormContainer>
  );
};

export default CategoryForm;
```

### 5.3 สร้างคอมโพเนนต์รายการหมวดหมู่ (`src/components/creator/CategoryList.js`)

```javascript
import React, { useContext } from 'react';
import styled from 'styled-components';
import { FaEdit, FaTrash } from 'react-icons/fa';
import { CardContext } from '../../context/CardContext';

const List = styled.ul`
  list-style: none;
  padding: 0;
  margin: 0;
`;

const ListItem = styled.li`
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  cursor: pointer;
  background-color: ${props => props.active ? '#e3f2fd' : 'white'};
  transition: all 0.2s;
  
  &:hover {
    background-color: ${props => props.active ? '#bbdefb' : '#f5f5f5'};
  }
`;

const CategoryHeader = styled.div`
  display: flex;
  justify-content: space-between;
  align-items: center;
`;

const CategoryName = styled.h3`
  margin: 0;
  font-size: 1rem;
  color: #2c3e50;
`;

const ActionButtons = styled.div`
  display: flex;
  gap: 10px;
`;

const ActionButton = styled.button`
  background: none;
  border: none;
  cursor: pointer;
  color: #7f8c8d;
  padding: 5px;
  
  &:hover {
    color: ${props => props.delete ? '#e74c3c' : '#3498db'};
  }
`;

const CategoryDescription = styled.p`
  margin: 5px 0 0;
  font-size: 0.9rem;
  color: #7f8c8d;
`;

const PublicBadge = styled.span`
  background-color: #2ecc71;
  color: white;
  font-size: 0.7rem;
  padding: 2px 6px;
  border-radius: 10px;
  margin-left: 10px;
`;

const EmptyMessage = styled.p`
  color: #7f8c8d;
  text-align: center;
  font-style: italic;
`;

const CategoryList = ({ categories, onCategorySelect, onEditCategory, currentCategoryId }) => {
  const { deleteCategory } = useContext(CardContext);

  const handleDelete = async (e, categoryId) => {
    e.stopPropagation();
    if (window.confirm('คุณแน่ใจหรือไม่ว่าต้องการลบหมวดหมู่นี้? การ์ดทั้งหมดในหมวดหมู่นี้จะถูกลบด้วย')) {
      await deleteCategory(categoryId);
    }
  };

  const handleEdit = (e, category) => {
    e.stopPropagation();
    onEditCategory(category);
  };

  if (!categories || categories.length === 0) {
    return <EmptyMessage>ยังไม่มีหมวดหมู่ กรุณาสร้างหมวดหมู่ใหม่</EmptyMessage>;
  }

  return (
    <List>
      {categories.map(category => (
        <ListItem 
          key={category._id} 
          onClick={() => onCategorySelect(category._id)}
          active={category._id === currentCategoryId}
        >
          <CategoryHeader>
            <CategoryName>
              {category.name}
              {category.isPublic && <PublicBadge>สาธารณะ</PublicBadge>}
            </CategoryName>
            <ActionButtons>
              <ActionButton onClick={(e) => handleEdit(e, category)}>
                <FaEdit />
              </ActionButton>
              <ActionButton delete onClick={(e) => handleDelete(e, category._id)}>
                <FaTrash />
              </ActionButton>
            </ActionButtons>
          </CategoryHeader>
          {category.description && (
            <CategoryDescription>{category.description}</CategoryDescription>
          )}
        </ListItem>
      ))}
    </List>
  );
};

export default CategoryList;
```

### 5.4 สร้างคอมโพเนนต์ฟอร์มการ์ด (`src/components/creator/CardForm.js`)

```javascript
import React, { useContext, useState } from 'react';
import { Formik, Form, Field, ErrorMessage, FieldArray } from 'formik';
import * as Yup from 'yup';
import styled from 'styled-components';
import { FaPlus, FaTrash } from 'react-icons/fa';
import { CardContext } from '../../context/CardContext';
import Button from '../common/Button';
import { FormGroup, Label, ErrorMessage as StyledError } from '../common/FormElements';

const FormContainer = styled.div`
  margin-bottom: 20px;
  padding: 15px;
  background-color: #ecf0f1;
  border-radius: 8px;
`;

const FormTitle = styled.h3`
  margin-top: 0;
  margin-bottom: 15px;
  color: #2c3e50;
`;

const ButtonGroup = styled.div`
  display: flex;
  gap: 10px;
  margin-top: 15px;
`;

const StyledField = styled(Field)`
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  
  &:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }
`;

const BackItems = styled.div`
  margin-top: 10px;
`;

const BackItem = styled.div`
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
`;

const AddItemButton = styled.button`
  background: none;
  border: none;
  color: #3498db;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 0;
  font-size: 0.9rem;
  
  &:hover {
    text-decoration: underline;
  }
`;

const RemoveItemButton = styled.button`
  background: none;
  border: none;
  color: #e74c3c;
  cursor: pointer;
`;

// สคีมาการตรวจสอบความถูกต้อง
const validationSchema = Yup.object({
  front: Yup.string().required('กรุณาระบุข้อความด้านหน้าการ์ด'),
  back: Yup.array()
    .of(Yup.string().required('กรุณาระบุข้อความ'))
    .min(1, 'ต้องมีข้อความด้านหลังอย่างน้อย 1 รายการ')
    .required('กรุณาระบุข้อความด้านหลังการ์ด'),
});

const CardForm = ({ categoryId, onClose, editingCard }) => {
  const { createCard, updateCard } = useContext(CardContext);

  const initialValues = editingCard 
    ? { 
        front: editingCard.front, 
        back: editingCard.back 
      }
    : { front: '', back: [''] };

  const handleSubmit = async (values, { setSubmitting, resetForm }) => {
    if (editingCard) {
      await updateCard(editingCard._id, values);
    } else {
      await createCard(categoryId, values);
    }
    
    setSubmitting(false);
    resetForm();
    onClose();
  };

  return (
    <FormContainer>
      <FormTitle>{editingCard ? 'แก้ไขการ์ด' : 'สร้างการ์ดใหม่'}</FormTitle>
      
      <Formik
        initialValues={initialValues}
        validationSchema={validationSchema}
        onSubmit={handleSubmit}
      >
        {({ values, isSubmitting }) => (
          <Form>
            <FormGroup>
              <Label htmlFor="front">ข้อความด้านหน้าการ์ด</Label>
              <StyledField as="textarea" id="front" name="front" rows="3" />
              <ErrorMessage name="front" component={StyledError} />
              <div style={{ fontSize: '0.8rem', color: '#7f8c8d', marginTop: '5px' }}>
                สามารถใช้ HTML tag พื้นฐานได้ เช่น &lt;br&gt; สำหรับขึ้นบรรทัดใหม่
              </div>
            </FormGroup>

            <FormGroup>
              <Label>ข้อความด้านหลังการ์ด</Label>
              <ErrorMessage name="back" component={StyledError} />
              
              <FieldArray name="back">
                {({ remove, push }) => (
                  <BackItems>
                    {values.back.map((item, index) => (
                      <BackItem key={index}>
                        <StyledField 
                          name={`back.${index}`} 
                          placeholder={`รายการที่ ${index + 1}`} 
                        />
                        {values.back.length > 1 && (
                          <RemoveItemButton 
                            type="button" 
                            onClick={() => remove(index)}
                          >
                            <FaTrash />
                          </RemoveItemButton>
                        )}
                      </BackItem>
                    ))}
                    
                    <AddItemButton
                      type="button"
                      onClick={() => push('')}
                    >
                      <FaPlus /> เพิ่มรายการ
                    </AddItemButton>
                  </BackItems>
                )}
              </FieldArray>
            </FormGroup>

            <ButtonGroup>
              <Button type="submit" disabled={isSubmitting}>
                {isSubmitting ? 'กำลังบันทึก...' : (editingCard ? 'บันทึกการแก้ไข' : 'สร้างการ์ด')}
              </Button>
              <Button type="button" secondary onClick={onClose}>
                ยกเลิก
              </Button>
            </ButtonGroup>
          </Form>
        )}
      </Formik>
    </FormContainer>
  );
};

export default CardForm;
```

### 5.5 สร้างคอมโพเนนต์รายการการ์ด (`src/components/creator/CardList.js`)

```javascript
import React, { useContext, useState } from 'react';
import styled from 'styled-components';
import { FaEdit, FaTrash, FaEye } from 'react-icons/fa';
import { CardContext } from '../../context/CardContext';
import CardPreview from './CardPreview';

const List = styled.ul`
  list-style: none;
  padding: 0;
  margin: 0;
`;

const ListItem = styled.li`
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  background-color: white;
`;

const CardHeader = styled.div`
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
`;

const CardFront = styled.div`
  margin: 0;
  font-size: 1rem;
  color: #2c3e50;
`;

const ActionButtons = styled.div`
  display: flex;
  gap: 10px;
`;

const ActionButton = styled.button`
  background: none;
  border: none;
  cursor: pointer;
  color: #7f8c8d;
  padding: 5px;
  
  &:hover {
    color: ${props => {
      if (props.delete) return '#e74c3c';
      if (props.preview) return '#2ecc71';
      return '#3498db';
    }};
  }
`;

const CardBackList = styled.ul`
  margin: 10px 0 0;
  padding-left: 20px;
  color: #7f8c8d;
  font-size: 0.9rem;
  display: ${props => props.visible ? 'block' : 'none'};
`;

const ShowMoreButton = styled.button`
  background: none;
  border: none;
  color: #3498db;
  cursor: pointer;
  padding: 5px 0;
  font-size: 0.8rem;
  margin-top: 5px;
  
  &:hover {
    text-decoration: underline;
  }
`;

const EmptyMessage = styled.p`
  color: #7f8c8d;
  text-align: center;
  font-style: italic;
`;

const CardList = ({ cards, onEditCard }) => {
  const { deleteCard } = useContext(CardContext);
  const [expandedCard, setExpandedCard] = useState(null);
  const [previewCard, setPreviewCard] = useState(null);

  const handleDelete = async (cardId) => {
    if (window.confirm('คุณแน่ใจหรือไม่ว่าต้องการลบการ์ดนี้?')) {
      await deleteCard(cardId);
    }
  };

  const toggleExpand = (cardId) => {
    setExpandedCard(expandedCard === cardId ? null : cardId);
  };

  const handlePreview = (card) => {
    setPreviewCard(card);
  };

  const closePreview = () => {
    setPreviewCard(null);
  };

  if (!cards || cards.length === 0) {
    return <EmptyMessage>ยังไม่มีการ์ดในหมวดหมู่นี้ กรุณาสร้างการ์ดใหม่</EmptyMessage>;
  }

  return (
    <>
      {previewCard && (
        <CardPreview card={previewCard} onClose={closePreview} />
      )}
      
      <List>
        {cards.map(card => (
          <ListItem key={card._id}>
            <CardHeader>
              <CardFront dangerouslySetInnerHTML={{ __html: card.front }} />
              <ActionButtons>
                <ActionButton preview onClick={() => handlePreview(card)}>
                  <FaEye />
                </ActionButton>
                <ActionButton onClick={() => onEditCard(card)}>
                  <FaEdit />
                </ActionButton>
                <ActionButton delete onClick={() => handleDelete(card._id)}>
                  <FaTrash />
                </ActionButton>
              </ActionButtons>
            </CardHeader>
            
            {card.back.length > 0 && (
              <>
                <ShowMoreButton onClick={() => toggleExpand(card._id)}>
                  {expandedCard === card._id ? 'ซ่อนคำตอบ' : 'แสดงคำตอบ'}
                </ShowMoreButton>
                
                <CardBackList visible={expandedCard === card._id}>
                  {card.back.map((item, index) => (
                    <li key={index}>{item}</li>
                  ))}
                </CardBackList>
              </>

			{card.back.length > 0 && (
              <>
                <ShowMoreButton onClick={() => toggleExpand(card._id)}>
                  {expandedCard === card._id ? 'ซ่อนคำตอบ' : 'แสดงคำตอบ'}
                </ShowMoreButton>
                
                <CardBackList visible={expandedCard === card._id}>
                  {card.back.map((item, index) => (
                    <li key={index}>{item}</li>
                  ))}
                </CardBackList>
              </>
            )}
          </ListItem>
        ))}
      </List>
    </>
  );
};

export default CardList;
```

### 5.6 สร้างคอมโพเนนต์แสดงตัวอย่างการ์ด (`src/components/creator/CardPreview.js`)

```javascript
import React, { useState } from 'react';
import styled from 'styled-components';
import { FaTimes } from 'react-icons/fa';
import {
  CardOuter,
  CardInner,
  CardFront,
  CardBack,
  CardNumber,
  CardContent,
} from '../common/Card';

const ModalOverlay = styled.div`
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
`;

const ModalContent = styled.div`
  width: 90%;
  max-width: 600px;
  background-color: white;
  border-radius: 10px;
  padding: 20px;
  position: relative;
`;

const CloseButton = styled.button`
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  color: #7f8c8d;
  cursor: pointer;
  font-size: 1.2rem;
  
  &:hover {
    color: #e74c3c;
  }
`;

const Title = styled.h3`
  margin-top: 0;
  margin-bottom: 20px;
  color: #2c3e50;
`;

const Instructions = styled.p`
  margin-bottom: 20px;
  color: #7f8c8d;
`;

const CardPreview = ({ card, onClose }) => {
  const [flipped, setFlipped] = useState(false);

  const handleFlip = () => {
    setFlipped(!flipped);
  };

  return (
    <ModalOverlay>
      <ModalContent>
        <CloseButton onClick={onClose}>
          <FaTimes />
        </CloseButton>
        
        <Title>ตัวอย่างการ์ด</Title>
        <Instructions>คลิกที่การ์ดเพื่อพลิกดูด้านหลัง</Instructions>
        
        <CardOuter>
          <CardInner flipped={flipped} onClick={handleFlip}>
            <CardFront>
              <CardNumber>Preview</CardNumber>
              <CardContent dangerouslySetInnerHTML={{ __html: card.front }} />
            </CardFront>
            
            <CardBack>
              <CardNumber>Preview</CardNumber>
              <CardContent>
                <ul>
                  {card.back.map((item, index) => (
                    <li key={index}>{item}</li>
                  ))}
                </ul>
              </CardContent>
            </CardBack>
          </CardInner>
        </CardOuter>
      </ModalContent>
    </ModalOverlay>
  );
};

export default CardPreview;
```

## ขั้นตอนที่ 6: การสร้างโหมดผู้เล่น (Player Mode)

### 6.1 สร้างหน้าผู้เล่น (`src/components/player/PlayerDashboard.js`)

```javascript
import React, { useContext, useEffect, useState } from 'react';
import styled from 'styled-components';
import { CardContext } from '../../context/CardContext';
import Button from '../common/Button';
import FlipCardGame from './FlipCardGame';

const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
`;

const Title = styled.h1`
  color: #2c3e50;
  text-align: center;
  margin-bottom: 20px;
`;

const CategorySelection = styled.div`
  margin-bottom: 20px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
`;

const CategoryTitle = styled.h2`
  color: #34495e;
  margin-top: 0;
  margin-bottom: 15px;
`;

const CategoryGrid = styled.div`
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 15px;
`;

const CategoryCard = styled.div`
  padding: 15px;
  background-color: white;
  border-radius: 6px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  }
`;

const CategoryName = styled.h3`
  margin: 0 0 10px;
  color: #2c3e50;
  font-size: 1.1rem;
`;

const CategoryDescription = styled.p`
  margin: 0;
  color: #7f8c8d;
  font-size: 0.9rem;
`;

const PublicBadge = styled.span`
  background-color: #2ecc71;
  color: white;
  font-size: 0.7rem;
  padding: 2px 6px;
  border-radius: 10px;
  margin-left: 10px;
`;

const Alert = styled.div`
  padding: 10px;
  background-color: ${props => props.success ? '#d4edda' : '#f8d7da'};
  color: ${props => props.success ? '#155724' : '#721c24'};
  border-radius: 4px;
  margin-bottom: 20px;
  text-align: center;
`;

const EmptyMessage = styled.p`
  color: #7f8c8d;
  text-align: center;
  font-style: italic;
`;

const PlayerDashboard = () => {
  const { 
    categories, 
    currentCategory,
    cards, 
    loading, 
    error, 
    fetchCategories,
    fetchCards
  } = useContext(CardContext);
  
  const [selectedCategory, setSelectedCategory] = useState(null);

  useEffect(() => {
    fetchCategories();
  }, [fetchCategories]);

  const handleCategorySelect = async (categoryId) => {
    await fetchCards(categoryId);
    setSelectedCategory(categoryId);
  };

  const handleBackToCategories = () => {
    setSelectedCategory(null);
  };

  return (
    <Container>
      <Title>โหมดผู้เล่น (Player Mode)</Title>
      
      {error && <Alert>{error}</Alert>}
      
      {!selectedCategory ? (
        <CategorySelection>
          <CategoryTitle>เลือกหมวดหมู่การ์ด</CategoryTitle>
          
          {!categories || categories.length === 0 ? (
            <EmptyMessage>ยังไม่มีหมวดหมู่การ์ด กรุณาสร้างหมวดหมู่ในโหมดผู้สร้าง</EmptyMessage>
          ) : (
            <CategoryGrid>
              {categories.map(category => (
                <CategoryCard 
                  key={category._id} 
                  onClick={() => handleCategorySelect(category._id)}
                >
                  <CategoryName>
                    {category.name}
                    {category.isPublic && <PublicBadge>สาธารณะ</PublicBadge>}
                  </CategoryName>
                  {category.description && (
                    <CategoryDescription>{category.description}</CategoryDescription>
                  )}
                </CategoryCard>
              ))}
            </CategoryGrid>
          )}
        </CategorySelection>
      ) : (
        <>
          <Button onClick={handleBackToCategories} margin="0 0 20px 0">
            &larr; กลับไปยังหมวดหมู่
          </Button>
          
          {currentCategory && cards && (
            <FlipCardGame 
              category={currentCategory} 
              cards={cards} 
            />
          )}
        </>
      )}
    </Container>
  );
};

export default PlayerDashboard;
```

### 6.2 สร้างคอมโพเนนต์เกมการ์ด (`src/components/player/FlipCardGame.js`)

```javascript
import React, { useState, useEffect } from 'react';
import styled from 'styled-components';
import {
  CardContainer,
  Header,
  Instructions,
  Controls,
  CardsContainer,
  CardOuter,
  CardInner,
  CardFront,
  CardBack,
  CardNumber,
  CardContent,
  ProgressContainer,
  ProgressText,
  ProgressBar,
  ProgressFill,
} from '../common/Card';
import Button from '../common/Button';

const FlipCardGame = ({ category, cards }) => {
  const [activeCards, setActiveCards] = useState([]);
  const [flippedCards, setFlippedCards] = useState({});
  const [totalFlipped, setTotalFlipped] = useState(0);

  useEffect(() => {
    setActiveCards(cards);
    setFlippedCards({});
    setTotalFlipped(0);
  }, [cards]);

  const handleCardFlip = (cardId) => {
    if (!flippedCards[cardId]) {
      const newFlippedCards = { ...flippedCards, [cardId]: true };
      setFlippedCards(newFlippedCards);
      setTotalFlipped(Object.keys(newFlippedCards).length);
    }
  };

  const handleFlipAll = () => {
    const allFlipped = {};
    cards.forEach(card => {
      allFlipped[card._id] = true;
    });
    setFlippedCards(allFlipped);
    setTotalFlipped(cards.length);
  };

  const handleReset = () => {
    setFlippedCards({});
    setTotalFlipped(0);
  };

  const getProgressPercentage = () => {
    if (cards.length === 0) return 0;
    return (totalFlipped / cards.length) * 100;
  };

  return (
    <CardContainer>
      <Header>
        <h1>{category.name}</h1>
        <div className="subtitle">{category.description}</div>
      </Header>

      <Instructions>
        <p>คลิกที่การ์ดเพื่อพลิกดูคำตอบ ใช้ปุ่มด้านล่างเพื่อควบคุมการเล่น</p>
      </Instructions>

      <Controls>
        <Button onClick={handleFlipAll}>เปิดทุกการ์ด</Button>
        <Button onClick={handleReset}>เริ่มใหม่</Button>
      </Controls>

      <CardsContainer>
        {activeCards.map((card, index) => (
          <CardOuter key={card._id}>
            <CardInner 
              flipped={flippedCards[card._id]} 
              onClick={() => handleCardFlip(card._id)}
            >
              <CardFront>
                <CardNumber>{index + 1}/{activeCards.length}</CardNumber>
                <CardContent dangerouslySetInnerHTML={{ __html: card.front }} />
              </CardFront>
              
              <CardBack>
                <CardNumber>{index + 1}/{activeCards.length}</CardNumber>
                <CardContent>
                  <ul>
                    {card.back.map((item, i) => (
                      <li key={i}>{item}</li>
                    ))}
                  </ul>
                </CardContent>
              </CardBack>
            </CardInner>
          </CardOuter>
        ))}
      </CardsContainer>

      <ProgressContainer>
        <ProgressText>
          ความคืบหน้า: <span>{totalFlipped}/{activeCards.length}</span> การ์ด
        </ProgressText>
        <ProgressBar>
          <ProgressFill percentage={getProgressPercentage()} />
        </ProgressBar>
      </ProgressContainer>
    </CardContainer>
  );
};

export default FlipCardGame;
```

## ขั้นตอนที่ 7: การสร้างเส้นทาง (Routes) และหน้าหลัก

### 7.1 สร้างหน้าหลัก (`src/pages/HomePage.js`)

```javascript
import React, { useContext } from 'react';
import { Link } from 'react-router-dom';
import styled from 'styled-components';
import { AuthContext } from '../context/AuthContext';
import Button from '../components/common/Button';

const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
  text-align: center;
`;

const Title = styled.h1`
  font-size: 2.5rem;
  color: #2c3e50;
  margin-bottom: 20px;
`;

const Subtitle = styled.p`
  font-size: 1.2rem;
  color: #7f8c8d;
  margin-bottom: 40px;
`;

const CTASection = styled.div`
  margin-bottom: 60px;
`;

const ButtonGroup = styled.div`
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-top: 30px;
  
  @media (max-width: 768px) {
    flex-direction: column;
    align-items: center;
  }
`;

const FeatureSection = styled.div`
  margin-top: 60px;
`;

const FeatureTitle = styled.h2`
  color: #34495e;
  margin-bottom: 30px;
`;

const Features = styled.div`
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 30px;
  margin-top: 30px;
`;

const Feature = styled.div`
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  text-align: left;
`;

const FeatureIcon = styled.div`
  font-size: 2rem;
  color: #3498db;
  margin-bottom: 15px;
`;

const FeatureName = styled.h3`
  margin: 0 0 10px;
  color: #2c3e50;
`;

const FeatureDescription = styled.p`
  margin: 0;
  color: #7f8c8d;
  font-size: 0.9rem;
`;

const HomePage = () => {
  const { user } = useContext(AuthContext);

  return (
    <Container>
      <Title>ยินดีต้อนรับสู่ Flip Card App</Title>
      <Subtitle>แอปพลิเคชันการเรียนรู้แบบการ์ดพลิกที่ช่วยในการจดจำและทบทวนความรู้</Subtitle>
      
      <CTASection>
        {user ? (
          <ButtonGroup>
            <Button as={Link} to="/creator">
              โหมดผู้สร้าง
            </Button>
            <Button as={Link} to="/player">
              โหมดผู้เล่น
            </Button>
          </ButtonGroup>
        ) : (
          <>
            <p>เริ่มต้นใช้งานตอนนี้เพื่อสร้างและเล่นกับการ์ดของคุณ</p>
            <ButtonGroup>
              <Button as={Link} to="/login">
                เข้าสู่ระบบ
              </Button>
              <Button as={Link} to="/register" secondary>
                ลงทะเบียน
              </Button>
            </ButtonGroup>
          </>
        )}
      </CTASection>
      
      <FeatureSection>
        <FeatureTitle>คุณสมบัติเด่น</FeatureTitle>
        <Features>
          <Feature>
            <FeatureIcon>🔄</FeatureIcon>
            <FeatureName>การเรียนรู้แบบสองด้าน</FeatureName>
            <FeatureDescription>
              การ์ดแต่ละใบมีสองด้าน: ด้านหน้าสำหรับคำถามและด้านหลังสำหรับคำตอบ
              ช่วยเสริมความจำและความเข้าใจในเนื้อหา
            </FeatureDescription>
          </Feature>
          
          <Feature>
            <FeatureIcon>📚</FeatureIcon>
            <FeatureName>จัดหมวดหมู่ได้ตามต้องการ</FeatureName>
            <FeatureDescription>
              สร้างหมวดหมู่การ์ดตามหัวข้อที่คุณต้องการเรียนรู้ เพื่อให้ง่ายต่อการจัดการ
              และทบทวนความรู้อย่างเป็นระบบ
            </FeatureDescription>
          </Feature>
          
          <Feature>
            <FeatureIcon>👥</FeatureIcon>
            <FeatureName>แบ่งปันความรู้</FeatureName>
            <FeatureDescription>
              สามารถตั้งค่าหมวดหมู่การ์ดเป็นสาธารณะเพื่อแบ่งปันกับผู้อื่น
              ช่วยให้การเรียนรู้เป็นไปอย่างกว้างขวาง
            </FeatureDescription>
          </Feature>
          
          <Feature>
            <FeatureIcon>📱</FeatureIcon>
            <FeatureName>ใช้งานได้ทุกอุปกรณ์</FeatureName>
            <FeatureDescription>
              รองรับการใช้งานบนคอมพิวเตอร์และอุปกรณ์มือถือ ทำให้สามารถเรียนรู้
              ได้ทุกที่ทุกเวลา
            </FeatureDescription>
          </Feature>
          
          <Feature>
            <FeatureIcon>🔒</FeatureIcon>
            <FeatureName>ความปลอดภัย</FeatureName>
            <FeatureDescription>
              ระบบการยืนยันตัวตนและการควบคุมการเข้าถึงที่มีความปลอดภัยสูง
              ช่วยปกป้องข้อมูลของคุณ
            </FeatureDescription>
          </Feature>
          
          <Feature>
            <FeatureIcon>📊</FeatureIcon>
            <FeatureName>ติดตามความก้าวหน้า</FeatureName>
            <FeatureDescription>
              ระบบแสดงความก้าวหน้าในการเรียนรู้ ช่วยให้คุณทราบว่าได้ทบทวนการ์ดไปแล้วกี่ใบ
            </FeatureDescription>
          </Feature>
        </Features>
      </FeatureSection>
    </Container>
  );
};

export default HomePage;
```

### 7.2 สร้างหน้า 404 (`src/pages/NotFoundPage.js`)

```javascript
import React from 'react';
import { Link } from 'react-router-dom';
import styled from 'styled-components';
import Button from '../components/common/Button';

const Container = styled.div`
  max-width: 800px;
  margin: 0 auto;
  padding: 40px 20px;
  text-align: center;
`;

const Title = styled.h1`
  font-size: 6rem;
  color: #3498db;
  margin: 0;
`;

const Subtitle = styled.h2`
  font-size: 2rem;
  color: #2c3e50;
  margin: 20px 0;
`;

const Message = styled.p`
  font-size: 1.2rem;
  color: #7f8c8d;
  margin-bottom: 30px;
`;

const NotFoundPage = () => {
  return (
    <Container>
      <Title>404</Title>
      <Subtitle>ไม่พบหน้าที่คุณกำลังค้นหา</Subtitle>
      <Message>
        หน้าที่คุณกำลังพยายามเข้าถึงอาจถูกย้าย ลบ เปลี่ยนชื่อ
        หรือไม่เคยมีอยู่จริง
      </Message>
      <Button as={Link} to="/">
        กลับไปยังหน้าหลัก
      </Button>
    </Container>
  );
};

export default NotFoundPage;
```

### 7.3 สร้างการป้องกันเส้นทาง (`src/components/common/ProtectedRoute.js`)

```javascript
import React, { useContext } from 'react';
import { Navigate } from 'react-router-dom';
import { AuthContext } from '../../context/AuthContext';

const ProtectedRoute = ({ children }) => {
  const { user, loading } = useContext(AuthContext);

  if (loading) {
    return <div>กำลังโหลด...</div>;
  }

  if (!user) {
    return <Navigate to="/login" />;
  }

  return children;
};

export default ProtectedRoute;
```

### 7.4 สร้างไฟล์ App หลัก (`src/App.js`)

```javascript
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import { CardProvider } from './context/CardContext';
import Layout from './components/common/Layout';
import ProtectedRoute from './components/common/ProtectedRoute';
import HomePage from './pages/HomePage';
import NotFoundPage from './pages/NotFoundPage';
import Login from './components/auth/Login';
import Register from './components/auth/Register';
import CreatorDashboard from './components/creator/CreatorDashboard';
import PlayerDashboard from './components/player/PlayerDashboard';
import './styles/global.css';

function App() {
  return (
    <Router>
      <AuthProvider>
        <CardProvider>
          <Layout>
            <Routes>
              <Route path="/" element={<HomePage />} />
              <Route path="/login" element={<Login />} />
              <Route path="/register" element={<Register />} />
              <Route 
                path="/creator" 
                element={
                  <ProtectedRoute>
                    <CreatorDashboard />
                  </ProtectedRoute>
                } 
              />
              <Route 
                path="/player" 
                element={
                  <ProtectedRoute>
                    <PlayerDashboard />
                  </ProtectedRoute>
                } 
              />
              <Route path="*" element={<NotFoundPage />} />
            </Routes>
          </Layout>
        </CardProvider>
      </AuthProvider>
    </Router>
  );
}

export default App;
```

### 7.5 สร้างไฟล์สไตล์กลาง (`src/styles/global.css`)

```css
@import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap');

:root {
  --primary-color: #3498db;
  --secondary-color: #2980b9;
  --success-color: #2ecc71;
  --danger-color: #e74c3c;
  --light-color: #ecf0f1;
  --dark-color: #2c3e50;
}

* {
  box-sizing: border-box;
}

body {
  font-family: 'Sarabun', sans-serif;
  background-color: #f5f5f5;
  margin: 0;
  padding: 0;
  color: var(--dark-color);
  line-height: 1.6;
}

a {
  text-decoration: none;
  color: var(--primary-color);
}

a:hover {
  color: var(--secondary-color);
}

button {
  font-family: 'Sarabun', sans-serif;
}

input, textarea, select {
  font-family: 'Sarabun', sans-serif;
}
```

## ขั้นตอนที่ 8: การทดสอบและการเปิดใช้งาน

### 8.1 เริ่มต้น Frontend

ในโฟลเดอร์ `frontend` ให้รันคำสั่ง:

```bash
npm start
```

Frontend จะเริ่มทำงานที่พอร์ต 3000 โดยอัตโนมัติ

### 8.2 ทดสอบการทำงานพื้นฐาน

ทดสอบการทำงานพื้นฐานดังนี้:

1. ลงทะเบียนผู้ใช้ใหม่
2. ล็อกอินเข้าสู่ระบบ
3. สร้างหมวดหมู่ใหม่
4. สร้างการ์ดในหมวดหมู่นั้น
5. ทดลองโหมดผู้เล่น

## สรุป

เราได้สร้างแอปพลิเคชัน Flip-Card ที่สมบูรณ์โดยใช้ MERN Stack (MongoDB, Express, React, Node.js) ซึ่งมีความสามารถดังนี้:

1. **Backend**:
    
    - การยืนยันตัวตนและการควบคุมการเข้าถึงด้วย JWT
    - API สำหรับการจัดการผู้ใช้ หมวดหมู่ และการ์ด
    - การรักษาความปลอดภัยหลายชั้น
2. **Frontend**:
    
    - โหมดผู้สร้าง (Creator Mode) สำหรับสร้างและจัดการหมวดหมู่และการ์ด
    - โหมดผู้เล่น (Player Mode) สำหรับเรียนรู้ผ่านการพลิกการ์ด
    - ส่วนติดต่อผู้ใช้ที่ใช้งานง่ายและรองรับการใช้งานบนอุปกรณ์ต่างๆ

แอปพลิเคชันนี้เป็นตัวอย่างที่ดีของการใช้หลักการ AAA (Authentication, Authorization, Accounting) ในการพัฒนาเว็บแอปพลิเคชันที่มีความปลอดภัย ตามหลักสูตรของรายวิชา ทนด.132 พื้นฐานการใช้งานเครือข่ายคอมพิวเตอร์และความมั่นคงไซเบอร์

นักศึกษาสามารถนำโค้ดนี้ไปต่อยอดเพิ่มเติมได้ เช่น:

- เพิ่มการค้นหาการ์ดและหมวดหมู่
- เพิ่มสถิติการเรียนรู้
- เพิ่มการแชร์การ์ดผ่านสื่อสังคมออนไลน์
- เพิ่มการอัปโหลดรูปภาพในการ์ด
- พัฒนาเป็นแอปพลิเคชันมือถือด้วย React Native