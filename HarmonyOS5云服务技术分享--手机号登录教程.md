### Why Choose Phone Number Authentication?  
Before diving into the code, let's discuss its advantages:  

- **User-Friendly**: No need to memorize complex usernames.  
- **High Security**: Dual verification mechanism (SMS + password).  
- **Quick Integration**: Core logic is already encapsulated in the HarmonyOS Auth SDK.  


### Environment Preparation  
Ensure your project has:  

- Integrated the AGC authentication SDK.  
- Enabled phone authentication capabilities in the AGC console.  
- Added the following permission in module.json5:  
  ```json  
  "requestPermissions": [  
    {"name": "ohos.permission.READ_SMS"} // Required for automatic SMS verification code reading  
  ]  
  ```  


### Core Function Implementation (with Code)  
🔑 **Scenario 1: New User Registration**  
**Two steps: Get verification code → Create account**  

```typescript  
// Step 1: Request verification code  
import auth, { VerifyCodeAction } from '@hw-agconnect/auth';  

auth.requestVerifyCode({  
  action: VerifyCodeAction.REGISTER_LOGIN, // Shared type for registration and login  
  lang: 'zh_CN', // SMS language  
  sendInterval: 60, // SMS resend interval (seconds)  
  verifyCodeType: {  
    phoneNumber: '13812345678', // Remember to replace with a real number!  
    countryCode: "86",  
    kind: "phone"  
  }  
}).then(() => {  
  console.log("Verification code sent, please check!");  
}).catch(err => {  
  console.error("Failed to send:", err.message);  
});  

// Step 2: Create user (auto-login)  
auth.createUser({  
  kind: 'phone',  
  countryCode: '86',  
  phoneNumber: '13812345678',  
  password: 'Init@123', // Optional initial password  
  verifyCode: '654321' // 6-digit code received by the user  
}).then(user => {  
  console.log("Registration successful! UID:", user.uid);  
});  
```  

🔐 **Scenario 2: Password Login**  
```typescript  
auth.signIn({  
  credentialInfo: {  
    kind: 'phone',  
    phoneNumber: '13812345678',  
    countryCode: '86',  
    password: 'your_password' // User-set password  
  }  
}).then(user => {  
  console.log("Login successful! Current user:", user);  
}).catch(err => {  
  if(err.code === 203817858) { // Special handling for incorrect password  
    console.warn("Incorrect password, remaining attempts:", err.remainingTimes);  
  }  
});  
```  

📲 **Scenario 3: Verification Code Login**  
```typescript  
// First request the verification code (same parameters as above)  
auth.requestVerifyCode({  
  action: VerifyCodeAction.REGISTER_LOGIN,  
  // ...other parameters remain the same  
});  

// Login with verification code  
auth.signIn({  
  credentialInfo: {  
    kind: 'phone',  
    phoneNumber: '13812345678',  
    countryCode: '86',  
    verifyCode: '123456'  
  }  
}).then(user => {  
  console.log("Password-free login successful!");  
});  
```  


### Account Management Tips  
**Change bound phone number (requires login):**  
```typescript  
user.updatePhone({  
  countryCode: '86',  
  phoneNumber: '13887654321', // New number  
  verifyCode: '112233',  
  lang: "zh_CN"  
}).then(() => {  
  console.log("Phone number updated successfully!");  
});  
```  

**Change password (after login):**  
```typescript  
user.updatePassword({  
  password: 'NewPwd@2024',  
  providerType: 'phone'  
});  
```  

**Forgot password? Reset it with one click:**  
```typescript  
// First request a verification code for password reset  
auth.requestVerifyCode({  
  action: VerifyCodeAction.RESET_PASSWORD,  
  // ...other parameters remain the same  
});  

// Execute reset  
auth.resetPassword({  
  kind: 'phone',  
  password: 'FreshStart@123',  
  phoneNumber: '13812345678',  
  countryCode: '86',  
  verifyCode: '665544'  
});  
```  


### Pitfall Prevention Guide 🚧  
- **Sensitive operation protection**: When changing the phone number/password, the user must have logged in within the last 5 minutes.  
- **Error code handling**:  
  - 203817932: Incorrect verification code.  
  - 203817933: Verification code expired.  
  - 203817945: Operation too frequent.  
- **Multi-device login**: Enable via `auth.settings.enableMultiDevice(true)`.  


### Extended Capabilities 🔗  
Want to strengthen your authentication system? Try these:  

- **Account linking**: Bind WeChat/email for multi-way login.  
- **Cloud function triggers**: Listen for user registration/login events.  
- **Security reinforcement**: Enable two-factor authentication (2FA).  


### Final Words  
Hope this guide helps you master phone number authentication in HarmonyOS! If you encounter issues, feel free to ask in the comments. Don’t forget to implement exception handling and logging in actual development.  

If there are other features you’d like to learn about, leave a message in the comments! See you next time~ 👋
