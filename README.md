# Hydra Poll dApp

Hydra-Poll was a quick attempt to have an independent frontend working with the
opened Hydra Head.

What we wanted is to have users build and sign Hydra transactions on the frontend.
There is a live Hydra Head between two participants and the frontend connects to it
using websocket protocol.

So how can any user sign and submit a Hydra transaction if they are not part of
the Head I hear you ask?

What we did is commit a script output into a Head. The script is very simple
and its validator always returns True so anybody can spend this utxo over and
over again. You don't actually need to sign the transaction but we wanted to
add some interactivity to the app. Don't worry the fees are zero so you don't
need to pay anything thanks to the altered protocol parameters in the Head.

One problem we had and this is where the flexibility of Hydra really shines is
that normally ledger rules will prevent you from providing script output as
collateral input. Since Hydra uses it's own local ledger which you can tweak to
your needs we were able to alter it and remove this rule so that we can
actually use the same script utxo we are spending in the Head also as the
collateral input!

We couldn't find an easy way to trick MeshJS into not requiring collateral
input since normally when you want to spend from a script collateral input must
be provided (we are working with their team to have a bit more flexible rules
when it comes to building transactions).

In the end the app is really simple but might be of use to someone who wants to
start to build on Hydra Head protocol.

By the way we opened the necessary ports on our server so anybody can connect
to this Hydra Head instance from anywhere and run this frontend or even better
go wild and build something a bit different.

What we currently do is put number 1,2 or 3 in the metadata. These numbers
represent three voting options. When clicking the vote button we send out a
new Hydra transaction and then observe `TxValid` Hydra message over websocket.
When we detect it, it means the transaction is valid and applied to Hydra's
local ledger and then the counter is incremented. You could alter the metadata
to include something you would like and build a completely different frontend!

### How to run the frontend

This repo contains the frontend part of the `Hydra Poll`. It is built using MeshJS
and the first thing to do to run the app is to install dependencies.
This can be done using `npm` or `yarn`:

```
$ npm i
```

or:

```
$ yarn
```

After the dependencies are installed we need to build the project:

```
$ npm run build

```

If the frontend is successfully built you can invoke:

```
$ npm run start

```

for the development version you can run:

```
$ npm run dev
```

and the frontend will start on default port 3000.

