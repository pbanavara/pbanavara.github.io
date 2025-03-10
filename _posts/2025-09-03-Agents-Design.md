### Designing Lightweight asynchronous AI agents
The concept of an AI agent goes back to at least 2015-16 when the term was first coined by [Bengio et al](https://arxiv.org/abs/1511.07275).
From the paper
"We present an approach for learning simple algorithms such as copying, multi-digit addition and single digit multiplication directly from examples. Our framework consists of a set of interfaces, accessed by a controller. Typical interfaces are 1-D tapes or 2-D grids that hold the input and output data. For the controller, we explore a range of neural network-based models which vary in their ability to abstract the underlying algorithm from training instances and generalize to test examples with many thousands of digits. The controller is trained using Q-learning with several enhancements and we show that the bottleneck is in the capabilities of the controller rather than in the search incurred by Q-learning."

Fast forward to 2025 and the concept remains the same. 
What has changed or advanced are the following:
* The input and output interfaces
* The controller

#### Input and output interfaces
What was as simple as 1-D tapes and 2-D grids has evolved into embeddings.
In the same dimensions we are now able to encode much more information since we are mostly dealing with natural language.
We are also able to provide prgrammable functions as interfaces. This gives the controller enhanced capabilities to direct and manage tasks that were hitherto unfathomable. 

#### The controller
The controller is the main component of the agent. It is the brain that makes decisions. This brain is mostly powered by the intelligence of a LLM. One could argue that the controller by itself can function as an agent eventually.

"For this to become a reality, the controller would need an infinitely long context window—an impractical requirement even for the human brain. Instead, intelligent retrieval mechanisms and memory-efficient architectures will likely bridge the gap."

That begs the question, if the designers of the controller can design agents by adding the peripheral input/output interfaces for the agent to execute in the real world. The answer is yes, as we are seeing the top LLM companies coming out with their own agentic workflows and agents.

#### Agentic design
One of the cornerstones of agentic design is planning. For an agent to perform any task it needs a plan of action and this has to be defined upfront. Somewhat like a DSL. This reminds of how you had to define the graph in tensorflow before eager mode became a reality. I see this as a major limitation. Humans don't need a plan. You can pick up any arbitrary task and start executing. Sure, a plan really helps but it's not a requirement or a mandate. For an agent to be designed well, in any framework today - you need to define a plan. 

Hence you have agents that are tightly coupled to plans. For every task you need a new agent. This is regardless of the agentic framework at play whether it is [Smolagents](https://github.com/huggingface/smolagents) or [langchain](https://www.langchain.com/). 

The one advantage [Smolagents](https://github.com/huggingface/smolagents) offer is that you can prompt the controller to respond with blocks which the agent can execute directly, thereby giving more flexibility for the agent. However, the agent still needs a plan for execution and is tightly coupled to that plan.

#### Practical limitation
Let's look at smolagents. This is in no way meant to poke holes in their framework. I use smolagents quite a lot so it's just familiarity. 

```python
search_tool = DuckDuckGoSearchTool()

agent = CodeAgent(
    tools=[search_tool],
    model=HfApiModel("Qwen/Qwen2.5-72B-Instruct"),
    planning_interval=3 # This is where you activate planning!
)
```
Now if I want to use this agent for a task that doesn't involve web search , I need to create a new agent. 
So for every task, I end up creating a new agent. 
Now this is cool if we are building only one agent. But in reality you will build a constellation of agents so now you have to endure a new abstraction 'Manager agent'.

#### A new design paradigm
Given the limitations of the tight coupling between an agent and the workflow, why not design agents that are more dynamic and are not tied to workflows.

Can we build an actor like model that can subscribe to a series of events and execute the messages in those events ?

#### An event driven agent
The actor model is a design pattern that allows for asynchronous communication between components in a system. It is based on the idea of a message passing system where components can send and receive messages to communicate with each other.

This was primarily used first in the Erlang programming language and later adopted in other languages such as Elixir and Ruby.

The actor model is based on the concept of actors, which are entities that can send and receive messages. Each actor has a mailbox where it stores incoming messages, and it can process these messages asynchronously. Actors can also create new actors and send messages to them.

What if an agent can be an actor ? Instead of a planned set of tasks that the agent needs to execute, the agent can subscribe to a series of events and execute the messages in those events.

#### The actor model in action
Let's take a look at an example of how the actor model can be used to build an agent that can perform a task.

```python
class Agent(Actor):
    def __init__(self):
        self.state = "idle"
        self.tasks = []
        self.current_task = None
        self.current_task_index = 0
    
    def receive_message(self, message):
        if message == "start":
            self.state = "executing"
            self.current_task = self.tasks[self.current_task_index]
            self.current_task_index += 1
            self.current_task.execute()

        elif message == "task_completed":
            self.state = "idle"
            self.current_task = None

        elif message == "add_task":
            self.tasks.append(task)
            self.state = "idle"

        elif message == "stop":
            self.state = "stopped"
            self.current_task = None
            self.current_task_index = 0
            self.tasks = []

```
Now the tasks are decoupled from the agent. The task designers are free to define the tasks and the agent can freely execute these tasks at scale without any coupling.

#### The good old pub sub model
The publish-subscribe (pub-sub) model is a foundational design pattern for asynchronous communication between agents. The core idea is that publishers generate messages, and subscribers listen for relevant messages without being directly coupled to the publisher.

For AI agents, pub-sub allows them to react dynamically to events instead of following a rigid execution plan. Depending on the degree of coupling between agents and messages, we can take two primary approaches:

***The MQTT-Style Model (Loose Coupling, 1:Many Mapping)***
This approach follows a message-agnostic, event-driven paradigm, where:

* Publishers post messages to broad topics without needing to know which agents will consume them.
* Subscribers listen to topics but are responsible for interpreting the messages and determining their next action.
* Messages are typically key-value pairs, JSON payloads, or blobs, allowing flexibility at the cost of stricter guarantees on message format.

Pros
* Highly decoupled: Producers do not need to know about consumers.
* Scales well: Many agents can subscribe to the same event stream.
* Flexible: Allows for different types of subscribers with varying logic.
 
Cons
* Parsing overhead: Each agent must interpret messages dynamically.
* Risk of message format inconsistencies: Without strict schemas, errors may arise.
* Limited real-time guarantees: Agents may introduce latency in processing.

***The ROS-Style Model (Strong Typing, 1:1 Mapping)***
This approach enforces strict message structure, leading to a deterministic and type-safe system, where:

* Each topic has a well-defined schema.
* Agents subscribe to topics knowing exactly what data format to expect.
* Message payloads follow structured definitions, ensuring high reliability.

Pros
* Deterministic and reliable: Agents know the exact structure of messages.
* Efficient processing: No need to parse unstructured data.
* Easier debugging and validation: Errors in message format are caught early.
  
Cons
* Less flexible: Any change in the message format requires updating all subscribers.
* Tighter coupling: Agents must be explicitly designed to handle specific message types.
* More rigid scaling: Requires schema management as new agents are added.

May be there is some scope for a hybrid approach. 

#### The future
The pub sub model is not a panacea and is not to be treated as such. It introduces a whole new array of problems. 
* How to handle backpressure ?
* How is traceability handled ?
* How to handle message loss ? Retransmission ? Exponential backoff

These are not simple problems but there are existing paradigms that have solved this at scale. It will be easier to adopt those paradigms than to struggle with tight coupling.

Then you have the likes of [CrewAI](https://github.com/CrewAI/CrewAI) and a whole gamut of UI based agents such as [Claude's computer use](https://www.anthropic.com/news/3-5-models-and-computer-use) and [OpenAI's operator](https://openai.com/blog/chatgpt-plugins) which solve agent orchestration just based off of a command or a prompt.

Looking at all these challenges, it feels like we are just scratching the surface of agentic AI. Exciting times ahead!





