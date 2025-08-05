## go-sqlcipher

[![GoDoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](http://godoc.org/github.com/mutecomm/go-sqlcipher) [![CI](https://github.com/mutecomm/go-sqlcipher/workflows/CI/badge.svg)](https://github.com/mutecomm/go-sqlcipher/actions)

### My Changelog

Add these pragmas:

- _pragma_cipher_compatibility
- _pragma_kdf_iter
- _pragma_cipher_kdf_algorithm
- _pragma_cipher_hmac_algorithm

So you can open an encrypted database like this:

```go
dbPath := "EnMicroMsg.db"
pwd := url.QueryEscape("your key")
dbname := fmt.Sprintf("%s?_key=%s&_pragma_cipher_compatibility=1", dbPath, pwd)
db, err := sql.Open("sqlite3", dbname)
if err != nil {
    fmt.Printf("open sqlite3 error: %v\n", err)
}
defer db.Close()
if err != nil {
    fmt.Printf("error: %v\n", err)
}
cur, err := db.Prepare("select value from userinfo where id = 2;")
if err != nil {
    fmt.Printf("select userinfo error: %v\n", err)
}
var wxid string
err = cur.QueryRow().Scan(&wxid)
if err != nil {
    fmt.Printf("select userinfo error: %v\n", err)
}
fmt.Printf("userinfo: %v\n", wxid)
```

Of course, you can decrypt a database:

```go
dbPath := "FTS5IndexMicroMsg_encrypt.db"
key := url.QueryEscape("your key")
dbName := fmt.Sprintf("%s?_key=%s&_pragma_kdf_iter=64000&_pragma_cipher_kdf_algorithm=PBKDF2_HMAC_SHA1&_pragma_cipher_hmac_algorithm=HMAC_SHA1", dbPath, key)
db, err := sql.Open("sqlite3", dbName)
if err != nil {
	t.Fatal(err)
}
// create a decrypted database
decCmd := fmt.Sprintf("ATTACH DATABASE 'FTS_dec.db' AS FTS_dec KEY '';SELECT sqlcipher_export('FTS_dec');DETACH DATABASE FTS_dec;")
_, err = db.Exec(decCmd)
if err != nil {
	t.Fatal(err)
}
```

### Description

Self-contained Go sqlite3 driver with an AES-256 encrypted sqlite3 database
conforming to the built-in database/sql interface. It is based on:

- Go sqlite3 driver: https://github.com/mattn/go-sqlite3 (master)
- SQLite extension with AES-256 codec: https://github.com/sqlcipher/sqlcipher (4.6.1)
- AES-256 implementation from: https://github.com/libtom/libtomcrypt (develop)

SQLite itself is part of SQLCipher.

### Incompatibilities of SQLCipher

The version tags of go-sqlcipher are the same as for SQLCipher.

**SQLCipher 4.x is incompatible with SQLCipher 3.x!**

go-sqlcipher does not implement any migration strategies at the moment.
So if you upgrade a major version of go-sqlcipher, you yourself are responsible
to upgrade existing database files.

See [migrating databases](https://www.zetetic.net/sqlcipher/sqlcipher-api/#Migrating_Databases) for details.

To upgrade your Go code to the 4.x series, change the import path to

    "github.com/WXjzcccc/go-sqlcipher"

### Installation

This package can be installed with the go get command:

    go get github.com/WXjzcccc/go-sqlcipher


### Documentation

To create and open encrypted database files use the following DSN parameters:

```go
key := "2DD29CA851E7B56E4697B0E1F08507293D761A05CE4D1B628663F411A8086D99"
dbname := fmt.Sprintf("db?_key=x'%s'&_pragma_cipher_page_size=4096", key)
db, _ := sql.Open("sqlite3", dbname)
```

`_key` is the hex encoded 32 byte key (must be 64 characters long).
`_pragma_cipher_page_size` is the page size of the encrypted database (set if
you want a different value than the default size).

```go
key := url.QueryEscape("secret")
dbname := fmt.Sprintf("db?_key=%s&_pragma_cipher_page_size=4096", key)
db, _ := sql.Open("sqlite3", dbname)
```

This uses a passphrase directly as `_key` with the key derivation function in
SQLCipher. Do not forget the `url.QueryEscape()` call in your code!

See also [PRAGMA key](https://www.zetetic.net/sqlcipher/sqlcipher-api/#PRAGMA_key).

API documentation can be found here:
http://godoc.org/github.com/mutecomm/go-sqlcipher

Use the function
[sqlite3.IsEncrypted()](https://godoc.org/github.com/mutecomm/go-sqlcipher#IsEncrypted)
to check whether a database file is encrypted or not.

Examples can be found under the `./_example` directory


### License

The code of the originating packages is covered by their respective licenses.
See [LICENSE](LICENSE) file for details.
