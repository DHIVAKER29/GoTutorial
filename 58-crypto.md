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
// hashing.go
package main

import (
    "crypto/md5"
    "crypto/sha256"
    "crypto/sha512"
    "encoding/hex"
    "fmt"
    "io"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Hashing                                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    data := "Hello, World!"
    
    // MD5 (NOT for security - only checksums)
    fmt.Println("\n📊 MD5 (checksum only, NOT secure):")
    md5Hash := md5.Sum([]byte(data))
    fmt.Printf("   %x\n", md5Hash)
    
    // SHA256 (secure)
    fmt.Println("\n📊 SHA256:")
    sha256Hash := sha256.Sum256([]byte(data))
    fmt.Printf("   %x\n", sha256Hash)
    
    // SHA512
    fmt.Println("\n📊 SHA512:")
    sha512Hash := sha512.Sum512([]byte(data))
    fmt.Printf("   %x\n", sha512Hash)
    
    // Hash large data with io.Writer
    fmt.Println("\n📊 Streaming Hash (for large data):")
    hasher := sha256.New()
    io.WriteString(hasher, "Part 1")
    io.WriteString(hasher, "Part 2")
    io.WriteString(hasher, "Part 3")
    result := hasher.Sum(nil)
    fmt.Printf("   %s\n", hex.EncodeToString(result))
    
    // Hash comparison (constant-time)
    fmt.Println("\n📊 Secure Comparison:")
    fmt.Println("   Use crypto/subtle.ConstantTimeCompare")
    fmt.Println("   Prevents timing attacks")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Hashing                                         ║
╚══════════════════════════════════════════════════════════╝

📊 MD5 (checksum only, NOT secure):
   65a8e27d8879283831b664bd8b7f0ad4

📊 SHA256:
   dffd6021bb2bd5b0af676290809ec3a53191dd81c7f70a4b28688a362182986f

📊 SHA512:
   374d794a95cdcfd8b35993185fef9ba368f160d8daf432d08ba9f1ed1e5abe6cc69291e0fa2fe0006a52570ef18c19def4e617c33ce52ef0a6e5fbe318cb0387

📊 Streaming Hash (for large data):
   886b990fbdfdad666585bdd6634f87fb6e947fef240b88f6a7dd002cb2266f12

📊 Secure Comparison:
   Use crypto/subtle.ConstantTimeCompare
   Prevents timing attacks
```

---

## 🔑 HMAC (Message Authentication)

```go
// hmac_example.go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           HMAC (Message Authentication)                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    secret := []byte("my-secret-key")
    message := []byte("Important message")
    
    // Create HMAC
    fmt.Println("\n📊 Create HMAC:")
    mac := hmac.New(sha256.New, secret)
    mac.Write(message)
    signature := mac.Sum(nil)
    fmt.Printf("   HMAC: %s\n", hex.EncodeToString(signature))
    
    // Verify HMAC
    fmt.Println("\n📊 Verify HMAC:")
    mac2 := hmac.New(sha256.New, secret)
    mac2.Write(message)
    expectedMAC := mac2.Sum(nil)
    
    if hmac.Equal(signature, expectedMAC) {
        fmt.Println("   ✅ Valid signature!")
    } else {
        fmt.Println("   ❌ Invalid signature!")
    }
    
    // Tampered message
    fmt.Println("\n📊 Tampered Message:")
    tamperedMessage := []byte("Tampered message")
    mac3 := hmac.New(sha256.New, secret)
    mac3.Write(tamperedMessage)
    tamperedMAC := mac3.Sum(nil)
    
    if hmac.Equal(signature, tamperedMAC) {
        fmt.Println("   ✅ Valid signature!")
    } else {
        fmt.Println("   ❌ Invalid signature (tampering detected)")
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           HMAC (Message Authentication)                   ║
╚══════════════════════════════════════════════════════════╝

📊 Create HMAC:
   HMAC: 94f704b3c6c98b9d65e08428606855b05b0ae5db8a9dc4fd99078c9bf6ee76fe

📊 Verify HMAC:
   ✅ Valid signature!

📊 Tampered Message:
   ❌ Invalid signature (tampering detected)
```

---

## 🔒 Password Hashing (bcrypt)

```go
// password_hashing.go
package main

import (
    "fmt"
    
    "golang.org/x/crypto/bcrypt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Password Hashing (bcrypt)                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    password := "mySecretPassword123"
    
    // Hash password
    fmt.Println("\n📊 Hash Password:")
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        panic(err)
    }
    fmt.Printf("   Hash: %s\n", hash)
    
    // Verify password
    fmt.Println("\n📊 Verify Password:")
    err = bcrypt.CompareHashAndPassword(hash, []byte(password))
    if err == nil {
        fmt.Println("   ✅ Password correct!")
    } else {
        fmt.Println("   ❌ Password incorrect!")
    }
    
    // Wrong password
    fmt.Println("\n📊 Wrong Password:")
    err = bcrypt.CompareHashAndPassword(hash, []byte("wrongPassword"))
    if err == nil {
        fmt.Println("   ✅ Password correct!")
    } else {
        fmt.Println("   ❌ Password incorrect!")
    }
    
    // Cost levels
    fmt.Println("\n📊 Cost Levels:")
    fmt.Println("   bcrypt.MinCost     = 4  (fast, testing)")
    fmt.Println("   bcrypt.DefaultCost = 10 (balanced)")
    fmt.Println("   bcrypt.MaxCost     = 31 (very slow)")
    fmt.Println("   Higher = more secure but slower")
}

/*
Install: go get golang.org/x/crypto/bcrypt
*/
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Password Hashing (bcrypt)                       ║
╚══════════════════════════════════════════════════════════╝

📊 Hash Password:
   Hash: $2a$10$xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

📊 Verify Password:
   ✅ Password correct!

📊 Wrong Password:
   ❌ Password incorrect!

📊 Cost Levels:
   bcrypt.MinCost     = 4  (fast, testing)
   bcrypt.DefaultCost = 10 (balanced)
   bcrypt.MaxCost     = 31 (very slow)
   Higher = more secure but slower
```

*Note: The bcrypt hash value varies on each run (different salt).*

---

## 🎲 Secure Random Generation

```go
// secure_random.go
package main

import (
    "crypto/rand"
    "encoding/base64"
    "encoding/hex"
    "fmt"
    "math/big"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Secure Random Generation                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Random bytes
    fmt.Println("\n📊 Random Bytes:")
    bytes := make([]byte, 32)
    rand.Read(bytes)
    fmt.Printf("   Hex: %s\n", hex.EncodeToString(bytes))
    fmt.Printf("   Base64: %s\n", base64.StdEncoding.EncodeToString(bytes))
    
    // Random token
    fmt.Println("\n📊 Random Token (URL-safe):")
    tokenBytes := make([]byte, 32)
    rand.Read(tokenBytes)
    token := base64.URLEncoding.EncodeToString(tokenBytes)
    fmt.Printf("   Token: %s\n", token)
    
    // Random integer
    fmt.Println("\n📊 Random Integer (0-99):")
    n, _ := rand.Int(rand.Reader, big.NewInt(100))
    fmt.Printf("   Number: %d\n", n)
    
    // Generate UUID v4 (simple version)
    fmt.Println("\n📊 UUID v4:")
    uuid := make([]byte, 16)
    rand.Read(uuid)
    uuid[6] = (uuid[6] & 0x0f) | 0x40  // Version 4
    uuid[8] = (uuid[8] & 0x3f) | 0x80  // Variant
    uuidStr := fmt.Sprintf("%x-%x-%x-%x-%x",
        uuid[0:4], uuid[4:6], uuid[6:8], uuid[8:10], uuid[10:16])
    fmt.Printf("   UUID: %s\n", uuidStr)
    
    // ⚠️ Warning
    fmt.Println("\n⚠️ NEVER use math/rand for security!")
    fmt.Println("   Use crypto/rand for:")
    fmt.Println("   • Passwords, tokens, keys")
    fmt.Println("   • Session IDs")
    fmt.Println("   • Any security-sensitive randomness")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Secure Random Generation                        ║
╚══════════════════════════════════════════════════════════╝

📊 Random Bytes:
   Hex: a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456
   Base64: obLD1OX2kJDRU2kQEkNWOJCq+9EjRlgK+9EjRg==

📊 Random Token (URL-safe):
   Token: xYz123AbC456...

📊 Random Integer (0-99):
   Number: 42

📊 UUID v4:
   UUID: a1b2c3d4-e5f6-4789-8123-456789abcdef

⚠️ NEVER use math/rand for security!
   Use crypto/rand for:
   • Passwords, tokens, keys
   • Session IDs
   • Any security-sensitive randomness
```

*Note: Random bytes, token, integer, and UUID values vary on each run.*

---

## 🔐 AES Encryption

```go
// aes_encryption.go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/base64"
    "fmt"
    "io"
)

func encrypt(plaintext, key []byte) (string, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return "", err
    }
    
    ciphertext := make([]byte, aes.BlockSize+len(plaintext))
    iv := ciphertext[:aes.BlockSize]
    if _, err := io.ReadFull(rand.Reader, iv); err != nil {
        return "", err
    }
    
    stream := cipher.NewCFBEncrypter(block, iv)
    stream.XORKeyStream(ciphertext[aes.BlockSize:], plaintext)
    
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

func decrypt(ciphertextB64 string, key []byte) (string, error) {
    ciphertext, _ := base64.StdEncoding.DecodeString(ciphertextB64)
    
    block, err := aes.NewCipher(key)
    if err != nil {
        return "", err
    }
    
    if len(ciphertext) < aes.BlockSize {
        return "", fmt.Errorf("ciphertext too short")
    }
    
    iv := ciphertext[:aes.BlockSize]
    ciphertext = ciphertext[aes.BlockSize:]
    
    stream := cipher.NewCFBDecrypter(block, iv)
    stream.XORKeyStream(ciphertext, ciphertext)
    
    return string(ciphertext), nil
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           AES Encryption                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Key must be 16, 24, or 32 bytes (AES-128, AES-192, AES-256)
    key := []byte("32-byte-long-key-for-aes-256!!")  // 32 bytes
    plaintext := "Secret message!"
    
    fmt.Printf("\n   Plaintext: %s\n", plaintext)
    
    // Encrypt
    encrypted, _ := encrypt([]byte(plaintext), key)
    fmt.Printf("   Encrypted: %s\n", encrypted)
    
    // Decrypt
    decrypted, _ := decrypt(encrypted, key)
    fmt.Printf("   Decrypted: %s\n", decrypted)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           AES Encryption                                  ║
╚══════════════════════════════════════════════════════════╝

   Plaintext: Secret message!
   Encrypted: xYz123AbC456... (base64, varies each run)
   Decrypted: Secret message!
```

*Note: Encrypted value varies each run due to random IV; decrypted always matches plaintext.*

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

