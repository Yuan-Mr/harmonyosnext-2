一、为什么选择手机号认证？
在开始代码之前，先说说它的优势：

​​用户友好​​：不需要记忆复杂用户名
​​安全性强​​：双重验证机制（短信+密码）
​​快速接入​​：HarmonyOS Auth SDK已封装好核心逻辑
二、环境准备
先确保你的项目已经：

集成AGC认证SDK
在AGC控制台开启手机认证能力
在module.json5添加权限：
`"requestPermissions": [ {"name": "ohos.permission.READ_SMS"} // 如果需要自动读取短信验证码 ]`
三、核心功能实现（附代码）
🔑 场景1：新用户注册
​​分两步走：获取验证码 → 创建账号​

import auth, { VerifyCodeAction } from '@hw-agconnect/auth';

auth.requestVerifyCode({
  action: VerifyCodeAction.REGISTER_LOGIN, // 注册登录共用类型
  lang: 'zh_CN', // 短信语言
  sendInterval: 60, // 短信重发间隔(秒)
  verifyCodeType: {
    phoneNumber: '13812345678', // 记得替换真实号码！
    countryCode: "86",
    kind: "phone"
  }
}).then(() => {
  console.log("验证码已发送，注意查收~");
}).catch(err => {
  console.error("发送失败:", err.message);
});

// 第二步：创建用户（自动登录）
auth.createUser({
  kind: 'phone',
  countryCode: '86',
  phoneNumber: '13812345678',
  password: 'Init@123', // 初始密码（可选）
  verifyCode: '654321' // 用户收到的6位验证码
}).then(user => {
  console.log("注册成功！UID:", user.uid);
});
🔐 场景2：密码登录
  credentialInfo: {
    kind: 'phone',
    phoneNumber: '13812345678',
    countryCode: '86',
    password: 'your_password' // 用户设置的密码
  }
}).then(user => {
  console.log("登录成功！当前用户:", user);
}).catch(err => {
  if(err.code === 203817858) { // 密码错误特殊处理
    console.warn("密码错误，还剩", err.remainingTimes, "次尝试机会");
  }
});
✨ 嗨，各位HarmonyOS开发者小伙伴！今天咱们来聊聊如何在应用中集成「手机号登录认证」功能。无论是用"手机号+密码"还是"手机号+验证码"，这篇保姆级教程都会手把手带你实现。准备好了吗？Let's go~ 🚀

一、为什么选择手机号认证？
在开始代码之前，先说说它的优势：

​​用户友好​​：不需要记忆复杂用户名
​​安全性强​​：双重验证机制（短信+密码）
​​快速接入​​：HarmonyOS Auth SDK已封装好核心逻辑
二、环境准备
先确保你的项目已经：

集成AGC认证SDK
在AGC控制台开启手机认证能力
在module.json5添加权限：
json
复制
"requestPermissions": [
  {"name": "ohos.permission.READ_SMS"} // 如果需要自动读取短信验证码
]
三、核心功能实现（附代码）
🔑 场景1：新用户注册
​​分两步走：获取验证码 → 创建账号​​

typescript
复制
// 第一步：申请验证码
import auth, { VerifyCodeAction } from '@hw-agconnect/auth';

auth.requestVerifyCode({
  action: VerifyCodeAction.REGISTER_LOGIN, // 注册登录共用类型
  lang: 'zh_CN', // 短信语言
  sendInterval: 60, // 短信重发间隔(秒)
  verifyCodeType: {
    phoneNumber: '13812345678', // 记得替换真实号码！
    countryCode: "86",
    kind: "phone"
  }
}).then(() => {
  console.log("验证码已发送，注意查收~");
}).catch(err => {
  console.error("发送失败:", err.message);
});

// 第二步：创建用户（自动登录）
auth.createUser({
  kind: 'phone',
  countryCode: '86',
  phoneNumber: '13812345678',
  password: 'Init@123', // 初始密码（可选）
  verifyCode: '654321' // 用户收到的6位验证码
}).then(user => {
  console.log("注册成功！UID:", user.uid);
});
🔐 场景2：密码登录
typescript
复制
auth.signIn({
  credentialInfo: {
    kind: 'phone',
    phoneNumber: '13812345678',
    countryCode: '86',
    password: 'your_password' // 用户设置的密码
  }
}).then(user => {
  console.log("登录成功！当前用户:", user);
}).catch(err => {
  if(err.code === 203817858) { // 密码错误特殊处理
    console.warn("密码错误，还剩", err.remainingTimes, "次尝试机会");
  }
});
📲 场景3：验证码登录
auth.requestVerifyCode({
  action: VerifyCodeAction.REGISTER_LOGIN,
  // ...其他参数同上
});

// 使用验证码登录
auth.signIn({
  credentialInfo: {
    kind: 'phone',
    phoneNumber: '13812345678',
    countryCode: '86',
    verifyCode: '123456' 
  }
}).then(user => {
  console.log("无密码登录成功！");
});
四、账号管理技巧
修改绑定手机号（需已登录）
  user.updatePhone({
    countryCode: '86',
    phoneNumber: '13887654321', // 新号码
    verifyCode: '112233',
    lang: "zh_CN"
  }).then(() => {
    console.log("换绑成功！");
  });
});
修改密码（登录后操作）
  user.updatePassword({
    password: 'NewPwd@2024',
    providerType: 'phone' 
  });
});
忘记密码？一键重置！
auth.requestVerifyCode({
  action: VerifyCodeAction.RESET_PASSWORD,
  // ...其他参数同上
});

// 执行重置
auth.resetPassword({
  kind: 'phone',
  password: 'FreshStart@123',
  phoneNumber: '13812345678',
  countryCode: '86',
  verifyCode: '665544'
});
五、避坑指南 🚧
​​敏感操作保护​​：修改手机号/密码时，必须5分钟内登录过

​​错误码处理​​：

203817932: 验证码错误
203817933: 验证码过期
203817945: 操作过于频繁
​​多设备登录​​：通过auth.settings.enableMultiDevice(true)开启支持

六、扩展能力 🔗
想让你的认证系统更强大？ 试试这些：

• ​​账号关联​​：绑定微信/邮箱实现多方式登录
• ​​云函数触发​​：监听用户注册/登录事件
• ​​安全加固​​：启用二步验证（2FA）
最后的话
希望这篇指南能让你轻松玩转HarmonyOS手机认证！如果遇到问题，欢迎评论区提问交流。别忘了在实际开发中做好异常处理和日志记录哦~

如果有其他想了解的功能，欢迎在评论区留言告诉我！咱们下期见~ 👋 （文章结束）