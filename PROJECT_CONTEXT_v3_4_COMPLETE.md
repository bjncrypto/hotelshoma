# 🔐 PROJECT CONTEXT v3.4 - Hotel Shoma Authentication System

**Date:** 2025-11-27  
**Version:** 3.4  
**Status:** Production Ready ✅

---

## 📋 تاریخچه Context

- **v1.0:** معماری اولیه + توابع Register
- **v2.0:** اضافه Session Management + Login
- **v3.0:** اضافه CustomerLogin اولیه (با مشکل Hash)
- **v3.1:** اصلاح Change Password
- **v3.2:** پشتیبانی UserAgent
- **v3.3:** CustomerLogin اولیه (همچنان با مشکل)
- **v3.4:** 🔧 حل مشکل Hash در CustomerLogin (نسخه فعلی)

---

## 🆕 تغییرات v3.4 (مهم!)

### مشکل حل شده:
```
❌ مشکل: در Login، Hash های متفاوت تولید می‌شدند
   - VB.NET: HashPasswordPBKDF2(password) → Salt جدید → Hash جدید
   - SP: مقایسه ساده → Hash جدید ≠ Hash DB → همیشه False

✅ راه‌حل: استفاده از VerifyPassword
   - VB.NET: VerifyPassword(password, storedHash) → استفاده از Salt ذخیره شده
   - SP: بر اساس نتیجه Verify عمل می‌کند
```

### فایل‌های اصلاح شده:

1. **CustomerLogin_FINAL.vb** (جدید) - 7 KB
   - بررسی رمز با `VerifyPassword()`
   - خواندن Hash از DB
   - فراخوانی SP با `@PasswordVerified`

2. **prcApp_CustomerLogin_Simple** (جدید) - 9 KB
   - پارامتر جدید: `@PasswordVerified BIT`
   - بدون مقایسه Hash
   - فقط کنترل‌ها و Update ها

3. **نصب_سریع_CustomerLogin.md** - 6 KB
4. **راه_حل_کامل_CustomerLogin.md** - 14 KB

---

## 🏗️ معماری سیستم

### تکنولوژی
- **Backend:** VB.NET, ASP.NET Web Forms
- **Database:** SQL Server (VACC)
- **Authentication:** PBKDF2 (100,000 iterations, SHA256)
- **Architecture:** Layered (Presentation → Business Logic → Data Access)

### اصول طراحی
- ✅ Stored Procedures برای تمام عملیات DB
- ✅ PBKDF2 برای Hash رمز عبور
- ✅ Separation of Concerns
- ✅ Transaction Management

---

## 🗄️ دیتابیس (24 جدول)

### جداول کلیدی:

#### 1. tblAppUser (کاربران)
```sql
- idUser INT PRIMARY KEY
- idEntity INT (FK → tblAppEntity)
- idPrimaryCompany INT (FK → tblAppCompany)
- Username NVARCHAR(100) UNIQUE
- Email NVARCHAR(200) UNIQUE
- Mobile NVARCHAR(20) UNIQUE
- UserType INT (1=Company, 2=User, 3=Customer/Guest)
- IsActive BIT
- IsLocked BIT
- LockoutEnd DATETIME2
- FailedLoginAttempts INT
- LastLoginDate DATETIME2
- LastLoginIP NVARCHAR(50)
- EmailConfirmed BIT
- MobileConfirmed BIT
- TwoFactorEnabled BIT
- CreatedDate DATETIME2
- IsDeleted BIT
```

#### 2. tblAppUserAuth (احراز هویت Company/User)
```sql
- idUserAuth INT PRIMARY KEY
- idUser INT (FK → tblAppUser)
- PasswordHash VARBINARY(MAX)  -- PBKDF2: [Salt 32][Hash 32] = 64 bytes
- PasswordSalt VARBINARY(256)
- HashAlgorithm NVARCHAR(50) = 'PBKDF2'
- PasswordParams NVARCHAR(MAX) = '{"iterations":100000,"keyLength":32}'
- LastPasswordChangeDate DATETIME2
- MustChangePassword BIT
- PasswordExpiryDays INT
- SecurityStamp UNIQUEIDENTIFIER
```

#### 3. tblAppUserGuestAuth (احراز هویت Customer/Guest)
```sql
- idGuestAuth INT PRIMARY KEY
- idUser INT (FK → tblAppUser)
- PasswordHash VARBINARY(MAX)  -- PBKDF2: [Salt 32][Hash 32]
- PasswordSalt VARBINARY(256)
- HashAlgorithm NVARCHAR(50) = 'PBKDF2'
- PasswordParams NVARCHAR(MAX)
- LastPasswordChangeDate DATETIME2
- MustChangePassword BIT
- SecurityStamp UNIQUEIDENTIFIER
- FailedLoginAttempts INT
- IsLockedOut BIT
- LastLoginIP NVARCHAR(50)
- CreatedDate DATETIME2
- CreatedByIP NVARCHAR(50)
```

