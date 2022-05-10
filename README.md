# **Upper: A Client-Side Encrypted Image Host**

_Upper is a simple host that client-side encrypts images, text,... and stores them without the server knowing about the contents.
It has the ability to view images, text with syntax highlighting, short videos, and arbitrary binaries as downloadables._

## **GETTING STARTED**

To install and run the server:

    apt-get install nodejs
    git clone https://github.com/trndk/7Up
    cd Upper
    cd server
    npm install
    node server.js

Client configuration is done through the [`server.conf`](https://github.com/trndk/7Up/server/server.conf) file:

- `listen` is an `address:port`-formatted string, where either one are optional. Some examples include `":9000"` to listen on any interface, port 9000; `"127.0.0.1"` to listen on localhost port 80; `"1.1.1.1:8080"` to listen on 1.1.1.1 port 8080; or even `""` to listen on any interface, port 80.

- `api_key` is a very basic security measure, requiring any client making an upload to know this key. This doesn't seem very useful and should be revamped; replace it with HTTP auth maybe?

- `delete_key` is a key used to secure the deletion keys. Set this to something that only the server knows.

- `maximum_file_size` is the largest file, in bytes, that's allowed to be uploaded to the server. The default here is a decimal 50MB.

- There are three additional sections in the configuration file: `http`, `https` and `cloudflare-cache-invalidate`. The first two are fairly self-explanitory (and at least one must be enabled).

- `cloudflare-cache-invalidate` is disabled by default and only useful if you choose to run the Up1 server behind Cloudflare. When this section is enabled, it ensures that when an upload is deleted, Cloudflare doesn't hold on to copies of the upload on its edge servers by sending an API call to invalidate it.

## **HOW IT WORKS?**

Before an image is uploaded, a "seed" is generated. This seed can be any length (Default: 25 characters). The seed is then run through SHA512, giving the AES key in bytes 0-256, the CCM IV in bytes 256-384, and the server's file identifier in bytes 384-512. Using this output, the image data is encrypted using AES key and IV using SJCL's AES-CCM methods, and sent to the server with an identifier. Within the encryption, there is also a prepended JSON object that contains metadata. The (decrypted) blob format starts with 2 bytes denoting the JSON character length, the JSON data itself, and then the file data at the end.

Image deletion functionality is also available. When an image is uploaded, a delete token is returned. Sending this delete token back to the server will delete the image. On the server side, `HMAC-SHA256(static_delete_key, identifier)` is used, where the key is a secret on the server.

## **CONTRIBUTORS**

- Original code by [Upload](https://github.com/upload/)
- Recoded by [trndk](https://github.com/trndk)

Thank you for reading and have a nice day!
