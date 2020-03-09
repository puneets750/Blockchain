# Blockchain

To send raw private tx, you need to get the hash of your tx, sign with any private key and send it using any sendRawRequest or sendRawPrivateTransaction API. 

 

Using this feature users can send private transactions without connecting to any account or simply using any random key to sign the transaction. 

Private Transaction is sent to an Account that holds Contract code. 

User sends encoded abi of function call to the tessera by calling storeraw api with public key of node. 

 
when user call storeraw api with encoded abi of function call. 

1. tessera passes this payload to enclave, enclave generates symmetric key and nonce. 

      sym ; 

      nonce; 

2. encrypting the Transaction payload and Nonce with the symmetric key from 1.  

      sym(payload); 

      sym(nonce); 

3. calculates the SHA-512 hash of the encrypted payload from 2. 

      var # = SHA-512(sym(payload)); 

4. iterates through the list of Transaction recipients, and encrypts the symmetric key from 1. with the recipient's public key 

      public(sym); 

5. sends all data back to tessera 

6. tessera stores encrypted payload and encrypted sym key via public key with index as hash. 

      M[#] = {sym(payload), public(sym)}; 

7.  tessera sends back the sym(payload). 

 
using this # as data a raw transaction is created signed with any private key. 

Tis tx is sent to the quorum node , quorum node passes this tx to the tessera (private tx). 

Using the hash tessera checks that it is having the corresponding value in its DS( M[#] = {sym(payload), public(sym)}) 

decryptes the tx and submits it to the evm. 

here is the complete working example 

 

const Web3 = require("web3"); 

const web3 = new Web3( 
   new Web3.providers.HttpProvider("http://localhost:9000")// quorum node endpoints 
   ); 

var extendEth = require("./node_modules/quorum-js/lib/index.js"); 

var ABI = [{"constant":true,"inputs":[],"name":"storedData","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"retVal","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[{"name":"initVal","type":"uint256"}],"payable":false,"type":"constructor"}]; 

var data = web3.eth.abi.encodeFunctionCall({ name: 'set', type: 'function',inputs: [{type: 'uint256',name: 'x'}]}, ['19']); 

//this encoded function call will be sent to storeraw api as bas64 payload 

console.log("encoded abi"); 

console.log(data); 
console.log(data); 

const quorumjs   = require("quorum-js"); 
  
extendEth.extend(web3,"eth"); 
var Tx = require('ethereumjs-tx').Transaction; 

const txnManager = quorumjs.RawTransactionManager(web3, { 
  publicUrl: "http://localhost:9000",  //quorum endpoints 
  privateUrl: "http://localhost:9081" //tessera endpoint 
}); 

 

txnManager.storeRawRequest(data, "Wd+SHPa7qj4sgwuS6RMg592Ch5imu1OK+RVSPFbg+Aw=").then(function (datah){ 

 var rawTx = { 

   nonce: '0x0', 

   gasPrice: '0x0', 

   gasLimit: '0x47b760', 

   isPrivate: true, 

   to: '0xc3db4f6af243130aa290fe2a346158d04c44382c',//address of private contract 

   value: `0x${(0).toString(16)}`, 

   // This data should be the hex value of the hash returned by Quorum's privacy transaction manager after invoking storeraw api 

   data: '0x' + datah 

	} 

  var tx = new Tx(rawTx); 
  tx.sign(new Buffer("8d002d17cfe933d4a37b0b1cea3ecc945e25ab156d6854d1d5c65f9abe60025e",'hex'));//any private key to sign the tx 

  var serializedTx = tx.serialize(); 

   web3.eth.sendRawPrivateTransaction('0x' + serializedTx.toString('hex'), {privateFor: ["Wd+SHPa7qj4sgwuS6RMg592Ch5imu1OK+RVSPFbg+Aw="]}, function(err, hash) { 

   if (!err) 
       console.log(hash); // "0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385" 

                     }); 

 }).catch(function (err) { 
   console.log("error"); 
   console.log(err); 
  }); 

 
