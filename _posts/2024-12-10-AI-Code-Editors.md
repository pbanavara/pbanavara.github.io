### My journey with AI Code Editors

I got introduced to cursor at a hackathon in October. The ease of development was obvious and I jumped on the bandwagon. I tried cursor for a few days and happened to attend another talk hosted by [Chroma](https://www.trychroma.com//) where the CTO of Sourcegraph Beyang gave a talk. I wasn't new to Sourcegraph and quickly discovered that they had a version of their AI code editor as well. It was far cheaper than Cursor at $9 with almost the same features, afterall both rely on Claude Sonnet. 

My productivity immediately soared. I could do not only code development but also get design recommendations. I wish [Cody](https://sourcegraph.com/cody) maintained a history to allow me to go back and pull out some of the design recommendations Cody had provided. Some of these were:
* A serverless cost effective alternative to hosting apps on a VM.
* A very easy redis deployment that was free and on cloud - absolutely no setup needed.
* A scalable pub-sub architecture based on simple hashing and a simple message queue.
  * Have multiple message queues and route messages to each based on the hash of the message. Like a crude load balancer.
  * A fully stateless architecture allowing for workers to pop in and out of the system as needed.

I was not only able to validate my design but also could get code snippets to quickly experiment and deploy. What a fantastic tool to go about system design.
My usage naturally increased and by November I was building my ideas fully on Cody. As with all tools, it took me a while to get to the aspects of Cody that were not what I expected. I think, the tool sets up an expectation in your mind that it can solve anything under the sun. The initial euphoria sort of led me in this direction.

The first gotcha was in deploying my application. It's a FastAPI backend app powered by websockets for real time communication. I followed the instructions provided by Cody and deployed the app.


I'd then run into this error when I ran `wscat`
```
2024-12-10 07:07:27.021 PST
WARNING: Invalid HTTP request received.
```
And I'd ask for recommendations and Cody would essentially go into the same loop of checking for logging, rebuilding and re-deploying.
![Imgur](https://i.imgur.com/gJHtFOm.png)

This went on for about 2-3 hours. I was reluctant to Google and was pushing Cody really hard to see if it can figure out what was wrong.

Then I askked for a couple of alternatives and Cody was super helpful here.
![Imgur](https://i.imgur.com/32kx7l9.png)
Then again if you examine closely this app.yaml is incorrect. I found this out later. I deployed on Google app engine and the first error I saw was
```
ERROR: (gcloud.app.deploy) INVALID_ARGUMENT: Error(s) encountered validating runtime. Your runtime version for python39 is past End of Support. 
```
Cody recommended I upgrade to the Python 311. I countered and asked shouldn't it be 312 and look at Cody's response.
![Imgur](https://i.imgur.com/iAw2Y20.png)

Wow, the conviction was mind blowing. But this turned out to be completely wrong. The correct app engine config should have
```
runtime_config:
  operating_system:  "ubuntu22"
  runtime_version: "3.12"
  ```
Well, one could argue that the creators of App Engine can't fix a simple error message and that's fair. But Google had the answer and Cody didn't. 

So at this time, I realized I am relying too much on Cody and ditched the App Engine deployment as it would be simply way too expensive at this time.

Went back to Google cloud run and checked Google for the 502 error message I was getting. Apparently you have to disable end to end http2 in your deployment.

![image](https://i.imgur.com/czd49tx.png)
 
 It took me nearly 2 hours to debug this. Now I am no debugging expert and I have lost my 20s style of aggressive debugging but this was bonkers. Google cloud has to win the award for the most confusing error messages ever. Their log messages are also so misleading for instance
 ```
 (gcloud.run.deploy) Revision 'realestate-00002-zw6' is not ready and cannot serve traffic. The user-provided container failed to start and listen on the port defined provided by the PORT=8080 environment variable within the allocated timeout. This can happen when the container port is misconfigured or if the timeout is too short. The health check timeout can be extended. Logs for this revision might contain more information.
 ```
 The actual reason for this error turned out to be that I had deployed the Mac docker image on Google cloud. So Cody somehow had this data. This is such a comical error message so easy to fix - I mean, check the damn image and raise a flag saying this image is not built for Linux. Period. 
 Sure you can check logs and see a format error and all that but why introduce friction ?

 So after wrangling with this for nearly 3-4 hours I finally got it working. I know that editors like Cody are still not there when it comes to debugging but the day isn't far off. It's just a matter of the langugae models indexing all that stackoverflow data. 
 
 The day is not far off when [Devin](https://devin.ai/) and [Cody](https://sourcegraph.com/cody) become the best pair programmers.

