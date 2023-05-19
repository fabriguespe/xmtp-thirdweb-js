### Introduction

Creating an effective 'Connect Wallet' flow is a challenging aspect of Web3 development. Thankfully, the team at Thirdweb has developed an excellent SDK, simplifying this process and enabling a superior wallet experience for your users.

<!--truncate-->

### Concepts

#### Thirdweb WalletSDK
The WalletSDK is a development kit that grants developers access to a comprehensive selection of wallets, ranging from custodial to MPC to smart contracts.
[Read more](https://twitter.com/thirdweb/status/1654191962751389697)

#### XMTP Content-Types
Content types are a way to describe the *type* of *content* a message contains on XMTP. Out of the box, XMTP's SDKs support one content type: `text`. 
[Read more](https://xmtp.org/docs/dev-concepts/content-types)

#### Thirdweb storage
StorageSDK handles the complexities of decentralized file management for you. No need to worry about fetching from multiple IPFS gateways, handling file and metadata upload formats, etc.
[Read more](https://thirdweb.com/storage)

### Demo App

This repository demonstrates the implementation of these concepts within a simple chat app.

[GitHub repo](https://github.com/fabriguespe/xmtp-thirdweb-js) 

Next, run the app:
```tsx
npm run dev
//or
yarn dev
```