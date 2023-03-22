# sql_exporter
modifications to https://github.com/free/sql_exporter

Added the capability to encrypt the dsn. The orginal code exposes the login credentials in the dsn. The encryption function is adapted from https://gist.github.com/humamfauzi/a29ea50edeb175e2e8a9e3456b91fe36

The secrey key can be set using the environment variable SQLEXPORTER_KEY. The challenge is to the user on how to keep the environment of the sql_exporter process secure. 

The key is a 64 byte HEX.

```
export SQLEXPORTER_KEY=badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbada
```

Alternatively you can bake the key into the code itself (as commented in the code).

To generate the code, the easiest way is to paste the following into https://go.dev/play/. Make sure you change the key and dsn to your own. In the example below, the dsn is sqlserver://username:password@my.target.host:1433/ and the key is "badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbada".

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/base64"
	"encoding/hex"
	"fmt"
	"io"
)

func main() {
	exampleString := "sqlserver://username:password@my.target.host:1433/"
	toBinary := []byte(exampleString)

	// Secret Key; DO NOT USE THIS IN REAL APPLICATION
	key, _ := hex.DecodeString("badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbada")
	block, err := aes.NewCipher(key)
	if err != nil {
		panic(err)
	}

	gcm, err := cipher.NewGCM(block)
	if err != nil {
		panic(err)
	}

	nonce := make([]byte, gcm.NonceSize())
	nonce3 := make([]byte, 23)
	if _, err = io.ReadFull(rand.Reader, nonce); err != nil {
		fmt.Println(err)
	}
	if _, err = io.ReadFull(rand.Reader, nonce3); err != nil {
		fmt.Println(err)
	}

	// nonce should be append in the seal for decryption
	// it also could use different kind of header for identification
	result := gcm.Seal(nonce, nonce, toBinary, nil)

	encryptedString := base64.StdEncoding.EncodeToString(result)
	fmt.Println("ENCRYPTED", encryptedString)

}
```

Once you have the encrypted string, you configure the dsn as below.

```
target:
  data_source_name: 'encrypted://Yoca7r/sjkIUzZSFKGTjNxCjWZIXvanutcC9tti7AqwvAM7PrkkmOl5fZcMcfnGYrwyPLFuudH+AMmN6Z8pQPoBWPGwmEHyQ1Vu8wPMu'
```
