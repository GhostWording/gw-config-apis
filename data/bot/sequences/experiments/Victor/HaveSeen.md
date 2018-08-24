# Sequences Feedback

## Context
The objective of the Feedback system is to provide information about which sequences (and paths in sequences) users prefer.

The feedback system is defined in 3 main parts:
* the step Feedback within sequences that defines what feedback to send 
* the user events api that will handle the feedback sent from the apps
* the tables filled from user events that will be used for later analysis


## End-to-End exemple

### 1. Define and write feedback sequences

Let's suppose you have a joke sequence called `SequenceWithFeedback` that actually ends with a leaf like that:
```
{
  "Type": "Leaf",
  "Comments":"this show how to use feedback",
  "Id": "MyCoffeJokeSequence",
  "Steps": [
    {
      "Type": "Text",
      "Label": {
        "en": "Someone enters a coffee,and..., splash",
        "fr": "C'est quelqu'un qui rentre dans un café, et... plouf"
      }
    }
  ] 
}
```

you want to get feedback, in order to do that you'll change your leaf in node and add a `LinksToFragment` pointing to a fragment containing the feedback:

```
{
  "Type": "Node",
  "Comments":"this show how to use feedback",
  "Id": "MyCoffeJokeSequence",
  "Steps": [
    {
      "Type": "Text",
      "Label": {
        "en": "Someone enters a coffee,and..., splash",
        "fr": "C'est quelqu'un qui rentre dans un café, et... plouf"
      }
    }
  ],
  "LinksToFragment": {
       "FragmentPath": "MyCoffejokeFeedback"
  }    
}
```

Then you write the feedback fragment with a binary question that will always point to only two feedback values `Good` or `Bad`:

```
{
    "Type": "Node",
    "Id": "MyCoffejokeFeedback",
    "Steps": [
      {
        "Type": "Text",
        "Label": {
          "en": "How did you like this joke ?",
          "fr": "comment as-tu trouvé cette blague ?",
          "es": "¿cómo conseguiste esa broma?"
        }
      }
    ],
    "Randomize": "true",
    "Commands": [
      {
        "Type": "Leaf",
        "Id": "MyCoffejokeFeedbackGood",
        "CommandLabel": {
          "en": "It was great !",
          "fr": "Super !",
          "es": "Estupendo !"
        },
        "Steps": [
          {
            "Type": "Action",
            "Name": "Feedback",
            "Parameters": { 
              "SequenceId" : "MyCoffeJokeSequence",
              "FeedbackValue" : "Good"
            }  
          }
        ]
      },
      {
        "Type": "Leaf",
        "Id": "MyCoffejokeFeedbackBad",
        "CommandLabel": {
          "en": "meh ...",
          "fr": "meh ...",
          "es": "meh ..."
        },
        "Steps": [
          {
            "Type": "Action",
            "Name": "Feedback",
            "Parameters": { 
              "SequenceId" : "MyCoffeJokeSequence",
              "FeedbackValue" : "Bad"
            }
          }
        ]
      }
    ]
  }
```

This fragment will be executed, send the choosen feedback as a specific user event and it will terminate the sequence.

note the feedback Step that contains the value to send as feedback and that will be sent:

```
{
    "Type": "Action",
    "Name": "Feedback",
    "Parameters": { 
      "SequenceId" : "MyCoffeJokeSequence"  <- this is the ID of the sequence you want to give feedback.
      "FeedbackValue" : "Bad",              <- this is always "Good" or "Bad"
    }
}
```

_Note that you can directly integrate in your bots this exemple for testing:_
* Sequence : [/data/bot/sequences/experiments/rui/MyCoffeJokeSequence.json](https://github.com/GhostWording/gw-config-apis/blob/master/data/bot/sequences/experiments/rui/MyCoffeJokeSequence.json)
* Feedback fragment [/data/bot/sequences/experiments/rui/MyCoffejokeFeedback.json](https://github.com/GhostWording/gw-config-apis/blob/master/data/bot/sequences/experiments/rui/MyCoffejokeFeedback.json)


### 2. Post event

As an app running a bot, when you'll execute the sequences and the feedback step, you'll have to post a userevent with special data relative to this feedback

The way you fill the [userevent payload](https://github.com/GhostWording/PublicDocumentation/blob/master/API/Sections/useractions.md) is the same as usual, only these fields are specific:

|  ActionType   |   TargetId | TargetParameter |  Context   |
|---------------|------------|-----------------|------------|
|  Feedback     | sequenceId | good/bad        | sequenceId |

which means that if you have the step in the previous paragraph with these values:
```
"Parameters": { 
  "SequenceId" : "MyCoffeJokeSequence",
   "FeedbackValue" : "Good"
}  
```

you'll have to post an event like that:

```
POST http://gw-usertracking.azurewebsites.net/userevent/batch

[
    {
      "ActionType":"Feedback",
      "AreaId":"testBot",
      "Context":"MyCoffeJokeSequence",
      "Last":1,
      "TargetId":"MyCoffeJokeSequence",
      "TargetParameter":"Good",
      "DeviceId":"device id",
      "FacebookId":"user fb id"
      //...(other usual fields like OS, version,etc...)
    }
]
```

### 3. Show query to read saved data

At this point you created feedback steps, you executed them and you sent the data. If you want to analyse your data, Feedback events are persisted in a specific table called `Votes.Feedback` with the same structure as userevents.

For exemple, imagine you want to count how many goods/bads you have and see first the sequences with more goods:
```
SELECT  Sequenceid = TargetId,
        Total      = Count(*),
        NbGood     = SUM(CASE WHEN TargetParameter='Good' THEN 1 ELSE 0 END),
        NbBad      = SUM(CASE WHEN TargetParameter='Bad' THEN 1 ELSE 0 END)
        
FROM Votes.Feedback
GROUP BY  
        TargetId 
ORDER BY NbGood DESC     
```

## Some additional Details

### Step Feedback

This step should be the terminal action of a sequence and replace and continue the actual leaves terminating the sequences. 

Step definition:
```
{
    "Type": "Action",
    "Name": "Feedback",
    "Parameters": { 
      "SequenceId" : "sequence id you refer",
      "FeedbackValue" : "Good|Bad"
    }  
 }
 ```
 
Please note: 

* that as in the exemple, `leaves` will became `nodes`.
* the only accepted values for the feedback are actually `Good` and `Bad`
* the sequence id is the id of the sequence you are trying to get feedback not the id of the last fragment. 
* you can write the sequence element containing the question and feedback step in a sequence after the supposed terminal leaf but it's cleaner to write it in a dedicated fragment (one feedback fragment per sequence) : 
   `"LinksToFragment": { "FragmentPath": "MyCoffejokeFeedback" }`
* in order to keep things clean, we'll use a common convention for the Id of the sequence element carrying good and bad values : `{feedbackFragement id}{good or bad}` (in our exemple with `MyCoffejokeFeedbackBad` it will be `MyCoffejokeFeedbackGood` and `MyCoffejokeFeedbackBad`)




