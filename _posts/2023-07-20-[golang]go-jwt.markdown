---
layout: post
title:  "[golang]学习:GO-JWT"
date:   2023-07-20
categories: golang
---

在golang中jwt的运用

## 介绍

JWT (JSON Web Token) 是一种开放标准（RFC 7519），用于在不同实体之间安全地传输信息。它是一种基于 JSON 的令牌，通常由服务器生成，并且可以被用于验证和授权用户的访问权限。

JWT 由三部分组成，使用点号（.）分隔：

- 头部（Header）

包含令牌的类型和签名算法等信息，通常是一个 JSON 对象的形式。

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

上述示例中，alg 表示签名算法，typ 表示令牌的类型为 JWT。

- 载荷（Payload）

包含实际的用户信息或其他相关数据，也是一个 JSON 对象。

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

上述示例中，sub 表示主题（subject），name 表示用户名，admin 表示用户是否是管理员。
**虽然载荷部分也受到保护，也有防篡改，但是这一部分是公共可读的，所以不要把敏感信息存放在JWT内。**

- 签名（Signature）

使用头部和载荷的内容，结合服务器的密钥，通过指定的签名算法生成的签名。签名用于验证令牌的完整性和真实性，确保令牌在传输过程中没有被篡改。

## 工作原理

服务器生成 JWT，并将其发送给客户端。客户端在后续的请求中携带 JWT，通常在请求的头部使用 Authorization 字段进行传递。服务器接收到 JWT 后，可以验证签名，解析载荷，以及根据需要进行授权和认证。

## 优势

JWT 的优点是它具有自包含性，因为所有信息都存储在令牌本身中，避免了在服务器端存储会话信息的需求。另外，JWT 可以在不同的系统和服务之间共享，因为它们使用相同的密钥进行签名和验证。

## 缺陷

- 没有服务器端状态

JWT是无状态的，意味着所有必要的信息都包含在令牌本身中。这在可扩展性方面是有优势的，但如果需要在令牌到期之前撤销或使其无效，就会面临问题，因为没有简单的方法来实现这一点，除非维护一个服务器端的黑名单或额外的基础设施。

- 令牌过期

一旦颁发，JWT在其过期时间之前都是有效的，该过期时间在创建令牌时设置。如果过期时间设置不当或管理不正确，可能会导致安全漏洞，特别是在令牌被拦截或盗取时。

- 敏感数据

避免在JWT负载中存储敏感信息，因为它并没有加密。即使负载进行了Base64Url编码，仍然可以轻松解码，从而可能使敏感信息暴露给攻击者。

- 令牌大小

根据声明的数量和大小，JWT可能会变得相当大，从而增加网络开销，特别是在令牌在客户端和服务器之间频繁传递的情况下。

- 有限的实时控制

由于JWT通常设置为在特定时间后过期，因此对用户会话的实时控制可能会有挑战。要求更短的过期时间可以提高安全性，但也会增加用户频繁登录的次数。

## 示例

- 安装jwt-go

```
go get -u github.com/golang-jwt/jwt
```

- secret

在JWT中，Secret是用于签名和验证令牌的关键。它是一个秘密的字符串，只有服务器知道。当服务器颁发JWT时，会使用Secret对JWT进行签名，以确保令牌在传输过程中没有被篡改。在客户端收到JWT后，客户端可以使用相同的Secret来验证JWT的真实性和完整性，从而确保JWT是合法且未被篡改的。

```
secret := []byte("my secret")
//设置一个secret
```

- 生成Token

定义 Claims 结构体，其中包含 ID 和 Username 字段，还有在 jwt-go 包预定义的 jwt.StandardClaims

```
type Claims struct {
  ID       int
  Username string
  jwt.StandardClaims
}
```

其中jwt.StandardClaims包含：

```

type StandardClaims struct {
  Audience  string `json:"aud,omitempty"`
  ExpiresAt int64  `json:"exp,omitempty"`
  Id        string `json:"jti,omitempty"`
  IssuedAt  int64  `json:"iat,omitempty"`
  Issuer    string `json:"iss,omitempty"`
  NotBefore int64  `json:"nbf,omitempty"`
  Subject   string `json:"sub,omitempty"`
}

```

使用 jwt-go 库根据指定的算法生成 jwt token ，主要用到两个方法：

```

func jwt.NewWithClaims(method jwt.SigningMethod, claims jwt.Claims) *jwt.Token
//jwt.NewWithClaims 方法根据 Claims 结构体创建 Token 示例

func (*jwt.Token).SignedString(key interface{}) (string, error)
//SignedString 方法根据传入的空接口类型参数 key，返回完整的签名令牌

```

- 解析Token

解析JWT Token的意义在于确保数据的真实性和完整性，获取有效的用户信息和权限声明，以及实现安全的身份验证和授权机制。

```

func ParseToken(tokenString string) (claims *MyClaims, err error) {
	// 解析token
	var token *jwt.Token
	claims = new(MyClaims)
	token, err = jwt.ParseWithClaims(tokenString, claims, keyFunc)
	if err != nil {
		return
	}
	if !token.Valid { // 校验token
		err = errors.New("invalid token")
	}
	return
}

```

最后输出

```

token生成成功token is eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOjEyMzQ1NiwiVXNlck5hbWUiOiJ5dXV1dXV1YW4iLCJleHAiOjE2ODk4NTc3MDQsImlzcyI6ImFkbWluI
n0.Mm0vvVvspBldKtVd3WZ-WIiUQa-ALm84heverMa95Ls
token解析成功claims is &{123456 yuuuuuuan { 1689857704  0 admin 0 }}

```