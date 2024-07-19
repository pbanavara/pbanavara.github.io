---
layout: post
title: Evolution of my startup idea
---

### The initial premise
Around March 2024, I toyed with the idea of building  an AI coach for
tennis. Why tennis of all sports. Why not golf or badminton or other
racket sports.I don't have an answer to this. I found tennis to be a
far more technical sport than badminton or ping pong. Also it felt way
harder to learn than any other sport I have played. So I thought I
will build out an app that will be a very basic version of a coach. I
did some basic market research and found out that there was a play
here. Based on the total number of tennis players and enthusiasts this
seemed like a 10-100M$ idea at the very least. 

### The MVP
So what should the MVP accomplish. It should be able to view the
player like a coach does and offer corrections to the shot. So here
are the basic requirements for the phone MVP

* Record the play using the phone cameras.
* Get audio feedback.
* Ensure that the app doesn't drain the phone battery in an hour or so.

### MVP first iteration

* Recording the video was quite straightforward. After a couple of
  iterations I figured the phone is best positioned facing the
  player's back.
  
* Audio feedback - This became the crux of the app. What feedback to
  give ? At this time I was also looking at adjacent apps like
  SwingVision, SevenSix etc and trying to figure out why they have
  been unable to scale. I looked at the reddit forums for r/10s and
  found that most users of SwingVision used it purely for recording
  purposes and almost never used any other feature. 
  
  Two things came quite clear
  * There is no point just recording your game and extracting the
    highlights - SwingVision does this.
  * Players need real feedback. WHat am I doing wrong and how to
    correct. Most of the r/10s questions are around this. They post a
    video and ask for form correction.

### Pose identification

* There are models that already identify poses so I set out using one
  of these yolov8. In hindsight I should have just used Google
  mediapipe. The reason I didn't go for mediapipe is that the way they
  layered the poses on the videoPreview layer, it was not possible to
  record those frames. So while you'd get a live preview you wont have
  a recording unless you did something like screen recording on the
  phone.
* I finetuned a yolov8 model to run on the iPhone as I didn't want to
  waste GPU resources on the server. All good. I could even overlay
  the poses and record them.

### Pose correction

This is a prediction problem. So now I need to figure out when a
player takes a shot, what are the mistakes in the poses. One of the
ways of doing this
* Find a similar shot from a pro through video search in a small
  sample size. Not web search.
* Extract that shot video from the large game video.
* OVerlay the pose angles on this target video.
* Compare the shot frame by frame with the player (source)  shot
  video. 
* Measure the angle deltas right from the first split step till the
  ball makes contact with the racket and the swing is completed.

Each of these is a large problem statement on their own. 
* Video search is no joke, even in a small sample size.
* Finding the racket ball contact point - again though simple needs
  resources for fine tuning object detection and tracking. Given that
  these shots are so fast, it's almost impossible to train a model to
  predict the contact point. I tried a few iterations but the accuracy
  was miserable.

So I parked this idea and went back to customer discovery. It became
apparent that there is a simpler use case in gyms. 

### Pivot 1
Compound exercises such as squats, deadlifts, kettlebell swings need
to be done in a proper form
* A to avoid injuries
* B to get the most benefit from the movement.

So this became a straightforward pose detection and rule application
for correction as the rules are quite simple. Measure the key angles
* Hip hinge.
* Knee angle.

Identify the one perfect angle required for each exercise and visually
indicate the angle on the workout video frame.

I built this app and released a beta.

### Customer discovery v/s customer usage
No takers, very luke warm response. Why would someone who cared so
much about form correction not install and use this app. At least give
it a try. My beta stats

* 10 users, average 1-2 sessions per user. No growth despite posting
  on channels such as [buildspace](https://buildspace.so/nw)

### Analysing the poor uptick
So why is this ? Turns out, not many people like to keep their phones
in front of them or by their side during workouts. I myself wouldn't
do it after 10 workouts or so. So the phone placement is a major
friction point and almost no one overcame this friction
consistently. 
Also not many fitness enthusiasts cared about form correction all the
time. It was good once they identified their form and then they are
set. 

### What fitness enthusiasts really need ?
As evidenced by workout apps such as [Tonal](https://www.tonal.com/)
and [Tempo](https://tempo.fit/), those who workout need
recommendations and a change in their routine often. It is very hard
to do the same workout consistently for months. Even though,
statistically you are better off just doing the same workout
consistently instead of switching routines.

### Next steps
So where does that leave me. Here are my current thoughts.

* Despite all these apps, nearly 75% of adults don't workout
  consistently. I am yet to find data for 4-5 year time period. Per
  [CDC}(https://www.cdc.gov/nchs/products/databriefs/db443.htm#:~:text=Among%20all%20adults%20aged%2018,of%20activity%20(Figure%201).)
  only 24% of the adults meet their fitness criteria which is about 20
  minutes of moderate exercise every day or 150 mins per week.

* So there is a need for an application that can help people to
  maintain this workout routine for years. These apps are clearly not
  cutting it. 
  
* There has to be a free app that is ad supported to make this work at
  scale. And it should have no friction points of video or camera
  placement etc.
  
* The answer is perhaps in sensors and video generation using sensor
  data. People still need videos to show off, but they don't need the
  hassles of recording, placing the phone at some specific angle etc.
  
### My new pitch
workout just wearing an apple watch and get a summary video to post on
social media.













