# 🔐 CONTEXT v3.4 SHORT - Hotel Shoma Authentication

**Version:** 3.4 | **Date:** 2025-11-27 | **Status:** Production Ready ✅

---

## 🎯 تکنولوژی

- **Backend:** VB.NET, ASP.NET Web Forms
- **Database:** SQL Server (VACC)
- **Hash:** PBKDF2 (100,000 iterations, SHA256)
- **Salt:** 32 bytes per password
- **Architecture:** Layered with Stored Procedures

---

## 🗄️ دیتابیس

### جداول کلیدی (24 جدول):
1. **tblAppUser** - کاربران (Company, User, Customer/Guest)
2. **tblAppUserAuth** - احراز Company/User
3. **tblAppUserGuestAuth** - احراز Customer/Guest ⭐
4. **tblAppPasswordHistory** - تاریخچه رمز
5. **tblAppUserSession** - نشست‌ها
6. **tblAppOperationLog** - لاگ عملیات

### Stored Procedures (28 SP):
- **prcApp_RegisterNewCustomer** - ثبت‌نام میهمان
- **prcApp_CustomerLogin_Simple** ⭐ - ورود میهمان (v3.4 جدید)
- **prcApp_ChangePassword** - تغییر رمز
- **prcApp_CreateUserSession** - ایجاد Session
- + 24 SP دیگر

---

## 💻 کلاس cls_AuthenticationManager (47 تابع)

### 1️⃣ Password Hashing (3 تابع):

#### HashPasswordPBKDF2 ⭐
```vb
Public Shared Function HashPasswordPBKDF2(password As String) As String
```
- **استفاده:** فقط در Register
- Salt تصادفی 32 بایت
- PBKDF2: 100,000 iterations
- Return: [Salt 32][Hash 32] = 64 bytes → Base64 (88 chars)

#### VerifyPassword ⭐
```vb
Public Shared Function VerifyPassword(password As String, storedHash As String) As Boolean
```
- **استفاده:** فقط در Login
- جدا کردن Salt از storedHash
- Hash با همان Salt
- مقایسه SlowEquals
- Return: True/False

#### IsStrongPassword
- بررسی قدرت رمز (8+ chars, Upper, Lower, Digit, Special)

---

### 2️⃣ Customer Authentication (3 تابع) ⭐

#### CustomerLogin (v3.4) ⭐⭐⭐
```vb
Public Shared Function CustomerLogin(username As String, password As String,
    ipAddress As String, userAgent As String) As CustomerLoginResult
```

**فلوچارت:**
```
1. Query: خواندن PasswordHash از tblAppUserGuestAuth
2. تبدیل: Binary → Base64
3. بررسی: isPasswordCorrect = VerifyPassword(password, storedHash) ✅
4. SP: prcApp_CustomerLogin_Simple(@PasswordVerified = isPasswordCorrect)
5. SP: کنترل IsActive, IsLocked
6. SP: اگر @PasswordVerified=1 → موفق (Update)
7. SP: اگر @PasswordVerified=0 → ناموفق (افزایش تلاش، قفل)
8. Return: CustomerLoginResult
```

**نکته مهم:**
```vb
' ❌ هرگز در Login:
Dim hash = HashPasswordPBKDF2(password)  ' Salt جدید!

' ✅ همیشه در Login:
Dim result = VerifyPassword(password, storedHash)  ' Salt ذخیره شده
```

#### RegisterCustomer
- ثبت‌نام میهمان جدید

#### ValidateCustomerSession
- اعتبارسنجی Session

---

### 3️⃣ Company/User Authentication (6 تابع):
- RegisterCompany, RegisterUser
- CompanyLogin, UserLogin
- ValidateUserSession, LogoutUser

### 4️⃣ Password Management (4 تابع):
- ChangePassword, CheckPasswordExpiry
- ResetPassword, GetPasswordStrength

### 5️⃣ Session Management (5 تابع):
- CreateUserSession, ValidateUserSession, RefreshUserSession
- TerminateUserSession, TerminateAllUserSessions

### 6️⃣ Security & Locking (6 تابع):
- CheckAccountLockStatus, LockUserAccount, UnlockUserAccount
- RecordFailedAttempt, ResetFailedAttempts, IsAccountLocked

