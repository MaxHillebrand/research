# Externally funded succinct atomic swaps

Based on @RubenSomsen [succinct atomic swaps](https://gist.github.com/RubenSomsen/8853a66a64825716f51b409be528355f).

## Design goal

Creating a **trivial** user experience for sending high anonymity set coins.

**The client receives non-private coins, but can instantly spend high anonymity set coins.**

## Overview

There is a client - server relationship.
The client "outsources" his on-chain privacy to a non-custodial but privacy-trusted third party server.
The client swaps his non-private coins, for high anonymity set coinjoined coins of the server.
Whenever the client receives bitcoin from a different user [funder], this transaction directly funds a swap address.
The funder does not need to speak the swap protocol, there is no need for pre-signing refund transactions, the funder simply pays a single-pubkey address.
As soon as the funder's transaction confirms, the client can withdraw coins from the swap payment channel.
This protocol deviates from the initial SAS proposal, because at the time the client - server setup the swap, they do not know how much bitcoin the funder will send to the client.
This problem is solved by the use of a payment channel on one side of the swap.

## Protocol

### Participants

- **client**: The user of the swap wallet.
- **server**: The privacy expert hired by the client.
- **funder**: A random non-swap user who pays the client.

### Setup of server

The server has bitcoin that he owns, it is his non-custodial capital.
He does an elaborate privacy ceremony to gain a high anonymity set, mostly by doing CoinJoin. 
This will take the server a relatively long period of time, and should be done well in advance before negotiating with the client.

### Setup of client - server

The client creates a new wallet, and will start negotiation with the server.
The server will open a payment channel with the high anonymity set coins to the user, with all the balance on his side.
This amount must be higher than the user is expecting to be paid by the funder, however, it is not known exactly how much the funder will be sending.
For our example, let us consider that the client expects to receive less than 10 bitcoin, thus this channel will be funded wih 10 btc.
Before the server funds the channel, they negotiate the fallback transactions like with any other payment channel.
[2p-ECDSA](https://eprint.iacr.org/2017/552.pdf) can be used to create this channel with a single public key.

[TODO: detailled tx structure for all cases]

### Setup of external funding

The client asks the server for a `publicSecretServer` [with private key `secretServer`], and he combines this with a `publicSecretClient`, to generate a single public key address.
Only with knowledge of both secrets can a signature be cretaed for this public key.
The client gives the funder this single public key address, and the funder pays whatever amount she wants, for example 3 btc.
Notice that the funder doesn't know that this is a swap, it's just a regular single public key address.

### Negotiation of channel update

The client tells the server that 3 btc was received, and they negotiate an adaptor signature payment channel update.
The clients gets 3 btc in the channel, IF he reveals a `secretClient`.

[TODO: detailed tx structure of channel update]

### Server claims the funds

Now the server has both `secretClient` and `secretServer`, and will spend the 3 btc coin that funder sent on-chain.
This coin is now in the full possession of the server, and he can CoinJoin it so to have the capital available for the next swap channel opening.

### Client spending his coins

At any time, the client can choose to make an on-chain payment, by cooperatively closing the payment channel, and sending his balance into whatever destination addresses he chooses.
For example, if he wants to make a 2 btc payment, the closing transaction will have three outputs, 7 btc to the server [who will CoinJoin], 2 btc to the destination, and 1 btc change to the client [who might potentially swap again].
If the client has a second payment channel to the server, they might negotiate a 1 btc swap into the second channel so to avoid the 1 btc on-chain change to the client.

### Aggregating swaps in a channel

The client can choose to leave the channel open for an unlimited time.
If he receives another payment from a different funder, for example 4 btc, he can repeat the swap ceremony as described above.
This aggregation works for until the payment channel balance has moved all to the client.

## Potential improvements

Currently, the server can link the pre-swap transaction [from the funder] to the post-swap payment channel.
With the usage of [partially blind atomic swaps](https://github.com/ElementsProject/scriptless-scripts/blob/master/md/partially-blind-swap.md) or [anonymous atomic locks](https://eprint.iacr.org/2019/589.pdf) this leak might be closed.
It is an open question if such a setup works with externally funded succinct atomic swaps.
