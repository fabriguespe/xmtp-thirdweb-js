---
slug: thirdbweb-wallet-upload
hide_table_of_contents: true
title: "Introducing Wallet SDK and Sending remote storage"
date: 2023-05-12
authors: eng
image: "https://blog.thirdweb.com/content/images/size/w2000/2023/05/How-to-use-smart-wallet-.png"
description: "Sending remote attachments with XMTP"
tags:
- Content Types
- SDKs
- Media
- Developers
- WalletSDK
- Thirdweb
---

### Introduction

Building a great 'Connect Wallet' flow is the hardest part of Web3. Luckily our friends at third wallet have developed a great SDK to make it easy for you to build a great wallet experience for your users.

### Concepts

#### Wallet SDK
A development kit which gives devs access the largest catalog of wallets, from custodial to MPC to smart contracts. [Read more](https://twitter.com/thirdweb/status/1654191962751389697)

#### Content Types
Content types are a way to describe the *type* of *content* a message contains on XMTP. Out of the box, XMTP's SDKs support one content type: `text`. [Read more](https://xmtp.org/docs/dev-concepts/content-types)

### Demo App

In this article, I will be showing you how to create a NodeJS script that check out balances and interacts with the Lending Pool contract of AAVE.

[Checkout the GitHub repo](.) | [View Live Code](.)

#### What we will learn:
- Set up Thirwallet ConnectWallet button
- Sign in with XMTP
- Load conversation
- Send message
- Send remote attachment
- Receive attachments
- 

### Getting Started
To get started we'll first create and configure the Next.js application.

To generate a new Next.js app, run the following command from your terminal:

```
npx create-next-app xmtp-thirdweb

✔ Would you like to use TypeScript with this project? Yes
✔ Would you like to use ESLint with this project? Yes
✔ Would you like to use Tailwind CSS with this project?  Yes
✔ Would you like to use `src/` directory with this project? No
✔ Use App Router (recommended)? Yes
✔ Would you like to customize the default import alias? No
```

Next, change into the new directory and install these dependencies for using XMTP and Thirdweb:

```
npm install @thirdweb-dev/react @thirdweb-dev/sdk @xmtp/xmtp-js xmtp-content-type-remote-attachment 
```

#### Set up Thirwallet ConnectWallet button
![](./media/xmtp-thirdweb/screen1.png)

Lets start with wrapping the app with ThirdwebProvider and then using the ConnectWallet component to connect to the wallet.


```tsx
<ThirdwebProvider activeChain="goerli">
        <Home/>
</ThirdwebProvider>
```
```tsx
//Just one line of code to connect to wallet
<ConnectWallet theme="light" />
```
```tsx
//After logging in, we can use thirweb hooks to see the wallet
const address = useAddress();
const signer = useSigner();
```

That's it! Then we can follow up with the signing to XMTP

#### Sign in with XMTP
We are going to create a new instance of XMTP and we are going to register the content types we are going to use in our chat app.

```tsx
// Function to initialize the XMTP client
const initXmtp = async function () {
  // Create the XMTP client
  const xmtp = await Client.create(signer, { env: "production" });
  // Register the codecs. AttachmentCodec is for local attachments (<1MB)
  xmtp.registerCodec(new AttachmentCodec());
  //RemoteAttachmentCodec is for remote attachments (>1MB) using thirdweb storage
  xmtp.registerCodec(new RemoteAttachmentCodec());
  //Create or load conversation with Gm bot
  newConversation(xmtp,PEER_ADDRESS);
  // Set the XMTP client in state for later use
  setXmtpConnected(!!xmtp.address);
}
```

#### Load messages

In this case we are going to use our GM Bot and we are going to use the XMTP instance for creating the conversation and in case it exists it will bring its message history.

```tsx
const newConversation = async function (xmtp,addressTo) {
  const conversation = await xmtp.conversations.newConversation(addressTo);
  convRef.current = conversation;
  const messages = await conversation.messages();
  setMessages(messages);
};
  ```


#### Send messages

Sending text requires no codec or encryption. We can just send the message as is.

```tsx
const onSendMessage = async (value) => {
  return convRef.send(value);
};
```
Small attachments below 1MB can be sent using the AttachmentCodec. The codec will automatically encrypt the attachment and upload it to the XMTP network.

```tsx
// Function to handle sending a small file attachment
const handleSmallFile = async () => {
  const blob = new Blob([image], { type: "image/png" });
  let imgArray = new Uint8Array(await blob.arrayBuffer());

  const attachment = {
    filename: image.name,
    mimeType: 'image/png',
    data: imgArray
  };
  await convRef.send(attachment, { contentType: ContentTypeAttachment });
};
```

#### Send remote attachment
Large attachments above 1MB can be sent using the RemoteAttachmentCodec. The codec will automatically encrypt the attachment and upload it to the Thirdweb network.

```tsx
// Function to handle sending a large file attachment
const handleLargeFile = async (file) => {
  setIsLoading(true);

  setLoadingText("Uploading to ThirdWeb Storage...");
  const uploadUrl = await upload({
    data: [file],
    options: { uploadWithGatewayUrl: true, uploadWithoutDirectory: true },
  });
  setLoadingText(uploadUrl[0]);

  const attachment = {
    filename: file.name,
    mimeType: 'image/png',
    data: new TextEncoder().encode(file.name)
  };
  
  const encryptedAttachment = await RemoteAttachmentCodec.encodeEncrypted(
    attachment, 
    new AttachmentCodec()
  );

  const remoteAttachment = {
    url: uploadUrl[0],
    contentDigest: encryptedAttachment.digest,
    salt: encryptedAttachment.salt,
    nonce: encryptedAttachment.nonce,
    secret: encryptedAttachment.secret,
    scheme: "https://",
    filename: attachment.filename,
    contentLength: attachment.data.byteLength,
  };

  setLoadingText("Sending...");
  await convRef.send(remoteAttachment, {
    contentType: ContentTypeRemoteAttachment,
    contentFallback: file.name
  });
};
```


#### Receive attachments

In our parent component we are going to add a listener that will fetch new messages from a stream.

```tsx
// Function to stream new messages in the conversation
const streamMessages = async () => {
  const newStream = await convRef.current.streamMessages();
  for await (const msg of newStream) {
    const exists = messages.find(m => m.id === msg.id);
    if (!exists) {
      setMessages(prevMessages => {
        const msgsnew = [...prevMessages, msg];
        return msgsnew;
      });
    }
  }
};
streamMessages();
```

Later we will render this messages in the child component using a Blob for attachments.

```tsx
if (message.contentType.sameAs(ContentTypeAttachment)) {
  // Handle ContentTypeAttachment
  return objectURL(message.content);
}
// Function to render a local attachment as an image
const objectURL = (attachment) => {
  const blob = new Blob([attachment.data], { type: attachment.mimeType });
  return <img src={URL.createObjectURL(blob)} width={200} className="imageurl" alt={attachment.filename} />;
};
```
With remote storage we are just going to point to the url in our html

```tsx
if (message.contentType.sameAs(ContentTypeRemoteAttachment)) {
  // Handle ContentTypeRemoteAttachment
  return remoteURL(message.content);
}
// Function to render a remote attachment URL as an image
const remoteURL = (attachment) => {
  return <img src={attachment.url} width={200} className="imageurl" alt={attachment.filename} />;
};
```
