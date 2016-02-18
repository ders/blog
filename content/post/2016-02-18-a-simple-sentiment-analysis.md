+++
Description = ""
Tags = []
date = "2016-02-03T17:51:00+09:00"
title = "A simple sentiment analysis of two US presidential candidates"

+++

**Goal:**  To do some basic [sentiment analysis](https://en.wikipedia.org/wiki/Sentiment_analysis) on video content.

**Test cases:**  Two 5-minute clips of US presidential candidate speeches.

**Strategy:**

* Extract 5 minutes of audio from the beginning of each video.
* Generate a transcript using a speech-to-text program.
* Feed the transcript into a sentiment analyzer.

## The original content

<iframe width="560" height="315" src="https://www.youtube.com/embed/qOQCw7Hcwic" frameborder="0" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/p5ZB8Lg1tcA" frameborder="0" allowfullscreen></iframe>

## Extracting a 5-minute audio clip

There are many ways to do this.  One way is to download the video using a browser add-on.  Browser add-ons are easy to find but are also fickle, as they make it easy to download material in violation of copyright.  And if you're downloading from YouTube, you're violating their terms of service, even if you're not infringing on copyright.  (We maintain that this exercise falls under [fair use](https://en.wikipedia.org/wiki/Fair_use).)

Another way is to turn on audio capture while playing the video.

After the capture is complete, we'll want to convert to FLAC if we're not there already.  We'll use [ffmpeg](https://ffmpeg.org/) for this, e.g.:

    ffmpeg -i captured-content.mp4 captured-content.flac

And then to extract the first 5 minutes:

    flac --until=5:00 captured-content.flac -o five-minute-clip.flac

Both `ffmpeg` and `flac` are available via homebrew.

## Generating a transcript

The IBM Watson Developer Cloud has a [speech-to-text](http://www.ibm.com/smarterplanet/us/en/ibmwatson/developercloud/speech-to-text.html) service which is available through an API and also has a [demo page](https://speech-to-text-demo.mybluemix.net/).  In theory, one can get limited free access to the API after going through a mildly annoying sign-up process, but in practice I was unable to convince the API to accept the credentials I'd obtained.

Fortunately, the demo page allows file uploads and produced the following transcripts from the content above:

**Trump**

> This is no way I'm leaving South Carolina. And I was gonna leave for tonight come back as it upsets up saying you have a five days we got a win on Saturday we're going to win. Make America great again we're gonna make America. We're going to win. You know. It's been an amazing friend Ruben all over the state today I love you too girly looking out dnmt love you I love you all. I love you. So many things are happening for a country. And it's this is a movement time magazine last week at the most beautiful story cover. And they talk about its improvement they've never seen they say there's not been anything like this I don't know ever but they actually say ever. We went to Tampa the other day Tampa Florida would like two days notice fifteen thousand people that are turned away five thousand and by the way for all of the people in this room I can't believe it this is a huge room but downstairs to filling up another one and there sadly sending people away we don't like that right. No okay why don't we all get up go let's have that now. Now we have one of the great veterans outside I'll stand up while one of the great great you are great. Love this guy. He loves the veterans and I love the veterans are we going to take care of our veterans I'll tell you that we're going to take it did not. They are not properly taken care of so we're going to take a right we have sent a look at this. I knew you guys would say that I can spot a veteran a long ways off. But we are we going to take a break here we're gonna take you have a military we're going to take you have a military because our military is being whittled away whittled away we're going to make our military so big so strong so powerful nobody's going to mess with us anymore nobody nobody. Nobody. So Nikki Haley a very nice woman she better speech the other day you saw that and she was talking about anger and she said there's a lot of anger and I guess she was applying all of us you know really referring to us. And by the end of the day she was actually saying that Donald Trump is a friend of mine he's been a supporter of mine everything else you know the tone at the beginning was designed by the time that she was just barraged with people she said I think we better change our path here. And by the end of the day and it was fun it was great but she said you know there is anger but I said there is a group and I was asked during not this debate but the previous debate I was asked. I by the way did you love this last debate dnmt. Listen to like. They came at me from every angle possible. Don't know they came out before every angle you know sort of interesting. They were hitting me with things like and such until as you know I never realized I've always don't politicians it is honest but I've never known the level of dishonesty. And I deal in industries a lot of different but mostly real estate and like in Manhattan at different places but I've never seen people as dishonest as politicians. They will say anything. Like okay so a lot of you people understand that you you get when you've seen the speeches that you see in a lot of it and you know that I protect the second amendment more than anybody by far dnmt more than. And this guy Ted Cruz gets upset Donald Trump does not respect a second amendment and the more that anybody I'm with the second amendment. I saw no no it's lies. And then they do commercials and you know he did it to Ben Carson and him in particular in all fairness. Jeff is represents but these are minor misrepresentations and he's not going anywhere anyway so what would how casual. Not as. Well Jeb was talking about eminent domain Donald Trump used eminent domain privately then I see there's a big story I had to bring this out luck. Proof Jeb bush under eminent domain took a disabled veterans property. Something about me. No state. Honestly these guys are these guys are the worst. Eminent domain without eminent domain by the way you don't up highways roads airports hospitals you know not bridges you drive anything so. They say Donald Trump does look like Eminem and I don't even tell me but you need to road you need a highway you need you know it's funny they all want they all want the keystone pipeline right but without eminent domain without think of it without eminent domain you can't have the keystone pipeline and we're going to get the keystone pipeline approved but but fluids jumps. It's jobs but remember this when it gets approved a politicians go to baby approve it.

**Sanders**

> President Falwell and. David. Ok thank you very much for inviting my wife Jane and. Ought to be with you this morning we appreciate the invitation. Very much. And let me start off by acknowledging what I think. All of you already know. And that is the views. That many here at liberty university have. And all I. On a number of important issues. A very very different. I believe in women's rights dnmt. In the light of the woman to control her own body dnmt. I believe in gay rights. And now. Those of my views. And it is no secret. But I came here today. Because I believe from the bottom of my heart. That it is vitally important for those of us. Who hold different views. To be able to engage in any civil discourse. Who often in our country and I think both sides. Bear responsibility for us. There is too much shouting at each other. There is too much making fun of each other. Now in my view then are you can say this is somebody who whose voice is hoarse because I have given dozens of speeches. And the last few months it is easy. To go out and talk to people who agree with you are missing Greensboro North Carolina just last night. Alright. We are nine thousand people out. Mostly they agreed with me tonight. We're going to be a Manassas and thousands out they agree with me. It's not a whole lot to do. That's what politicians by and large do we go out and we talk to people who agree with us. But it is harder. But not less important. For us to try and communicate with those who do not agree with us on every issue. After. And it is important to see where if possible and I do believe it's possible we can find common grounds. No liberty university. Is a religious school obviously. Pn. All of you are proud of the. You already school. Which as all of us in our own way. Tries to understand the meaning of morality. What does it mean. To live a moral life. And you try to understand in this very complicated modern world that we live in. What the words of the Bible me in today's society. You are in school which tries to teach its students. How to behave with decency and with honesty and how you can best relates. To your fellow human beings and I applaud. You for trying to achieve those goals. Let me. Take a moment. Or a few moments. To tell you what motivates me. And the work that I do. As a public servant as a Sentinel. From the state of Vermont. And let me tell you that it goes without saying I am flaws foh throw me being a perfect human being. But all I am motivated by a vision.

## Sentiment Analysis

Again, there are a variety of tools to do this, including the [Natural Language Toolkit Project](http://www.nltk.org/), a free python library. Taking advantage of a [simple demo site](http://text-processing.com/demo/sentiment/) which uses the NLTK, we can see that both Sanders and Trump are polar, but Sanders is more positive.  Who would've known?

**Trump**

* Overall: negative
* Subjectivity
  * neutral: 0.2
  * **polar: 0.8**
* Polarity
  * pos: 0.4
  * **neg: 0.6**

**Sanders**

* Overall: positive
* Subjectivity
  * neutral: 0.2
  * **polar: 0.8**
* Polarity
  * **pos: 0.8**
  * neg: 0.2

For the adventuresome, [here are more detailed instructions](http://www.nltk.org/howto/sentiment.html) on using the NLTK for sentiment analysis.
