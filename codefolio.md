#Ashutosh's Code Portfolio

I’ve included various pieces of source code that I've worked on in the past. 

I've removed many of the files from the projects, deleted some important sections of code so that it can not be used as-is without significant effort.

I've included some details about some interesting and more algorithmic pieces of code I've written in the past to give a sense of my general competencies as a software engineer. 


## /scheduling_engine

__Overview__: This is one my favorite pieces of code and one that I'm particularly proud of because it works well at solving a very difficult problem, is well-written, and had very few bugs in production.

This code exposes an API for finding the best times to schedule a meeting given an arbitrary number of users, their calendar events, scheduling preferences, scheduling horizon, meeting duration and most importantly the location of the event or even locations.

The `scheduling_engine` can find the best slot in a day to schedule a meeting and make sure that all user's have enough time to travel to the meeting, it doesn't conflict, etc. There are several factors at play here.

Here's a demo video of how well it performs at scheduling location based events given real life constraints. [Demo Video](https://www.youtube.com/watch?v=yM2377M8zMk&feature=youtube_gdata_player)

The video is silent but here's a brief synopsis: What happens in this video is that I've got a calendar of events. One day has meetings on Sand Hill the other has meetings in SF. The calendar has lots of free time and some fully open days. The scheduling engine schedules the meeting at The Creamery on the SF day and the Sand Hill Road meeting on the Sand Hill day. It avoids scheduling it on days that are already empty. Happy to share more about the internals, what parameters it takes and how we've customized our scheduling engine to accept parameters for each user if you're interested. The hideous UI you see us using basically is how we tell it what we want, then the scheduling engine finds the best options, communicates it to our servers which then schedule it via google's apis.

__What it does__:
This code is still being used by Sunsama so I won't divulge in detail how it's used, you can browse the source code.

__Comments__: The package is found under `scheduling_engine/`.  My colleague Jacob and I pair-programmed the architecture of this module. I wrote all the algorithmic functionality myself.



## /date_range_parser

__Overview:__
The Date Range Parser is a standalone python module that takes inputs like *"sometime between Wednesday and Thursday in the morning"* and turns it into an list of *acceptable times*. This was used to interpret e-mails and automatically find times that fit a user's scheduling preferences.

__What it does:__
This code takes Natural Language text like *"sometime between Wednesday and Thursday in the morning"* and turns it into a python list of objects that contain start and end times.

The approach I took was to apply basic pre-processing, then I sequentially segment a sentence into what I called `Atoms` and then find common grammars of Atoms. For finding these grammars I creatively re-purposed NLTK's chunking functionality to define my grammars.

__(1) Pre-processing__
This step is wildly uninteresting, it involves adding spelling correction, removing stop words, and replacing numeric words like "seven" with their numeric representation "7". This is seen in `preprocessing.py`.

__(2a) Define Atoms and their Extraction Functions__

The approach I took was to analyze a sentence and break it into what I called *Atoms* (which you will find in `date_range_parser/atoms.py`). Each _atom_ is a particular grammatical feature encoded in its own python class. In the code attached I built the following atoms:

```
[
atoms.AbsoluteInnerDayModifierAtom,
atoms.AbsoluteOneWayInnerDayModifierAtom,
atoms.NaturalInnerDayModifierAtom,
atoms.RelativeOneWayMultiDayModifierAtom,
atoms.OperandAtom,
atoms.NaturalDaterangeAtom,
atoms.CalendarDateRangeAtom,
atoms.FillerAtom
]
```

Each of these atoms has an associated `@staticmethod` with a function called `def extract_atom(text, src_time, **kwargs_ignore):` so that a higher level function can attempt a sequential parse of a natural language string for atoms. Each atom has it's own precedence.

Each of the extraction functions are custom regular expressions except for one which relies on a parser from the github library [bear/datetime](https://github.com/bear/parsedatetime).

__(2b) Sequentially parse__

In `date_range_parser/extraction.py` I go through the natural language text and pick off Atoms. The atoms have precedence. What is returned is a list of tuples of word chunks with an associated tag, the chunk representing the natural language words that led to the detection of an Atom and a tag that represents the type of Atom detected.

This is all handled in the class `SequentialMatcher`. I've provided a highlight of how it works here from it's documentation.

```
class SequentialMatcher(object):
    """ SequentialMatcher applies a string matching function on an input string and returns
    an ordered list of matches and non-matches where each match as determined by the match_function
    is transformed using the function match_transformed before being placed in the result list.

    The final result looks something like this:
        >>> input_string = u'Some text some_matcheable_text more text other_matcheable_text'
        >>> chunks
        # If match_transformer is provided.
        ['Some text ', match_transformer('some_matchable_text'), 'more text ', match_transformer('other_matcheable_text')]
        # Otherwise the default behavior is a tuple (match_text, label).
        ['Some text ', ('some_matchable_text', label), 'more text ', ('other_matcheable_text', label)]

    Attributes:
        match_function (function): A python function that returns unaltered matched text.
        match_transformer (function): A python function that takes a single parameter `match_string` which
        is none other than the result of a match_function. The `match_string` is passed into the transformer
        before being inserted into the resultant list.
        label (function): If no match_transformer function is provided the default transformation will be to
        use a tuple (match, label) as the inserted result.
    """
```

__(3) Grammars__

Finally we use NLTK's chunking functionality to find common grammars and then based on the type of grammar that is detected to splice the dates and times into a list of start and end datetime objects. You can find this in `date_range_parser/grammar_py`

__Comments:__ I was the only author for this entire package and none of this code has been reviewed by my colleagues.

## /recommendation
__Overview:__
I built the NetworkEngine module for Sunsama to take a list of a users contacts, calendar contacts and connections within the app to create recommendations on who to connect with. This is basically a recommendation engine for "__people you may know__".

__What it does:__
This code takes care of pulling all of the relevant contacts and users. Then it creates a feature vector for each *edge* in the network of users. The feature vectors contain a host of information such as mutual connections, shared calendar events, etc. This vector is then transformed to a scalar value via a linear combination of weights.

__Comments:__ I've included two files from the module that highlight the main algorithmic components of what I built. This code was written entirely by me but my colleague Michael Stuecheli pair programmed it with me.

You can find it in the files: `recommendation/suggestion_engine.py` and `recommendation/network_engine.py`


## /reachd

This was the app I sent you a video of. The source code for the iOS and python backend are included. I've left most of the main source files in tact for your viewing pleasure. This app makes use of iOS's location services as well as geofencing that happens on the backend. 

## /yogg

Yogg is an experimental app I built, it uses Twilio and SMS heavily. It makes heavy use of location services, communicating it to the python-flask backend and triggering location based events on the backend. Instead of doing geofencing on the iOS app itself, the geofencing is performed manually on the backend so that users can leave location based reminders for one another. 


## /assistant_parser

This code takes inbound e-mails when an assistant is CCed, reads the e-mail, parses it and sends a structured command to the main Meteor server asking it to perform the scheduling task.

The code is found in `assistant_parser/`

The Natural language processing is quite simple here and is performed in `assistant_parser/utils/nl_parser.py`

## /lumo

I've provided a few short snippets of C code that I've written and deployed on production level hardware when I worked at Lumobodytech. It's mostly to demonstrate that I can write algorithmic code in C.

