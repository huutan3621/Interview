# Task 1

# Issue 1: Host header đang gửi từ request

```
api-service/src/app/auth/auth.controller.ts
```

- Severity: Critical

- Backend đang nhận host của FE từ request
- Ngay tại đoạn Line 52: const host = req.get('host');
- Backend đang tin tưởng bât kì host header nào, gây lỗi nghiêm trọng, attacker có thể tự gửi request bằng curl, postman, script,...
- Sửa lỗi: dừng ngay đoạn defaultUrl hoặc thêm whitelist như sau:

```

const allowedHosts = ['shopabc.com'];

if (!allowedHosts.includes(host)) {
return defaultUrl;
}
```

---

# Issue 2: Environment Variable không validate kiểu

```
api-service/src/app/auth/auth.controller.ts
```

- Severity: Major

- Ngay tại line 73:

```

maxAge: Number(process.env.MAX_AGE_REFRESH_COOKIE)
```

- Environment variable không được validate
- Nếu .env sai thì:

```

MAX_AGE_REFRESH_COOKIE=abc

Number("abc") = NaN

```

- Sửa lỗi:

```

const maxAge = parseInt(process.env.MAX_AGE_REFRESH_COOKIE ?? '604800000');

res.cookie(TOKEN_TYPE.REFRESH_TOKEN, refresh_token, {
httpOnly: true,
secure: process.env.NODE_ENV === 'production',
sameSite: 'strict',
maxAge: maxAge,
});

```

---

# Issue 3: Security config đang bị phụ thuộc process.env

```

api-service/src/app/auth/auth.controller.ts

```

- Severity: Minor

- Ngay tại line 71:

```

secure: process.env.NODE_ENV === 'production'

```

- Controller phụ thuộc trực tiếp process.env, dẫn đến khó test, config dễ nằm rải rác và sai tên, ...

- Sửa lỗi:

Sử dụng ConfigService để truy cập configuration và dùng constant key để tránh lỗi string.

```

secure: this.configService.get(AppConfigKey.NODE_ENV) === 'production'

export class AppConfigKey {
static readonly NODE_ENV = 'app.nodeEnv';
}

```

---

# Issue 4: Không kiểm tra req.cookies

```

api-service/src/app/auth/auth.controller.ts

```

- Severity: Major

- Ngay tại line 114:

```

const refreshToken = req.cookies['refresh_token'];

```

- Lỗi: Gọi ra trực tiếp req.cookies['refresh_token'] mà không kiểm tra req.cookies có tồn tại hay không

- Sửa lỗi:

```

if (!req.cookies || !req.cookies['refresh_token']) {
throw new UnauthorizedException(AUTH_MESSAGES.INVALID_REFRESH_TOKEN);
}

const refreshToken = req.cookies['refresh_token']
```

# Issue 5: Lộ access token và thông tin nhạy cảm trong url khi đăng nhập với google

```

api-service/src/app/auth/auth.controller.ts

```

- Severity: Critical

- Trường hợp yêu cầu liên kết tài khoản, hệ thống truyền dữ liệu nhạy cảm qua query của url tại line 186:

```

const params = new URLSearchParams({
requiresLinking: 'true',
email: result.email,
googleId: result.googleId,
displayName: result.displayName || '',
avatarUrl: result.avatarUrl || '',
});
return res.redirect(`${frontendUrl}/login?${params.toString()}`);
```

- Trường hợp đăng nhập thành công tại line 207:

```

const params = new URLSearchParams({
success: 'true',
access_token: successResult.access_token,
user: JSON.stringify(successResult.user),
permissions: JSON.stringify(successResult.permissions),
});
return res.redirect(`${frontendUrl}/login?${params.toString()}`);

```

- Có thể lộ thông tin qua lịch sử duyệt web, server log,...Đồng thời lộ cả email, googleId, ...

- Sửa lỗi:
- Set cookie cho refresh token

```

res.cookie(TOKEN_TYPE.REFRESH_TOKEN, successResult.refresh_token, {
httpOnly: true,
secure: process.env.NODE_ENV === 'production',
sameSite: 'lax',
maxAge: Number(process.env.MAX_AGE_REFRESH_COOKIE),
});

```

- Redirect không truyền token

```

return res.redirect(`${frontendUrl}/login?success=true`);

```

- Khi cần account linking

```

if (result.requiresLinking) {
return res.redirect(`${frontendUrl}/login?requiresLinking=true`);
}

```

- FE sau đó gọi API để lấy thông tin

```
- FE sau đó gọi API để lấy thông tin

```
