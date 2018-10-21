# CoderSchool Chatbots in React Native Workshop!

## Part 1: Setting up React Native

1. Get started with React Native. Set it up locally, or run in Snack.

* To run in Snack: go download the "Expo App" from the app store.
* Go to https://snack.expo.io/. 

2. Import gifted messenger.

If running locally, `npm install react-native-gifted-chat --save`.

```
import {GiftedChat} from "react-native-gifted-chat";
```

Now click on the bottom "add to package JSON."

3. Change the code in the main render function:

```
      <View style={styles.container}>
        <GiftedChat
          messages={this.state.messages}
          onSend={messages => this.onSend(messages)}
          user={{
            _id: 1
          }}
        />{" "}
      </View>
```

4. Add a new constructor, with the following code: 

```
constructor(props) {
    super(props);

    let firstMsg = {
        _id: 1,
        text: "Hello developer",
        createdAt: new Date(),
        user: {
          _id: 2,
          name: "React Native",
          avatar: "https://placeimg.com/140/140/any"
        }
    };

    this.state = {
        messages: [firstMsg]
    };
  
  }

```

5. Hook up an onSend function

```
 onSend(messages = []) {
    console.log(messages);
    // let resp = this.getResponse(messages);

    this.setState(previousState => ({
      messages: GiftedChat.append(previousState.messages, messages)
    }));
  }
```

Now you can chat with yourself. Woo.


## Part 2: The local Chatbot. 

1. Add the following code to your Chat, to the end of your onSend.

```
    let message = messages[0].text;
    let response = this.getBotResponse(message);
    this.sendBotResponse(response);
```

2. Create a silly `getBotResponse` message.
```
  getBotResponse(text) {
    if (text.includes("hello")) {
      return "Hello to you too!";
    }
  }
  
  sendBotResponse(text) {
      let msg  = {
      _id: this.state.messages.length + 1,
      text,
      createdAt: new Date(),
      user: {
        _id: 2,
        name: "React Native",
        avatar: "https://placeimg.com/140/140/any"
      }
    };
    this.setState(previousState => ({
      messages: GiftedChat.append(previousState.messages, [msg])
    }));
  }
```

Hooray. Now you have a silly chatbot. You can extend the logic of the `getBotResponse` function to be smarter and understand simple if statements.