### 7️⃣ Validation (5 تابع):
- IsValidNationalId (11 رقمی)
- IsValidPersonalId (10 رقمی)
- IsValidEmail, IsValidMobile, IsValidUsername

### 8️⃣ User Management (5 تابع):
- GetUserById, GetUserByUsername, GetUserByEmail
- UpdateUserProfile, DeleteUser

### 9️⃣ Logging & Audit (5 تابع):
- LogUserActivity, LogError, GetLoginHistory
- GetUserActivityLog, CleanupOldLogs

### 🔟 Utilities (5 تابع):
- GetUserIP, ConvertToSqlBinary, GenerateSecurityStamp
- GenerateRandomPassword, SendEmail

---

## 📱 فرم‌ها (3 فرم)

1. **login.aspx** - ورود (Company, User, Customer)
2. **CustomerRegister.aspx** - ثبت‌نام میهمان
3. **changepassword.aspx** - تغییر رمز

---

## 🔒 امنیت

### PBKDF2:
```
Algorithm: SHA256
Iterations: 100,000
Salt: 32 bytes (random per user)
Hash: 32 bytes
Total: 64 bytes → 88 chars Base64
Format: [Salt 32][Hash 32]
```

### Password Policy:
- حداقل 8 کاراکتر
- 1 حرف بزرگ، 1 کوچک، 1 عدد، 1 خاص
- نباید در 5 رمز قبلی باشد

### Account Lockout:
- بعد از 5 تلاش ناموفق
- مدت: 15 دقیقه
- باز شدن خودکار

---

## 🆕 تغییرات v3.4 (مهم!) ⭐

### مشکل حل شده: Hash متفاوت در CustomerLogin

#### قبل (v3.3):
```vb
' در Login:
Dim hashedPassword = HashPasswordPBKDF2(password)  ❌
' Salt جدید → Hash جدید → همیشه False

' در SP:
IF @PasswordHash = @StoredPasswordHash  ❌
```

#### بعد (v3.4):
```vb
' در Login:
Dim storedHash = Convert.ToBase64String(hashBytes)
Dim isPasswordCorrect = VerifyPassword(password, storedHash)  ✅

' در SP:
IF @PasswordVerified = 1  ✅
```

### فایل‌های جدید:
1. **CustomerLogin_FINAL.vb** (7 KB) - تابع اصلاح شده
2. **prcApp_CustomerLogin_Simple.sql** (9 KB) - SP جدید

---

## 🐛 مشکلات حل شده

### v3.4:
- ✅ Hash متفاوت در CustomerLogin
- ✅ استفاده از VerifyPassword در Login

### v3.3:
- ✅ Change Password (LastPasswordChangeDate در Auth tables)
- ✅ Log در tblAppOperationLog

### v3.2:
- ✅ پشتیبانی UserAgent

---

## 📊 آمار

- **جداول:** 24
- **SP:** 28
- **توابع:** 47
- **فرم‌ها:** 3
- **مستندات:** 16 فایل

---

## 💡 نکات مهم

### 1. Hash vs Verify:
```vb
' Register: HashPasswordPBKDF2 (Salt جدید)
' Login: VerifyPassword (Salt ذخیره شده)
```

### 2. Hash Format:
```
DB: VARBINARY(MAX) - 64 bytes
Base64: 88 characters
[Salt 32 bytes][Hash 32 bytes]
```

### 3. VerifyPassword:
```
1. Decode Base64 → 64 bytes
2. Split: Salt (0-31), Hash (32-63)
3. PBKDF2(password, Salt) → Computed
4. Compare: Stored = Computed?
```

### 4. LastPasswordChangeDate:
```
❌ نه در tblAppUser
✅ در tblAppUserAuth (UserType 1,2)
✅ در tblAppUserGuestAuth (UserType 3)
```

---

## 📚 مستندات

- **Context کامل:** PROJECT_CONTEXT_v3_4_COMPLETE.md (19 KB)
- **خلاصه:** CONTEXT_v3_4_خلاصه.md (3 KB)
- **راهنمای نصب:** نصب_سریع_CustomerLogin.md (6 KB)

---

## 🔗 لینک پروژه

https://claude.ai/project/019a9272-8987-72d1-9fb9-8b59f17fb482

---

**© 2025 Hotel Shoma Authentication v3.4**  
**Status:** Production Ready ✅
