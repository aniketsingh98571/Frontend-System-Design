# Trello Frontend System Design

## Requirements Gathering
### Functional Requirements:-

1. User should be able to create list.
2. User should be able to create cards in a list.
3. Card should allow to import image and type the task.
4. User should be able to assign task to team in a card.
5. User should be able to archive the card.
6. User should be able to drag/drop the card.

### Non-Functional Requirements:-
1. Handle Concurrency when two people drag and drop at same time the same card.
2. Handle updates when two drops happens one after another of same card.

## Architecture
![Trello HLD](../../assets/Trello_HLD.png)

## Rendering Approach
SSR + hydration (Next.js)

## Data Model
```js
Lists - Server - [{
   list_id:"1",
   list_name:"Urgent"
}] 
Cards -  Server - [{
    list_id:"1",
    id:"2",
    title:"Create card"
}]
NewList -  Client - {}
NewCard - Client - {}
```

## Handling Concurrency (NFR)
1.  When two people drag and drop at same time the same card?
Ans- The person who drops at last wins, because
        Trello uses a last-write-wins conflict resolution system.

2.  When two drops happens one after another of same card?
Ans - It works also on the same principle of last-write-wins conflict resolution system.

## Realtime updates
We can make use of Websockets to listen for any upcoming message from server and update the UI accordingly.

Example Code:-
```js
// Create a WebSocket connection to the server
let socket = new WebSocket("ws://your-websocket-server.com");

// Connection established
socket.onopen = function(event) {
  console.log("Connection established ", event);
};

// Listen for messages from the server
socket.onmessage = function(event) {
  console.log("New message from server ", event.data);

  // Assuming the server sends a JSON string that includes an action type
  let serverMessage = JSON.parse(event.data);

  if (serverMessage.action === 'cardMoved') {
    // Update the UI to reflect the card move
    moveCard(serverMessage.cardId, serverMessage.newPosition);
  }
};

// Function to send a message to the server when a card is moved
function cardMoved(cardId, newPosition) {
  let message = {
    action: 'cardMoved',
    cardId: cardId,
    newPosition: newPosition
  };

  // Send the message as a JSON string
  socket.send(JSON.stringify(message));
}

// Function to update the UI when a card is moved
function moveCard(cardId, newPosition) {
}

```