#### 4. tblAppPasswordHistory (تاریخچه رمز)
```sql
- idPasswordHistory INT PRIMARY KEY
- idUser INT (FK → tblAppUser)
- PasswordHash VARBINARY(MAX)
- ChangedDate DATETIME2
- ChangedByIP NVARCHAR(50)
```

#### 5. tblAppUserSession (نشست‌ها)
```sql
- idSession UNIQUEIDENTIFIER PRIMARY KEY
- idUser INT (FK → tblAppUser)
- SessionToken NVARCHAR(500)
- IPAddress NVARCHAR(50)
- UserAgent NVARCHAR(500)
- DeviceInfo NVARCHAR(MAX)
- IsActive BIT
- CreatedDate DATETIME2
- ExpiryDate DATETIME2
- LastActivityDate DATETIME2
```

#### 6. tblAppOperationLog (لاگ عملیات)
```sql
- idOperationLog BIGINT PRIMARY KEY
- idUser INT
- idSession UNIQUEIDENTIFIER
- idMenuPage INT
- ActionType NVARCHAR(100)      -- 'Login', 'ChangePassword', 'Register'
- ActionEntity NVARCHAR(100)    -- 'User', 'Password', 'Session'
- EntityId NVARCHAR(50)
- ActionDetails NVARCHAR(MAX)
- IPAddress NVARCHAR(50)
- UserAgent NVARCHAR(500)
- StatusCode INT                -- 200, 401, 403, 404, 409, 423, 500
- ErrorMessage NVARCHAR(MAX)
- LogDate DATETIME2
```

---

## 📝 Stored Procedures (28 SP)

### Customer/Guest (5 SP):
1. **prcApp_RegisterNewCustomer** - ثبت‌نام میهمان
2. **prcApp_CustomerLogin_Simple** ⭐ - ورود میهمان (جدید v3.4)
3. **prcApp_CreateUserSession** - ایجاد Session
4. **prcApp_ValidateUserSession** - اعتبارسنجی Session
5. **prcApp_TerminateUserSession** - پایان Session

### Company/User (8 SP):
6. **prcApp_RegisterCompany** - ثبت‌نام شرکت
7. **prcApp_RegisterUser** - ثبت‌نام کاربر
8. **prcApp_CompanyLogin** - ورود شرکت
9. **prcApp_UserLogin** - ورود کاربر
10. **prcApp_GetUserByUsername** - دریافت کاربر
11. **prcApp_GetUserById** - دریافت کاربر با ID
12. **prcApp_UpdateUserProfile** - بروزرسانی پروفایل
13. **prcApp_DeleteUser** - حذف منطقی کاربر

### Password Management (5 SP):
14. **prcApp_ChangePassword** - تغییر رمز عبور
15. **prcApp_CheckPasswordExpiry** - بررسی انقضای رمز
16. **prcApp_ResetPassword** - بازیابی رمز
17. **prcApp_ValidatePasswordHistory** - بررسی تاریخچه
18. **prcApp_RecordPasswordChange** - ثبت تغییر رمز

### Security & Audit (10 SP):
19. **prcApp_RecordFailedLoginAttempt** - ثبت تلاش ناموفق
20. **prcApp_ResetFailedLoginAttempts** - ریست تلاش‌ها
21. **prcApp_LockUserAccount** - قفل کردن حساب
22. **prcApp_UnlockUserAccount** - باز کردن قفل
23. **prcApp_CheckAccountLockStatus** - بررسی وضعیت قفل
24. **prcApp_LogUserActivity** - ثبت فعالیت
25. **prcApp_GetUserSessions** - دریافت Session های کاربر
26. **prcApp_TerminateAllUserSessions** - پایان تمام Session ها
27. **prcApp_GetLoginHistory** - تاریخچه ورود
28. **prcApp_CleanupExpiredSessions** - پاکسازی Session های منقضی

---

## 💻 کلاس cls_AuthenticationManager (47 تابع)

### 1️⃣ Password Hashing & Verification (3 تابع)

#### HashPasswordPBKDF2 ⭐
```vb
Public Shared Function HashPasswordPBKDF2(password As String) As String
```
**استفاده:** فقط در Register  
**کار:**
- تولید Salt تصادفی (32 بایت)
- PBKDF2 با 100,000 iterations
- ترکیب [Salt 32 bytes][Hash 32 bytes] = 64 bytes
- Return: Base64 String (88 chars)

