# Build a customer feedback survey conversational AI app

## Intro

Email and sms customer feedback surveys suck. Phone surveys are cool because they are so rare… and expensive. At least they were expensive. Now you can build a Dasha conversational AI app that will reach out to your customer after an interaction, ask some questions, collect the ratings *and* collect some open, actionable feedback. It just works. Also, it’s a fun little project. 

You can watch the video below to see a live demo of what you will have built 
<a href="https://youtu.be/mJ_OZYTuJys" class="embedly-card" data-card-width="100%" data-card-controls="0">here</a>.

In this tutorial we will go over: 

- Designing the conversation map.
- Outsourcing your phrases to __phrasemap.json__ to clean up the body of your main.dsl application.
- Calling on external functions in your __index.js__ and passing variables from your DashaScript code.
- Running calculations within the body of your DashaScript application.
Using __`#getMessageText();`__ to save entire user phrases as strings. 

We will also touch on some things we’ve covered previously, such as:
- Creating custom intents and named entities, setting up data to train the neural network.
- Creating digressions and ensuring their native flow within the conversational app. 
- Structuring “perfect world” node flow. 

If this is your first time building conversational apps with Dasha, I recommend you join our [developer community](https://community.dasha.ai) and read [this post](https://dasha.ai/en-us/blog/conversational-ai-app-creation-guide). 

## How to start the demo app

2. Create or log into your account using the Dasha CLI tool. If you have any difficulty, go to https://community.dasha.ai 

```sh
npx dasha account login
```

3. To start a text chat, run:

```sh
npm start chat
```

4. To receive a phone call from Dasha, run:

```sh
npm start <your phone number>
```

The phone number should be in the international format without the `+` (e.g. `12223334455`)

## Preparing the script for your customer feedback survey conversational AI app 

A conversational AI application is literally a way of applying conversational AI technology to solve a specific real world problem, for example - how to get feedback from customers. The conversational app interacts with the user (customer) through speech - understanding, interpreting and generating natural language. For more on how Dasha Cloud Platform uses its conversational AI as a Service to make your apps human-like, you can read [here](https://dasha.ai/en-us/blog/dasha-conversational-ai-as-a-service). 

In order to create an app, you need to have a basic understanding of the type of interactions you expect the AI to have with the user. This includes user replies, requests, the AI’s phrases and the direction in which you want it to take the conversation. In a way, this first step is similar to how you may document the UX of a mobile or web app. 

For the sake of the present conversation, let’s envision a conversation in which Dasha calls ACME Bank’s customer a few hours after they had visited the bank’s office. She then proceeds to ask if they have got two minutes to fill out a survey. If they do, she asks three customer feedback questions with a rating of 1-5. If the rating is identified as negative, we will have Dasha ask the customer to specify what could have been done better. For the last question “what was your overall experience like,” we will ask to elaborate on the details of the experience even if it was positive, as well as negative.

I like to throw together a simple conversation map to outline my conversation. For detailed instructions on how to create your conversational map, you can refer to [this post](https://dasha.ai/en-us/blog/build-conversational-AI-app-1).

For the project we are building, this is the conversational AI app map I ended up with: 

You can find the spreadsheet [here](https://docs.google.com/spreadsheets/d/1BxyTsJ3sddEnM4Z6ZbMtSUjIojNEGtbEUE-tythKaVg/edit#gid=0). Feel free to copy to your Google Drive and edit it as you may see fit to change your conversational app.

Here is what we’ll cover next: 
- Creating the “perfect world” conversation flow
- Adding digressions

In the course of this we will cover everything promised above - using the phrasemap, creating neural training data, running calculations using DashaScript. 

## Building the “perfect world flow” version of your customer feedback survey conversational AI app 

First, make sure you’ve got the latest version of Node.js and Visual Studio Code. Now, head over to our [developer community](https://community.dasha.ai) where you will get instructions to your Dasha API key. You will also want to install the Dasha Studio extension in VS Code, a well as the Dasha [command line interface](https://dasha.ai/en-us/blog/dasha-cli) __`npm i -g "@dasha.ai/cli" `__. AIf you need a quick start guide, please refer to [this post](https://dasha.ai/en-us/blog/virtual-receptionist-software). If you have any difficulties, just ask in our developer community. 

Now open a Dasha app in VS Code. I propose you start with the [first app](https://github.com/dasha-samples/dasha-first-app). 

Now, open up the following files:  
- __main.dsl__  - you use your main DashaScript file to define the conversational workflow.
- __phrasemap.json__ - you use the phrase map to store phrases for Dasha to pronounce in the course of the conversation. You map to the phrases from __main.dsl__.  
-  __intents.json__ -  here is where you store the data with which to train the neural network to recognize custom intents and named entities. Pro tip: rename to data.json, because the file includes named entities, not only intents. 
-  __index.js__ - the NodeJS file which launches the Dasha SDK. This is where you can use external JS functions to augment your conversational workflow or build out integrations to external services.

Go to __main.dsl__. 

Let’s start by importing common libraries

```dsl
import "commonReactions/all.dsl";
```

Now, let’s declare some variables. We are going to use these to store variables. Here and onwards, refer to the in-code comments for additional specifications:

```dsl
context
{
   // declare input variables phone and name - these variables are passed at the outset of the conversation. In this case, the phone number and customer’s name
   input phone: string;
   input name: string = "";

   // declare storage variables
   q1_rate: string = "";
   q2_rate: string = "";
   q3_rate: string = "";
   q1_feedback: string = "";
   q2_feedback: string = "";
   q3_feedback: string = "";
   final_feedback: string = "";
   call_back: string = "";
}
```

Next, let’s declare an external function. External function is how you call out to __index.js__ from DashaScript (__main.dsl__) to make use of JavaScript functions. 

```dsl
// declaring external function for console logging, so that we can check the values of the variables, as the conversation progresses 
external function console_log(log: string): string;
```

We’ll look at this external function a bit later. Now, let’s move to the actual conversation flow. The first node in the conversation is called the __`node root`__. As above, please refer to the comments below. They will help to paint the full picture.

```dsl
start node root
{
   do //actions executed in this node
   {
       #connectSafe($phone); // connecting to the phone number which is specified in index.js that it can also be in-terminal text chat
       #waitForSpeech(1000); // give the person a second to start speaking
       #say("greeting", {name: $name} ); // and greet them. Refer to phrasemap.json > "greeting" (line 12); note the variable $name for phrasemap use
       wait *;
   }
   transitions // specifies to which nodes the conversation goes from here and based on which conditions. E.g. if intent “yes” is identified, the conversation transitions to node question_1
   {
       question_1: goto question_1 on #messageHasIntent("yes"); // feel free to modify your own intents for "yes" and "no" in data.json
       all_back: goto when_call_back on #messageHasIntent("no");
   }
}
```

Note that in the function __`#say("greeting", {name: $name} );`__ we refer to __`greeting`__. The __`#say()`__ function maps to the reference phrase in __phrasemap.json__.   This means we need to add the values to your phrasemap. Open it up. You will see up top the following 9 lines of code. Keep it. This code controls the speech synthesis. Feel free to play around with it but these are the preferred values. 

```json
{
 "default": 
   {
   "voiceInfo": 
     {
     "lang": "en-US",
     "speaker": "V2",
     "speed": 0.3,
     "variation": 4
     },

```

We will add some phrases to this file as we go along. If there are leftover phrases which are unused by our present app, it will not hurt the performance of the app. However, I encourage you to look through your JSON code and clean out all unused pieces of code. 

Let’s add the phrase “greeting”, so that it can map to the relevant code in __main.dsl__. 

```json
     "greeting": [
       { "text": "Hi " },
       { "id": "name", "type": "dynamic" },
       { "text": " this is Dasha with Acme Credit Union. You visited one of our offices earlier today. I'd like to ask you a few questions about the experience. Do you have two minutes now? " }
     ],
```
Now scroll down, until you see __```macros```:__ and add this line:
```json
      "greeting": {},
```

Remember that for every phrase you add to the phrase map, you need to have a corresponding macro. If you forget, your IDE will let you know you made the mistake. Now your app knows how to greet the user. Note that we are substituting a dynamic variable “name” to refer to the user by their name. 
The input variable __`name`__ is also used in the function we were just looking at __`#say("greeting", {name: $name} );`__. As you run your conversational app, you would input the value for “name” following for the phone number. The terminal command to launch a call would look something like this: __`npm start 12223334455 John`__. Now, in order for the application to recognize “John” as mapping to variable __`name`__, we need to provide instructions in the SDK. Open __index.js__ and __`const conv = app.createConversation`__ modify this line to read. 

```javascript
 // in the line below, to account for name input context variable, you declare below: name: process.argv[3] ?? ""
 const conv = app.createConversation({ phone: process.argv[2] ?? "", name: process.argv[3] ?? "" });
```

This code is found in lines 57-58 of __index.js__, as found in the GitHub repository.
Great work. Now let’s assume that our user replied in the positive to Dasha’s request for two minutes and move on to the perfect world flow below. We finally get to ask the first question of our automated customer feedback survey. 

```dsl
node question_1
{
   do
   {
       #say("question_1"); //call on phrase "question_1" from the phrasemap
       wait *;
   }
   transitions
   {
       q1Evaluate: goto q1Evaluate on #messageHasData("rating");// when Dasha identifies that the user's phrase contains "rating" data, as specified in the named entities section of data.json, a transfer to node q1Evaluate happens
   }
}
```

Pretty straightforward stuff. Dasha pronounces the phrase for __`question_1`__ from the __phrasemap__, waits for a response and, on recognizing rating data, transfers to __`node q1Evaluate`__. You will need to add  __`question_1`__ to the phrasemap file. I will show you this one last example, the rest of the phrasemap modifications you will do on your own, using previous ones as examples. 

```json
     "question_1":
       {
         "first":
         [{ "text": "Perfect, thank you. First question - how would you rate the bank employees with whom you interacted on the scale of 1 to 5." }],
         "repeat":
             [{ "text": "I was saying. how would you rate the bank employees with whom you interacted on the scale of 1 to 5." }]
       },
```

Note the __”repeat”__ value. This lets us provide an alternative phrase for the AI to substitute for the original in case this node gets called on a second time. Such a thing would usually happen when coming back from a __digression__. To learn more about digressions, you can take a look at [this article](https://dasha.ai/en-us/blog/using-digressions). 

The second part I want to draw your attention to in the node above is the transition to __`node q1Evaluate`__. The function __`#messageHasData()`__ tells Dasha to check for a specific set of data, as defined in the __“entities”__ section of __data.json__.  Go to the file. You will need to append the code below after the closing curly bracket for __”intents”__. 

```json
 "entities":
 {
   "rating":
   {
     "open_set": false, 
     "values": [
       {
         "value": "1",
         "synonyms": ["1", "one", "zero", "horrible", "worst ever", "the worst", "awful", "horrid", "despicable", "detestable", "very bad"]
       },
       {
         "value": "2",
         "synonyms": ["2", "two", "bad", "quite bad", "pretty bad", "not good", "crappy"]
       },
       {
         "value": "3",
         "synonyms": ["3", "three", "alright", "okay", "just okay"]
       },
       {
         "value": "4",
         "synonyms": ["4", "four", "good", "pretty good", "quite good", "good enough"]
       },
       {
         "value": "5",
         "synonyms": ["5", "five", "amazing", "incrdible", "just grand", "perfct", "wondrful", "very good", “ten”, “10”, “6”, “6”]
       }
     ],
     "includes": [
       "I would say it was (1)[rating]",
       "(4)[rating]",
       "I had an (3)[rating] experience",
       "It was (4)[rating]”,
	“Totally (2)[rating]”
     ]
   }
 }
}
```

Note the __`"open_set": false, `__. This tells the AI that it cannot substitute just any values for the ones defined in the file. The match has to be exact. Now, this only applies to the __”value”__, not to the __“synonym”__. For example, with time the neural network will recognize “brilliant”, as signifying “5”, even though it is not mentioned in the training data. But it will never recognize “6” as a reasonable value to save in place of “1” or “5”. If you were to set the parameter to “true”, it would. 

Also, pay attention to the __”includes”__ section. It provides a few variations of the types of constructions the AI may expect to hear from the user, so that it knows in which place to look for the value, in case it is not an exact match to one of the “synonyms”. 
We got through __`node question_1`__. Let’s assume the user gave us an acceptable value which was rightly interpreted by the neural network and we are on to the next node. In this one, we evaluate the received value to estimate if the response is positive or negative. 

```dsl
node q1Evaluate
{
   do
   {
       set $q1_rate =  #messageGetData("rating")[0]?.value??""; //assign variable $q1_rate with the value extracted from the user's previous statement
       var q1_num = #parseInt($q1_rate); // #messageGetData collects data as an array of strings; we convert the string into a number in order to evaluate whether the rating is positive or negative
       if ( q1_num >=4 && q1_num <=5 )
       {
           goto question_2; // note that this function refers to the transition's name, not the node name
       }
       else
       {
           goto question_1_n;
       }
   }
   transitions
   {
       question_2: goto question_2; // you need to declare transition name and the node it refers to here
       question_1_n: goto question_1_n;
   }
}
```

Named entity variables are stored as an array of strings. In order for us to interpret the extracted value, we need to convert it to an integer. Once it is converted to integer, we can compare the value. If it is greater than or equal to 4, we go on to __`node question_2`__. If it is less than 4, we want Dasha to ask the user how their experience could have been made better. Let’s do just that now. 

```dsl
node question_1_n
{
   do
   {
       #say("question_1_n");
       wait*;
   }
   transitions // specifies an action that Dasha AI should take, as it exits the node. The action must be mapped to a transition
   {
       q1_n_to_q2: goto q1_n_to_q2 on true; // "on true" is a condition which lets Dasha know to take the action if the user utters any phrase
   }
   onexit 
   {
       q1_n_to_q2: do
       {
           set $q1_feedback = #getMessageText();
           external console_log($q1_feedback); // call on external function console_log (we want to see that the data was collected properly), you can then use the variable to push to wherever you want to use it from index.js
       }
   }
}
```

Take a look at the __`onexit`__ section. This is where we use our external function which we initialized in the beginning of __main.dsl__. We want to be able to check that the values were collected properly in this part of the conversation. To do so, we need to store the value collected from the user’s reply in the previous node as variable (__`$q1_feedback`__) and send the value to our JS file and execute the __`console.log()`__ function.

Of course, in order to use the function, we need it to exist in our __index.js file__, so let’s head over there and add the code below within the __`async function main()`__ function. 

```javascript
// in the next 4 lines you set up a function for checking your acquired variables with external function console_log
app.setExternal("console_log", (args, conv) =>
 {
   console.log(args);
 });
```

This is line 50 if you are looking at the file found in the GitHub repo. 
Now that we collected the open feedback we can head over to the next question. However, logic and good upbringing call for us to say something encouraging to the customer who poured their heart out to us. Unfortunately, there is no way to say a phrase after the __`onexit`__ section, so we head to a transition node. 

```dsl 
node q1_n_to_q2
{
   do
   {
       #say("transition");
       goto question_2;
   }
   transitions
   {
       question_2: goto question_2;
   }
}
```

Quite self-explanatory. From here we head over to __`node question_2`__. I will leave it up to you to recreate questions 2 and 3, along with all the phrase maps based on the example nodes above. Do bear in mind that question 3 has branches - positive and negative, not negative and next question. When in doubt refer to the conversation map. There is also the final question that we ask before disconnecting - does the customer have anything else to add. That leads into the final node: 

```dsl 
node final_bye
{
   do
   {
       #say("final_bye");
       exit;
   }
}
```

You should also add two nodes for the call back flow.

## Digressions - what to do when your user strays from the customer feedback survey script 

A digression is activated when Dasha identifies that the user has mentioned a specific intent. A digression can be activated in any point in the conversation. You can read up more on digressions [here](https://dasha.ai/en-us/blog/using-digressions).

As you can see in our [conversation map](https://docs.google.com/spreadsheets/d/1BxyTsJ3sddEnM4Z6ZbMtSUjIojNEGtbEUE-tythKaVg/edit#gid=0), we have defined quite a few digressions. Let’s create the digression for “how_are_you”. First, you will want to define the intent, so that Dasha knows when the digression is called. Add this code to the __”intents”__ part of the __data.json__ file. 

```json
    "how_are_you": 
    {
      "includes": [
        "how are you?",
        "how is everything?", 
        "you okay?",
        "how are you",
        "what it do"
      ]
    }
```

Next, head over to __main.dsl__. Scroll down to the bottom and add this code for the digression. 

```dsl
digression how_are_you
{
   conditions {on #messageHasIntent("how_are_you");}
   do
   {
       #sayText("I'm well, thank you!", repeatMode: "ignore");
       #repeat(); // let the app know to repeat the phrase in the node from which the digression was called, when go back to the node
       return; // go back to the node from which we got distracted into the digression
   }
}
```

Again, quite self explanatory thanks to the comments. When intent __”how_are_you”__ is recognized, the platform says “I’m well, thank you!” (note that I’m using __`#sayText`__, not __`#say`__ here. This means I can type up the text right in the node and don’t have to refer to the phrase map). Then it goes back to the node from which it was so rudely interrupted and repeats the phrase that Dasha pronounced after which the user initiated the digression. If you provide alternative phrasing in your phrase map, Dasha will use it. 

And there you go. If you follow these instructions you will have built a basic customer feedback survey conversational AI app. Just in case you haven’t visited yet, here is the link to the source code in the [GitHub repository](https://github.com/arrrgr/customer-feedback-survey) again. 

If this tutorial was helpful, do let me know in [Dasha developer community](https://community.dasha.ai) or at arthur@dasha.ai. If it was hard to understand, please do the same. Good luck and godspeed!
