# A Secure Chat Application using React and the Signal Protocol

## Technology Stack
1. ReactJS library for UI
2. Signal Protocol Implementation for E2EE
3. Axios for AJAX calls
4. LocalStorage to store/fetch Pre-key bundle and Chats
5. Web Sockets Implementation for Instant Messaging

## Components
1. Login
2. Chat Window
    1. Contact List
    2. Message Box

## Axios Calls
1. GET - api/users/login/userName - Returns User Object of the given User
2. GET - api/users/users/userId/role - Returns Users Array other than the given User with given role

## Web Sockets
1. Establishing WS Connection: `let webSocket = new WebSocket("ws://localhost:3000/chat")`
2. Event Listeners of the webSocket Object:
```
    webSocket.onopen = () => {
        console.log(‘WebSocket Client Connected’);
        webSocket.send('Hi this is web client.');
    };
    webSocket.onmessage = (e) => {
        console.log(‘Received: ’ + e.data);
    };
    webSocket.close = () => {
        console.log('WebSocket Client Closed.’);
    };
```

## Signal Protocol Implementation
1. InMemorySignalProtocolStore.js (and helpers.js) are taken for storage purpose from Signal Github (link mentioned in resources)
2. libsignal-protocol.js (also from Signal Github) implements the protocol
3. Signal Gateway - Created by me to integrate React with Signal. It performs the Initialization, Encryption and Decryption functionality when required on Frontend. Check the file in src/signal/SignalGateway.js for detailed code.
4. Calling Signal Methods for Encryption 
```
async getNewMsgObj(newMsgObj) {
        let selectedUserChatId = this.getSelectedUserChatId()
        let msgToSend = { chatId: selectedUserChatId, senderid: this.props.loggedInUserObj._id, receiverid: this.state.messageToUser._id, ...newMsgObj }
        // Send Message for Encryption to Signal Protocol, then send the Encrypted Message to main server
        try {
            let encryptedMessage = await this.props.signalProtocolManagerUser.encryptMessageAsync(this.state.messageToUser._id, newMsgObj.message);
            msgToSend.message = encryptedMessage
            this.state.ws.send(JSON.stringify(msgToSend))
            this.setState({ lastSentMessage: newMsgObj.message }) // Storing last-sent message for Verification with Received Message
        } catch (error) {
            console.log(error);
        }
    }
```
5. Calling Signal Methods for Decryption
```
ws.onmessage = async (e) => {
            let newMessage = JSON.parse(e.data)
            // In case message is from self, save state-stored message to Chats i.e. no need of using/decrypting the received message
            // This is only for verifying that the messages have successfully been received.
            if (newMessage.senderid === this.props.loggedInUserObj._id) {
                newMessage.message = this.state.lastSentMessage
            } else { // Otherwise decrypt it and then save to Chats
                // Decryption using Signal Protocol
                let decrytedMessage = await this.props.signalProtocolManagerUser.decryptMessageAsync(newMessage.senderid, newMessage.message)
                newMessage.message = decrytedMessage
            }
    }
```

## Resources
1. [Signal Protocol in JavaScript Github](https://github.com/signalapp/libsignal-protocol-javascript)
2. [Why Axios](https://medium.com/@MinimalGhost/what-is-axios-js-and-why-should-i-care-7eb72b111dc0)
3. [ReactJS](https://reactjs.org/)
4. [Web Sockets API](https://developer.mozilla.org/en-US/docs/Web/API/Websockets_API)
