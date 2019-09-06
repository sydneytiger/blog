I have recently implemented an AES GCM mode on login credential of a online banking project. Here is the take away.

### What happen in client side
1. Client side send an initialisation request to the server.
2. Server responses a public RSA key (asymmetric crypto key).
3. Client side generate a random session key (symmetric crypto key) for AES encryption in GCM mode.
4. For every single field to encrypt, client side encrypt data using the session key and a **unique random IV/nonce for each individual field.** The nonce and encrypted data(ciphertext) will each be base64 encoded and added to the payload in the format _nonce.ciphertext_.
5. The session key is encrypted by the public key from server and base64 encode the result(cipherkey). The cipherKey is sent via http header of _X-payload-session_.

You may ask why I use temporary symmetric encryption(AES) and asymmetric encrypted(RSA) the symmetric key and send back to server. I could RSA encrypt the data directly and send back without any key. The rationale is that RSA encryption has a limit. A 2048 bit(256byte) RSA key and PKCS#1 padding.is limited to 245 bytes of data. It is ok to send small data like username and password. But RSA will fail when sending a file.

To encrypt multiple fields, we should always use one nonce per field. Theoretically, using same session key and same nonce for multiple fields open a door to session key to be recovered. This approach does not prevent the man-in-middle, but it adds additional layer of protection servers to increase the difficulty of successful attack.


### What happen in server side
1. Server extract _X-payload-session_ from http request header, base64 decode and decrypt it by private key.
2. Split the nonce and ciphertext from http request body, base 64 decode each part and then decrypt the data with the decrypted session key.

Here is the sequence diagram:
![AES encryption diagram](https://user-images.githubusercontent.com/1787825/63017727-6f3e1b80-beda-11e9-8c0c-307c377e40a8.png)
### window.crypto api

Lucky we are, current browsers(Excpet IE) are fully support crypto API. It make our life easier to implement it. The reference Web sites are:
1. [Window.crypto MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/crypto)
2. [SubtleCrypto MDN](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto)

### client side code implementation
```javascript
class Encrypt {
  constructor(publicKey){
    this.pubKey = publicKey;
  }
  
  _convertStringToArrayBufferView(str) {
    const bytes = new Uint8Array(str.length);
    for (let i = 0; i < str.length; i += 1) {
      bytes[i] = str.charCodeAt(i);
    }

    return bytes;
  }
  
  _convertBase64StringToArrayBuffer(b64str) {
    const byteStr = atob(b64str);
    const bytes = this._convertStringToArrayBufferView(byteStr);
    return bytes.buffer;
  }
    
  _convertPemToBinary(pem) {
    const lines = pem.split('\n');
    let encoded = '';
    for (let i = 0; i < lines.length; i += 1) {
      if (lines[i].trim().length > 0 &&
          lines[i].indexOf('-BEGIN') < 0 &&
          lines[i].indexOf('-END') < 0) {
        encoded += lines[i].trim();
      }
    }
    return this._convertBase64StringToArrayBuffer(encoded);
  }
  
  _convertArrayBufferToBase64String(buffer) {
    let binary = '';
    const bytes = new Uint8Array(buffer);
    const len = bytes.byteLength;
    for (let i = 0; i < len; i += 1) {
      binary += String.fromCharCode(bytes[i]);
    }
    return window.btoa(binary);
  }
  
  _encryptWithAESGCMKey(sessionKey, data, nonce) {
    return crypto.subtle.encrypt(
      {
        name: 'AES-GCM',
        iv: nonce,
        tagLength: 128
      },
      sessionKey,
      data
    );
  }
  
  _importPublicKey(pubKey) {
     return crypto.subtle.importKey(
       'spki',
       this._convertPemToBinary(pubKey),
       {
         name: 'RSA-OAEP',
         hash: { name: 'SHA-256' }
       },
       false,
       ['encrypt']
     );
   }
  
  _exportSessionKey(key) {
    return window.crypto.subtle.exportKey(
      'raw',
      key
    );
  }
  
  _rsaEncrypt(data, publicKey) {
    // iv: Is initialization vector. It must be 16 bytes
    const vector = crypto.getRandomValues(new Uint8Array(16));
    let payload = data;

    // if data is not an ArrayBuffer, treat it as a string and convert it into an ArrayBuffer
    if (data.toString().indexOf('ArrayBuffer') === -1) {
      payload = this._convertStringToArrayBufferView(data);
    }

    return crypto.subtle.encrypt(
      {
        name: 'RSA-OAEP',
        iv: vector,
        hash: { name: 'SHA-256' }
      },
      publicKey, 
      payload
    );
  }
  
  async generateSessionKey() {
    const key = await crypto.subtle.generateKey(
      {
        name: 'AES-GCM',
        length: 128
      },
      true,
      ['encrypt', 'decrypt']
    );
    
    return key;
  }
  
  async encrypt(payload, sessionKey){
    const nonce = crypto.getRandomValues(new Uint8Array(12));
    const publicKey = await this._importPublicKey(this.pubKey);
    const exportedSessionKey = await this._exportSessionKey(sessionKey);
    const cipherKey = await this._rsaEncrypt(exportedSessionKey, publicKey);
    const cipherText = await this._encryptWithAESGCMKey(sessionKey, this._convertStringToArrayBufferView(payload), nonce);
    
    return {
      cipherText: `${this._convertArrayBufferToBase64String(nonce)}.${this._convertArrayBufferToBase64String(cipherText)}`,
      cipherKey: this._convertArrayBufferToBase64String(cipherKey)
    };
  }
}
```
### How to use


```javascript
const publicKey = 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlADMZvsn1OcHCHJaytyAOXigJmKNaBW44UK4n4KXzwZpPa7DFDnDJl5nArLuXT+oT28asxwsz2Ocl8JX2qx7aoeS8znel4saYg5j2+9WN97J0XHUMlVE09NvZ6FPNpKQVvNuyyzj9/ll97mkS6LwlUOq3m6tkt2VxXhl38ezv+62jXzgjow5r9vSmy/7TBHbn6oT+XEmn10dJttdRlRCZUZFa6HSc4Kw3jiWsm8Fz46QunoJsWbhZTgApaqhm6asDYcecYl4hh9IVElBIEGyYXBRlehXfZ3JiceDLg3t8rWn7JVvpPW2wWSb1voaTJKXS9L2ClBzADVg5u0RIMFs+QIDAQAB';

const encr = new Encrypt(publicKey);
encr.generateSessionKey()
.then(key => {
  const sessionKey = key;
  console.log('session key: ' + sessionKey);
  return encr.encrypt('Hello world', sessionKey);
})
.then(data => {
  const result = data;
  console.dir(result);
});
/*
cipherKey: "MDUISGBmRoD8uIYFrlInn2VgXVHmD1+gW+La9GWRxbEXik/t0uGw4egop2SC7JVMRJr+wt907mihzZDSh50XYBPpjWeQpxmET2Ne0BU+l4NGuyufK6eH7CjCpRTILTuwJDJHevPRCI5c3otm/j5aOqpQt4uJc+XgbXT/9hOSs9YmAZ5o5mGre9JlM4ljJahJKOxZlpB8VJYA8urm0g9F0QtkHmS3F6lY8cpAcOLts66tdQO9vS1GvRkxyQOlLa8m00Mrygn8iZBbPntmgEKategbYdtHBfgu9wM/BPWd88FNQC+6/kxjvE2CFbFKMHU3XkrWOuMwABSfcjlrNQLIjw=="
cipherText: "5+ZOZTrsarIbubfh.jx+VZlymAbHa17juuH45VHrQj9fdfqN9sDmE"
*/
```