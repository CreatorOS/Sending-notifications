# Sending notifications
Whenever an event happens on your contract, there is no native way to send a push notification to the users of your contract. In this quest we will look at how we can still send push notifications using third party services. We will be using ECM - Ethereum Cloud Messaging. 

This quest requires that you have done a couple of quests in the Ethereum series. You should know how to deploy contracts to the Ropsten test network. You should also know how to import contracts. In quest 2 we imported Compound, in this quest we’ll be importing ECM.
## Why are notifications not inbuilt?
Ethereum is by default decentralized. Unlike apple and google, which own the mobile phones and hence the push notifications APNS and FCM respectively. There is no centralized entity that is responsible for maintaining user states. That also gives rise to the fact that there is no entity that has access to sending notifications to users either.

However, there is a semi-decentralized way to send notifications. Every browser is capable of receiving notifications from the provider associated with that browser. Google Chrome uses Google’s notification, Firefox uses Mozilla’s notifications and so on. They all adhere to a standard called *webpush*. 

ECM leverages this notification system that already exists to send notifications to the users’ browser.
## What is ECM?
ECM stands for Ethereum Cloud messaging. It provides functions to send notifications from smart contracts. The push notifications are sent by a relayer node that tells Google to send a push notification to a certain Chrome user or Mozilla to send a notification to a certain Firefox user, etc. This is called the ECM node. People around the world are running these nodes making the notifications more decentralized.

To send a notification on ECM, the sender must pay. They must pay a transaction fee and a fee that is set by the user. Every relayer node has a fixed price to send notifications through it. A user subscribes to a certain relayer node. By subscribing to a relayer node, they set the price of sending a notification to them. This avoids people from being able to send spammy notifications. The price is distributed between the user and the relayer node.

First, we’ll look at how we can setup a notification system for our contract and then look at how we can setup a node ourselves. 

We’ll send a notification whenever a new allowance is given to our ERC 20 token.
## Importing ECM
The ECM contract has a function called `sendNotification` that is what we’ll use

First, define the interface in your contract

```

interface ECM {

  sendNotification(address user, string memory title, string memory text, string memory image) external payable;

}

```

Then we’ll initialize the ECM contract 

```

ECM ecm = ECM(0x2F04837B299d8eD4cd8d6bBa69F48EdFEc401daD);

```

ECM is deployed on the above address on Ropsten. We have now initialized the object with the appropriate address and schema so that we can start calling contract functions. 

Let us now modify the allowance function that we had written in quest 4 (). Whenever a user approves another user to use the funds, we’ll send a notification to the user who has received the approval. 

At the end of the approve function add 

```

ecm.sendNotification(spender, “Notification from M20 coin”, “You have an awaiting approval”, “none”);

```

This will work whenever the user is ready to accept notifications without a charge. However, if the user has set a price to send a notification to them - this will not work, because we’re not sending any money to the ECM contract.
## Attaching the fees for sending a notification
This is a classic example of computers paying computers that was previously not possible with fiat currencies. 

Our contract will automatically pay the ECM contract to send a notification. 

First we need to know how much money we need to send to ECM so that it can send a notification to the user. To do that we need to add the function to the interface :

```

interface ECM {

  sendNotification(address user, string memory title, string memory text, string memory image) external payable;

  getUserPrice(address user) external;

}

```

The person approving the spender must also attach the ETH required to send the notification.

```

if(msg.value >= ecm.getUserPrice(spender)) {

  ecm.sendNotification{value: msg.value} (...);

}

```

This ensures that the required amount is sent to the function sendNotification by attaching all the amount received to send a notification. 

When a user calls the approve function, they must also attach money else the notification will not be sent. That means we need to convert our approve function to be payable so as to be able to accept money that needs to be forwarded to ECM. 

```

    function approve(address _spender, uint256 _value) public payable returns (bool success) {

        allowances\[msg.sender\]\[_spender\] = _value;

        if(msg.value >= ecm.getUserPrice(spender)) {

            ecm.sendNotification{value: msg.value} (...);

        }

    }

```
## Asking users to accept notifications
Now that you have setup your contract to send notifications, the users have to explicitly tell ECM that they wish to receive notifications. 

To do so the users can head over to https://madhavanmalolan.github.io/ecm.js/ and tap on subscribe to notifications. 

Once they do, they’ll be asked to send a signed transaction using Metamask.

Once they have subscribed to notifications from ECM, they’ll be able to accept notifications from any contract, without additional setup.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/8c920b78-2b93-45ac-90c6-6538cfa94a0d.jpg)

Once connected the badge at the bottom will look like this :

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/82c44a39-48fd-44db-9072-66e5a5b4df7d.jpg)
## What next?
Follow the documentation on ecmnetwork.org to run your own relayer node so that you can also start earning some money passively :)

Feel free to explore the JS code of the relayer node & the ecm contract. It is straightforward and you should be able to understand what’s happening there :)