#### VerifyPassword ⭐
```vb
Public Shared Function VerifyPassword(password As String, storedHash As String) As Boolean
```
**استفاده:** در Login  
**کار:**
- Decode Base64 → 64 bytes
- جدا کردن Salt (0-31) و Hash (32-63)
- PBKDF2(password, Salt) با 100,000 iterations
- مقایسه SlowEquals
- Return: True/False

#### IsStrongPassword
```vb
Public Shared Function IsStrongPassword(password As String) As Boolean
```
**بررسی:**
- حداقل 8 کاراکتر
- حداقل 1 حرف بزرگ (A-Z)
- حداقل 1 حرف کوچک (a-z)
- حداقل 1 عدد (0-9)
- حداقل 1 کاراکتر خاص (!@#$%^&*)

---

### 2️⃣ Customer Authentication (3 تابع) ⭐

#### CustomerLogin (v3.4 - اصلاح شده) ⭐⭐⭐
```vb
Public Shared Function CustomerLogin(username As String, password As String, 
    ipAddress As String, userAgent As String) As CustomerLoginResult
```
**فلوچارت:**
```
1. خواندن PasswordHash از DB (tblAppUserGuestAuth)
2. تبدیل Binary → Base64
3. بررسی: isPasswordCorrect = VerifyPassword(password, storedHash)
4. فراخوانی prcApp_CustomerLogin_Simple با @PasswordVerified
5. SP: بررسی IsActive, IsLocked
6. SP: اگر @PasswordVerified=1 → موفق (Update اطلاعات)
7. SP: اگر @PasswordVerified=0 → ناموفق (افزایش تلاش، قفل)
8. Return: CustomerLoginResult
```

**نکات مهم:**
- ❌ هرگز `HashPasswordPBKDF2` در Login استفاده نکنید
- ✅ همیشه `VerifyPassword` استفاده کنید
- ✅ Salt از storedHash جدا می‌شود
- ✅ مقایسه صحیح با همان Salt

#### RegisterCustomer
```vb
Public Shared Function RegisterCustomer(...) As RegistrationResult
```

#### ValidateCustomerSession
```vb
Public Shared Function ValidateCustomerSession(sessionToken As String) As SessionValidationResult
```

---

### 3️⃣ Company/User Authentication (6 تابع)

1. **RegisterCompany**
2. **RegisterUser**
3. **CompanyLogin**
4. **UserLogin**
5. **ValidateUserSession**
6. **LogoutUser**

---

### 4️⃣ Password Management (4 تابع)

1. **ChangePassword**
2. **CheckPasswordExpiry**
3. **ResetPassword**
4. **GetPasswordStrength**

---

### 5️⃣ Session Management (5 تابع)

1. **CreateUserSession**
2. **ValidateUserSession**
3. **RefreshUserSession**
4. **TerminateUserSession**
5. **TerminateAllUserSessions**

---

### 6️⃣ Security & Locking (6 تابع)

1. **CheckAccountLockStatus**
2. **LockUserAccount**
3. **UnlockUserAccount**
4. **RecordFailedAttempt**
5. **ResetFailedAttempts**
6. **IsAccountLocked**

---

### 7️⃣ Validation (5 تابع)

1. **IsValidNationalId** - شناسه ملی 11 رقمی
2. **IsValidPersonalId** - کد ملی 10 رقمی
3. **IsValidEmail**
4. **IsValidMobile**
5. **IsValidUsername**

---

### 8️⃣ User Management (5 تابع)

1. **GetUserById**
2. **GetUserByUsername**
3. **GetUserByEmail**
4. **UpdateUserProfile**
5. **DeleteUser**

---

### 9️⃣ Logging & Audit (5 تابع)

1. **LogUserActivity**
2. **LogError**
3. **GetLoginHistory**
4. **GetUserActivityLog**
5. **CleanupOldLogs**

---

### 🔟 Utilities (5 تابع)

1. **GetUserIP**
2. **ConvertToSqlBinary**
3. **GenerateSecurityStamp**
4. **GenerateRandomPassword**
5. **SendEmail**

---

## 📱 فرم‌ها (3 فرم)

### 1. login.aspx / login.aspx.vb
**کاربران:** Company, User, Customer/Guest

**فیلدها:**
- Username
- Password
- Captcha

**فلوچارت:**
```
1. Validation ورودی
2. بررسی Captcha
3. بررسی Lock Status
4. تشخیص نوع کاربر (از Username یا UserType)
5. فراخوانی تابع مناسب:
   - CompanyLogin (UserType=1)
   - UserLogin (UserType=2)
   - CustomerLogin (UserType=3)
6. ایجاد Session
7. هدایت به Panel
```

**کد نمونه:**
```vb
' خط 240 - CustomerLogin:
Dim loginResult = cls_AuthenticationManager.CustomerLogin(
    username, password, ipAddress, userAgent)

If loginResult.Success Then
    ' ایجاد Session
    FormsAuthentication.SetAuthCookie(username, False)
    Response.Redirect("~/customer/dashboard")
Else
    ShowMessage(loginResult.Message, "danger")
End If
```

---

### 2. CustomerRegister.aspx / CustomerRegister.aspx.vb
**کاربران:** Customer/Guest

**فیلدها:**
- FirstName, LastName
- Mobile (Username)
- Email (optional)
- Password, ConfirmPassword
- Terms Checkbox
- Captcha

**فلوچارت:**
```
1. Validation ورودی
2. بررسی Captcha
3. Hash کردن رمز: hashedPassword = HashPasswordPBKDF2(password)
4. فراخوانی RegisterCustomer(...)
5. ذخیره در: tblAppUser + tblAppEntity + tblAppUserGuestAuth
6. ارسال ایمیل خوش‌آمدگویی
7. ورود خودکار
8. هدایت به Dashboard
```

---

### 3. changepassword.aspx / changepassword.aspx.vb
**کاربران:** همه

**فیلدها:**
- CurrentPassword
- NewPassword
- ConfirmNewPassword

---

## 🔒 امنیت

### PBKDF2 Configuration
```
Algorithm: SHA256
Iterations: 100,000
Salt Size: 32 bytes
Hash Size: 32 bytes
Total: 64 bytes → 88 chars Base64
```

### Hash Format
```
Storage: VARBINARY(MAX) in DB
Format: [Salt 32 bytes][Hash 32 bytes]
Base64: 88 characters
```

### Password Policy
```
- حداقل 8 کاراکتر
- حداقل 1 حرف بزرگ
- حداقل 1 حرف کوچک
- حداقل 1 عدد
- حداقل 1 کاراکتر خاص
- نباید در 5 رمز قبلی باشد
```

### Account Lockout
```
- بعد از 5 تلاش ناموفق
- مدت قفل: 15 دقیقه
- باز شدن خودکار بعد از LockoutEnd
```

### Session Management
```
- Timeout: 8 ساعت (قابل تنظیم)
- Session Token: PBKDF2 Hashed
- Validation در هر Request
- Cleanup خودکار منقضی شده‌ها
```

---

## 📊 StatusCode (Logging)

```
200: موفق (Success)
401: احراز هویت ناموفق (Unauthorized)
403: دسترسی غیرمجاز (Forbidden)
404: یافت نشد (Not Found)
409: تضاد/تکراری (Conflict/Duplicate)
423: قفل شده (Locked)
500: خطای سرور (Server Error)
```

---

## 🐛 مشکلات حل شده

### v3.4: مشکل Hash متفاوت در CustomerLogin ⭐

**مشکل:**
```vb
' در Login (اشتباه):
Dim hashedPassword = HashPasswordPBKDF2(password)  ❌
' Salt جدید → Hash جدید → هیچوقت برابر نیست

' در SP:
IF @PasswordHash = @StoredPasswordHash  ❌
' مقایسه ساده → همیشه False
```

**راه‌حل:**
```vb
' در Login (صحیح):
Dim storedHash = Convert.ToBase64String(hashBytes)
Dim isPasswordCorrect = VerifyPassword(password, storedHash)  ✅

' در SP:
IF @PasswordVerified = 1  ✅
```

**فایل‌ها:**
- CustomerLogin_FINAL.vb (7 KB)
- prcApp_CustomerLogin_Simple.sql (9 KB)
- نصب_سریع_CustomerLogin.md (6 KB)

---

### v3.3: مشکل Change Password

**مشکل:**
- استفاده از tblAppErrorLog (جدول موجود نبود)
- LastPasswordChangeDate در tblAppUser (فیلد موجود نبود)

**راه‌حل:**
- ثبت Log در tblAppOperationLog
- LastPasswordChangeDate در tblAppUserAuth/GuestAuth

---

### v3.2: فقدان UserAgent

**راه‌حل:**
- اضافه UserAgent به تمام توابع
- ثبت در tblAppOperationLog

---

## 📂 فایل‌های مهم (مستندات)

### Context Files (4 فایل):
1. **PROJECT_CONTEXT_v3_4_COMPLETE.md** (این فایل) - 19 KB
2. **CONTEXT_v3_4_خلاصه.md** - 3 KB
3. **CONTEXT_v3_4_SHORT.md** - 6 KB
4. **راهنمای_استفاده_از_Context.md** - 4 KB

### CustomerLogin Fix (4 فایل):
5. **CustomerLogin_FINAL.vb** - 7 KB
6. **prcApp_CustomerLogin_Simple.sql** - 9 KB
7. **نصب_سریع_CustomerLogin.md** - 6 KB
8. **راه_حل_کامل_CustomerLogin.md** - 14 KB

### Previous Docs (8 فایل):
9. **Authentication_Security_Documentation.docx** - 15 KB
10. **Authentication_Technical_Review.docx** - 15 KB
11. **StoredProcedure_ChangePassword_FIXED.sql** - 18 KB
12. **cls_AuthenticationManager_ChangePassword_FIXED.vb** - 11 KB
13. **changepassword.aspx_FIXED.vb** - 9 KB
14. **changepassword.aspx** - 19 KB
15. **CustomerRegister_aspx.vb** - 4 KB
16. **StoredProcedures_CustomerLogin_Session.sql** - 15 KB

**مجموع:** 16 فایل مستندات

---

## 📈 آمار نهایی

- **جداول:** 24
- **Stored Procedures:** 28 (+ prcApp_CustomerLogin_Simple جدید)
- **توابع کلاس:** 47
- **فرم‌ها:** 3
- **مستندات:** 16 فایل
- **Context Files:** 4
- **نسخه:** v3.4

---

## ✅ چک‌لیست Production

### Database:
- [x] تمام SP ها نصب شده
- [x] تمام جداول ایجاد شده
- [x] Indexes تنظیم شده
- [x] Foreign Keys فعال
- [x] Backup فعال

### Security:
- [x] PBKDF2 با 100k iterations
- [x] Password Policy فعال
- [x] Account Lockout فعال
- [x] Session Management فعال
- [x] Logging فعال

### Code:
- [x] cls_AuthenticationManager کامل
- [x] تمام فرم‌ها تست شده
- [x] CustomerLogin اصلاح شده (v3.4)
- [x] Error Handling جامع
- [x] مستندات کامل

### Testing:
- [x] Register (Company, User, Customer) ✅
- [x] Login (Company, User, Customer) ✅
- [x] Change Password ✅
- [x] Session Management ✅
- [x] Account Lockout ✅

---

## 🎯 نکات مهم برای توسعه‌دهنده

### 1. هرگز در Login دوباره Hash نکنید:
```vb
' ❌ اشتباه:
Dim hash = HashPasswordPBKDF2(password)  ' Salt جدید!

' ✅ صحیح:
Dim result = VerifyPassword(password, storedHash)  ' Salt ذخیره شده
```

### 2. Hash Format:
```
DB: VARBINARY(MAX) → 64 bytes
Base64: 88 characters
Structure: [Salt 32][Hash 32]
```

### 3. VerifyPassword چطور کار می‌کند:
```vb
1. Decode Base64 → 64 bytes
2. Split: Salt (0-31), Hash (32-63)
3. PBKDF2(password, Salt, 100k) → Computed Hash
4. SlowEquals(Stored Hash, Computed Hash) → True/False
```

### 4. SP ها فقط برای Update:
```vb
' بررسی رمز در VB.NET:
Dim isCorrect = VerifyPassword(...)

' SP فقط برای Update و کنترل‌ها:
prcApp_CustomerLogin_Simple(@PasswordVerified = isCorrect)
```

### 5. LastPasswordChangeDate:
```
❌ نه در tblAppUser
✅ در tblAppUserAuth (UserType 1,2)
✅ در tblAppUserGuestAuth (UserType 3)
```

---

## 🔗 لینک‌های مفید

- **پروژه Master:** https://claude.ai/project/019a9272-8987-72d1-9fb9-8b59f17fb482
- **Context ها:** /mnt/user-data/outputs/
- **مستندات:** /mnt/project/

---

## 📝 یادداشت‌های توسعه

### برای گفتگوی جدید:
```
استفاده کنید از: CONTEXT_v3_4_SHORT.md
```

### برای مرور سریع:
```
استفاده کنید از: CONTEXT_v3_4_خلاصه.md
```

### برای مرجع کامل:
```
استفاده کنید از: PROJECT_CONTEXT_v3_4_COMPLETE.md (این فایل)
```

---

**© 2025 Hotel Shoma - Authentication System v3.4**  
**Last Updated:** 2025-11-27  
**Status:** Production Ready ✅
