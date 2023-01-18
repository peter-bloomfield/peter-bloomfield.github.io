---
layout: post
title: Array of Checkboxes in LimeSurvey
date: '2009-04-24 14:29:13'
redirect_from:
- /array-of-checkboxes-in-limesurvey
---

I’m using [LimeSurvey](https://www.limesurvey.org) to setup and conduct a survey as part of my PhD, and it’s working very well. (Great work, LimeSurvey developers!) It’s free and open source, and since I have my own server I can host the survey myself at no additional expense. That wouldn’t be the case if I used something like SurveyMonkey.

However, after discussion with a very helpful statistics guru here at <acronym title="University of the West of Scotland">UWS</acronym>, I found that I needed a question type LimeSurvey didn’t seem to support: an array of checkboxes.

## Background

Here’s the kind of question I was hoping to display:

```
Q: Which of these features have you used as a student and/or as a teacher?

               Student   Teacher
1. Forum       []        []
2. Chatroom    []        []
3. Wiki        []        []
4. Quiz        []        []
```

It’s like the “array” question type, but each `[]` is a checkbox instead of a radio button. I’m basically tracking two independent boolean variables for each category, instead of a single variable on a scale.

## The solution

After some very helpful (and very prompt) support on the LimeSurvey forums, it turns out that this question type _is_ possible, but I had to upgrade to LimeSurvey v1.81. (I had previously been running v1.72). With the new version, here’s how it’s done:

1. Create a label set for your columns of checkboxes
2. Add a new question (fill in the usual Code and Question boxes)
3. For Question Type, select “Array (Multi Flexible) (Numbers)
4. Select the label set you created above
5. Under Question Attributes, select “Checkbox layout”, and enter 1 in the box besides it
6. Click “Add question”

And that’s it! It’s not immediately obvious at first, but pretty easy once you know how. You might find that the checkboxes take up a lot of room if you only have a few. For 2 columns of checkboxes, I found that an “answer\_width” of 50 (under the question attributes) made the layout a little better.
