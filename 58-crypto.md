# 58 - Cryptography Basics

> Hashing, encryption, and secure random generation in Go.

---

## 📌 What You'll Learn

- Hashing (MD5, SHA256, etc.)
- HMAC for message authentication
- Password hashing with bcrypt
- Encryption/Decryption
- Secure random generation

---

## 🔐 Hashing

```go
data := "Hello, World!"

// MD5 (checksum only, NOT secure)
md5Hash := md5.Sum([]byte(data))
fmt.Printf("%x\n", md5Hash)
// Output: 65a8e27d8879283831b664bd8b7f0ad4

// SHA256 (secure)
sha256Hash := sha256.Sum256([]byte(data))
fmt.Printf("%x\n", sha256Hash)

// SHA512
sha512Hash := sha512.Sum512([]byte(data))
fmt.Printf("%x\n", sha512Hash)

// Streaming hash for large data
hasher := sha256.New()
io.WriteString(hasher, "Part 1")
io.WriteString(hasher, "Part 2")
result := hasher.Sum(nil)
// Use crypto/subtle.ConstantTimeCompare for secure comparison
```

---

## 🔑 HMAC (Message Authentication)

```go
secret := []byte("my-secret-key")
message := []byte("Important message")

mac := hmac.New(sha256.New, secret)
mac.Write(message)
signature := mac.Sum(nil)

// Verify
mac2 := hmac.New(sha256.New, secret)
mac2.Write(message)
expectedMAC := mac2.Sum(nil)

if hmac.Equal(signature, expectedMAC) {
    fmt.Println("Valid signature!")
}
// Tampered message produces different MAC
```

---

## 🔒 Password Hashing (bcrypt)

```go
password := "mySecretPassword123"

hash, _ := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
// Output: $2a$10$... (varies each run - different salt)

err := bcrypt.CompareHashAndPassword(hash, []byte(password))
// err == nil if correct

bcrypt.CompareHashAndPassword(hash, []byte("wrongPassword"))
// err != nil

// Cost: MinCost=4 (fast), DefaultCost=10 (balanced), MaxCost=31 (slow)
```

---

## 🎲 Secure Random Generation

```go
// Random bytes
bytes := make([]byte, 32)
rand.Read(bytes)
fmt.Printf("%x\n", hex.EncodeToString(bytes))

// Random token (URL-safe)
token := base64.URLEncoding.EncodeToString(make([]byte, 32))

// Random integer
n, _ := rand.Int(rand.Reader, big.NewInt(100))
// Output: 0-99

// UUID v4
uuid := make([]byte, 16)
rand.Read(uuid)
uuid[6] = (uuid[6] & 0x0f) | 0x40
uuid[8] = (uuid[8] & 0x3f) | 0x80
// NEVER use math/rand for security!
```

---

## 🔐 AES Encryption

```go
key := []byte("32-byte-long-key-for-aes-256!!")
plaintext := "Secret message!"

encrypted, _ := encrypt([]byte(plaintext), key)
decrypted, _ := decrypt(encrypted, key)
// Output: decrypted == plaintext

// Key must be 16, 24, or 32 bytes (AES-128, AES-192, AES-256)
// Use random IV per encryption (included in ciphertext)
```

---

## 🎯 Key Takeaways

1. **MD5/SHA1** - NOT for security, only checksums
2. **SHA256/SHA512** - Secure hashing
3. **HMAC** - Message authentication with shared secret
4. **bcrypt** - Password hashing (use this!)
5. **crypto/rand** - Secure randomness (NOT math/rand)
6. **AES** - Symmetric encryption
7. **Always use constant-time comparison** for secrets

---

## 🎉 Tutorial Complete!

You've now covered **58 comprehensive chapters** on Go!

**Review the full tutorial:** [README](./README.md)
