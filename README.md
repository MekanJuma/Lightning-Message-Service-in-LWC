# Lightning Message Service (LMS) in Lightning Web Components (LWC)

![LMS](https://github.com/MekanJuma/Lightning-Message-Service-in-LWC/blob/main/LMS.PNG)

## Table of Contents
- [Overview](#overview)
- [Creating a Custom Message Channel](#creating-a-custom-message-channel)
- [Importing in LWC](#importing-in-lwc)
- [Explanation of Imports](#explanation-of-imports)
- [Example Components](#example-components)
- [Manage LMS with SFDX Commands](#manage-lms-with-sfdx-commands)


## Overview

The Lightning Message Service (LMS) facilitates communication between Lightning Web Components (LWC), Aura components, and Visualforce pages. It allows for a decoupled communication model, where components can publish and subscribe to messages without being directly aware of each other, enhancing the reusability and maintainability of components.

## Design

LMS uses a publish-subscribe model where components can send (publish) or receive (subscribe) messages on predefined channels. These channels can be either standard or custom-defined, providing flexibility in how components interact across a Salesforce application.

## Creating a Custom Message Channel

- Go to your project's default folder **'force-app/main/default/'**
- Check if you have a folder named **'messageChannels'**, if not then create one folder under the default folder with the name **'messageChannels'**. Please use the provided name only, as this is the standard folder name.
- Now create a new file inside **'messageChannels'** folder with the following name format and give your desired name to the message channel.
  ```
  yourMessageChannelName.messageChannel-meta.xml
  ```
- Now open the file you just created in the previous step and copy-paste the below code in that.
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <LightningMessageChannel xmlns="http://soap.sforce.com/2006/04/metadata">
    <masterLabel>YourMessageChannelName</masterLabel>
    <isExposed>true</isExposed>
    <description>This is a sample Lightning Message Channel.</description>
	
    <lightningMessageFields>
        <fieldName>recordId</fieldName>
        <description>This is data parameter that you want to send through LMS</description>
    </lightningMessageFields>

    <lightningMessageFields>
        <fieldName>recordData</fieldName>
        <description>This is another data parameter</description>
    </lightningMessageFields>

  </LightningMessageChannel>
  ```
- the **lightningMessageFields** defines the parameters that you can send using this message channel. It has two properties, fieldName, and description. You can add as many parameters as you want, but it's good to keep it optimum for better performance.
- Now, deploy it before using this channel anywhere

## Importing in LWC
To use LMS, import the necessary modules from lightning/messageService:

- **APPLICATION_SCOPE**: Constant defining the scope of the subscription.
- **MessageContext**: A module to wire the message context in the component.
- **publish**: Function to send messages to a channel.
- **subscribe**: Function to subscribe to a channel.
- **unsubscribe**: Function to unsubscribe from a channel.

  Example:
  ```js
    import {
      APPLICATION_SCOPE,
      MessageContext,
      publish,
      subscribe,
      unsubscribe,
    } from 'lightning/messageService';
    import YourChannelName from '@salesforce/messageChannel/YourChannelName__c';
  ```

## Explanation of Imports
- **APPLICATION_SCOPE**: Allows the component to receive messages published by any component in the application, irrespective of where they are.
- **MessageContext**: Provides the necessary context for publishing and subscribing to messages.
- **publish**: Enables a component to publish a message to a specified message channel.
- **subscribe**: Used to subscribe to a message channel and define a callback function for handling incoming messages.
- **unsubscribe**: Cleans up and unsubscribes from a message channel, typically used when the component is destroyed.

## Example Components
**Publisher Component**
- The publisher component sends messages to a specified channel when an action occurs (e.g., input value changes).

**PublisherComponent.js**
```js
import { LightningElement, wire } from 'lwc';
import { publish, MessageContext } from 'lightning/messageService';
import INPUT_VALUE_CHANGE_CHANNEL from '@salesforce/messageChannel/InputValueChange__c';

export default class PublisherComponent extends LightningElement {
    inputValue = '';

    @wire(MessageContext)
    messageContext;

    handleInputChange(event) {
        this.inputValue = event.target.value;
        const payload = { inputValue: this.inputValue };
        publish(this.messageContext, INPUT_VALUE_CHANGE_CHANNEL, payload);
    }
}
```

**Subscriber Component**
- The subscriber component listens for messages on the specified channel and reacts accordingly.

**SubscriberComponent.js**
```js
import { LightningElement, wire } from 'lwc';
import { 
    subscribe,
    unsubscribe,
    APPLICATION_SCOPE,
    MessageContext 
} from 'lightning/messageService';
import INPUT_VALUE_CHANGE_CHANNEL from '@salesforce/messageChannel/InputValueChange__c';

export default class SubscriberComponent extends LightningElement {
    subscription = null;
    receivedValue = '';

    @wire(MessageContext)
    messageContext;

    connectedCallback() {
        this.subscribeToMessageChannel();
    }

    disconnectedCallback() {
        this.unsubscribeToMessageChannel();
    }

    subscribeToMessageChannel() {
        if (this.subscription) {
            return;
        }
        this.subscription = subscribe(
            this.messageContext, 
            INPUT_VALUE_CHANGE_CHANNEL, 
            (message) => this.handleMessage(message),
            { scope: APPLICATION_SCOPE }
        );
    }

    unsubscribeToMessageChannel() {
        unsubscribe(this.subscription);
        this.subscription = null;
    }

    handleMessage(message) {
        this.receivedValue = message.inputValue;
    }
}
```

## Manage LMS with SFDX Commands

**Deploy**:
```
sfdx force:source:deploy -m LightningMessageChannel
```

**Retrieve**:
```
sfdx force:source:retrieve -m LightningMessageChannel:YourChannelName
```

**Delete**:
```
sfdx force:source:delete -p ./force-app/main/default/messageChannels/YourChannelName.messageChannel-meta.xml
```
