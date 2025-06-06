---
title: How to retrieve a ‘crypt’ password from a config file
draft: false
date: 2024-05-24
tags:
  - howto
  - crypt
  - password
  - config
---

The password that is saved on `crypt` remotes on `~/.config/rclone.conf` is encrypted, but it can be recovered.

Update: Now it can be retrieved using the `rclone reveal` command, which is much easier than the code below.

```bash
rclone reveal u6rMjUm4G1Wn7O88g7R2OPeBchFMNOMkX2PZwcyHegEjX68
```

I've copied some code from the rclone source and added a line to make it easier for people to run it.

Go to `https://play.golang.org/p/IcRYDip3PnE` and replace the string `YOUR PSEUDO-ENCRYPTED PASSWORD HERE` with the actual password that is written in your `~/.config/rclone.conf` file, then click "Run".

```go
package main

import (
  "crypto/aes"
  "crypto/cipher"
  "crypto/rand"
  "encoding/base64"
  "errors"
  "fmt"
  "log"
)

// crypt internals
var (
  cryptKey = []byte{
    0x9c, 0x93, 0x5b, 0x48, 0x73, 0x0a, 0x55, 0x4d,
    0x6b, 0xfd, 0x7c, 0x63, 0xc8, 0x86, 0xa9, 0x2b,
    0xd3, 0x90, 0x19, 0x8e, 0xb8, 0x12, 0x8a, 0xfb,
    0xf4, 0xde, 0x16, 0x2b, 0x8b, 0x95, 0xf6, 0x38,
  }
  cryptBlock cipher.Block
  cryptRand  = rand.Reader
)

// crypt transforms in to out using iv under AES-CTR.
//
// in and out may be the same buffer.
//
// Note encryption and decryption are the same operation
func crypt(out, in, iv []byte) error {
  if cryptBlock == nil {
    var err error
    cryptBlock, err = aes.NewCipher(cryptKey)
    if err != nil {
      return err
    }
  }
  stream := cipher.NewCTR(cryptBlock, iv)
  stream.XORKeyStream(out, in)
  return nil
}

// Reveal an obscured value
func Reveal(x string) (string, error) {
  ciphertext, err := base64.RawURLEncoding.DecodeString(x)
  if err != nil {
    return "", fmt.Errorf("base64 decode failed when revealing password - is it obscured? %w", err)
  }
  if len(ciphertext) < aes.BlockSize {
    return "", errors.New("input too short when revealing password - is it obscured?")
  }
  buf := ciphertext[aes.BlockSize:]
  iv := ciphertext[:aes.BlockSize]
  if err := crypt(buf, buf, iv); err != nil {
    return "", fmt.Errorf("decrypt failed when revealing password - is it obscured? %w", err)
  }
  return string(buf), nil
}

// MustReveal reveals an obscured value, exiting with a fatal error if it failed
func MustReveal(x string) string {
  out, err := Reveal(x)
  if err != nil {
    log.Fatalf("Reveal failed: %v", err)
  }
  return out
}

func main() {
  fmt.Println(MustReveal("YOUR PSEUDO-ENCRYPTED PASSWORD HERE"))
}
```

Output of this code reveal the initial password choice.