## Part 3: Set up DialogFlow.
![](https://i.imgur.com/I1lmmdj.png)

1. Create first Agent

```
name: restaurant_bot

Default language: English - en
```
![](https://i.imgur.com/JFsE6CV.png)

2. Create an Intent

But wait, what is the Intent??

Very shortly, an Intents is a collection of sentences which have the same meaning and the bot knows how to answer to.

In other words, the Intent is how we map sentences to what we expect the bot to do (ex: we tell the bot to “make a booking” and we expect a booking to be made and to receive a confirmation). The collection of sentences, also called Training Phrases, are how we ask the bot to trigger the Intent.

So let's create our first Intents.

> From the left menu >  Intents --> select **CREATE INTENT**
![](https://i.imgur.com/dUYtNeK.png)
![](https://i.imgur.com/rUBnRS3.png)

Intent name: 
```
restaurant.booking.create
```

>Training phrases: --> **Add training phrases**

Training phrases:
```
Book a table
Make reservations
Reserve a table
Book a table for 3
Make reservations for 5
Reserve a table for 2
```

When the agent receives a text phrase that matches the above, the agent will look for **Responses** from this **Intent**.

At this step, we get a new term: an **ENTITY**.

An **ENTITY** is a keyword extracted from Training Phrases. The value can then be further used as a parameter. It is not necessary to create entities for every word, just the ones that are important for our final result (our fulfillment).

So it's a parameter for **Fulfillment**, let's rename things for readability. `Guests` makes more sense than just `number`.

> Action and parameters: --> **Add Action and parameters**

double check you have something looks similar to this
![image alt](https://i.imgur.com/IPyL4Ub.png)

Guests is a required param so tick the **REQUIRED** box. Let's make a prompt to handle the case for case where users do not specify the option.

> Define prompts...
![](https://i.imgur.com/jdfmcTc.png)

Prompts for "guests"

**PROMPTS**
```
For how many guests?
How many people are coming?
what is the number of guests?
```

DialogFlow will chose one randomly and send it to the user. So, add as many as you want. The more you have, the more conversational the bot seems.

**Responses**
> ADD RESPONSES

Text response
```
You have successfully booked a table for $guests guests.
Your booking was made for $guests guests.
Booking confirmed for $guests guests.
```

Now, you done with Intent but don't forget to save it. Changes are only effective once they've been saved.

Everything's ok now. Let's try your bot.
Prompt for you to input is on top left corner of the site. **Try it now** 
```
book a table
reserve a table for 2
I want to book a table for a group of ten
etc.
```

3. Validation
What if users want to book a table for 0 guests? That would be bad. We have to validate users' requests before further processing.


> Fulfillment: --> **ENABLE FULFILLMENT**
(Fulfillment section on Intent tab, not the one on the left menu bar)

Switch on "Enable webhook call for this intent" and **SAVE**

Well done with Intent, Let's go to **Fulfillment** on the left menu.

Enable **Inline Editor**
![](https://i.imgur.com/N3OJ6Il.png)

```javascript
  function welcome(agent) {
    agent.add(`Welcome to my agent!`);
  }
 
  function fallback(agent) {
    agent.add(`I didn't understand`);
    agent.add(`I'm sorry, can you try again?`);
  }
  // Your code here
  function createBooking(agent) {
    let guests = agent.parameters.guests;
    
    if (guests < 1) {
        agent.add('You need to reserve a table for at least one person. Please try again!');
    } else {
        agent.add(`You have successfully booked a table for ${guests} guests`);
        agent.add('See you at the restaurant!');
        agent.add('Have a wonderful day!');
    }
  }
  // End
  ...
  
  
  ...
  let intentMap = new Map();
  intentMap.set('Default Welcome Intent', welcome);
  intentMap.set('Default Fallback Intent', fallback);
  // Your code here
  intentMap.set('restaurant.booking.create', createBooking);
  // End
  agent.handleRequest(intentMap);
```
Make sure press on **DEPLOY** after changing. And let's rock, let's rock

## Part 4: Hook it up to React Native.

1. `npm install react-native-dialogflow-text --save`

CoderSchool made a special npm package just for this workshop. ;)

2. Set up your bot in Dialogflow for V2 Auth.

Don't worry if you need to ask for help here, it's hard to find the right menu options.

* Go to settings. Change to V2.
* Hit the save button.
* Click Add service account
* Click the service account

![](https://i.imgur.com/pmwlxZT.png)

Once in the service account, find the one that says Dialog flow and click create key. Download the JSON, and note your:

  * private key
  * client email
  * project id

3. Initialize your bot. 

In your constructor:

```
   Dialogflow_V2
      .setConfiguration(
        "dialogflow-ygqyvo@coffee-shop-1eaf1.iam.gserviceaccount.com",
        "-----BEGIN PRIVATE KEY-----\nMIIEvQIBAD...=\n-----END PRIVATE KEY-----\n",
        Dialogflow_V2.LANG_ENGLISH_US,
        "coffee-shop-1eaf1"
      )
```

4. Hook it up and watch the sparks fly.

Replace the call to `getBotResponse` in your `onSend` to the following:

```
Dialogflow_V2.requestQuery(
      message,
      result => this.handleGoogleResponse(result),
      error => console.log(error)
    );
```

Add a handleGoogleResponse function like so:

```

  handleGoogleResponse(result) {
    let text = result.queryResult.fulfillmentMessages[0].simpleResponses.simpleResponses[0].textToSpeech;
    this.sendBotResponse(text);
  }
```

![](https://i.imgur.com/ncF8teb.png)
