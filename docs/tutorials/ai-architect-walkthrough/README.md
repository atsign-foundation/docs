---
description: >-
  This walkthrough guides you through loading a Blueprint and exporting a prompt
  using AI Architect, then using that blueprint to generate and run an
  atPlatform app in your IDE.
icon: arrow-progress
---

# AI Architect Walkthrough

{% embed url="https://vimeo.com/1167932458" %}

### Pre-requisite - Get Your Starter Pack Atsigns <a href="#step1_atarchitect" id="step1_atarchitect"></a>

Before building or testing any atPlatform app, you need two Atsigns. These act as the identities your app will use during testing and development.

1. &#x20;Visit [**my.atsign.com/starterpack**](https://my.atsign.com/starterpack)**.**
2. &#x20;Verify your email and claim your two free starter-pack Atsigns.

If you already have Atsigns, you can log into [**my.atsign.com/login**](https://my.atsign.com/login) to access them.

{% hint style="info" %}
These Atsigns will be used later when you run your generated app to test authentication and secure communication.
{% endhint %}

{% stepper %}
{% step %}
### [Open AI Architect and Load the Example Blueprint](./#open-ai-architect) <a href="#open" id="open"></a>

Load the example so you can see how a Blueprint is structured.
{% endstep %}

{% step %}
### [Export the Prompt](./#id-2.-export-the-prompt)

Export the prompt that will be used to tell you LLM exactly what to build.
{% endstep %}

{% step %}
### [Open Your IDE, Plan and Code your App](./#id-3.-open-your-ide-plan-and-code-your-app)

Open your IDE, set your LLM to plan, and once happy, let it generate the code.
{% endstep %}

{% step %}
### [Build and Run the App](./#id-4.-build-and-run-the-app)

Run the app to make sure everything works as expected.
{% endstep %}

{% step %}
### [Create your own Blueprint](./#id-5.-create-your-own-blueprint)

Make a Blueprint for your own idea and repeat the process.
{% endstep %}
{% endstepper %}

***

### 1. Open AI Architect and Load the Example Blueprint <a href="#open-ai-architect" id="open-ai-architect"></a>

AI Architect is the visual blueprinting tool used to design your app’s structure before generating the LLM prompt.

1. Go to [**aiarchitect.atsign.com**](https://aiarchitect.atsign.com/). This opens the workspace where you create or load a Blueprint. The Blueprint you create here becomes the input for your LLM-powered code generation.
2. Click on **Start with Demo Blueprint** to load our prebuilt example Blueprint. AI Architect will populate the canvas.

{% hint style="info" %}
A Blueprint is a visual map of your application. Each box represents a node. This could be a person, a process, an AI agent, a service, or any other entity involved in your system. The lines between nodes show how information flows from one part of the system to another.

Every node includes a Notes section, which acts as the node’s job description. This is where you define:

* what the node is responsible for
* what information it needs
* how it behaves

A clear Blueprint gives the LLM the structure it needs to build your application in stages.
{% endhint %}

### 2. Export the Prompt

Once the blueprint is loaded:

1. Select **Export  Prompt.**
2. Copy the generated prompt to your clipboard.

Export Guide generates the full LLM prompt based on your blueprint.&#x20;

{% hint style="info" %}
The prompt includes:

* A high‑level application description
* A breakdown of all nodes (people, processes, things)
* A breakdown of all connections and their types
* atPlatform roles for each component
* Implementation notes for each node
* Stream/notification patterns for each connection
* Required dependencies and initialization code
* Authentication setup
* A step‑by‑step implementation guide

This becomes the instruction set your LLM will follow to generate the full application.
{% endhint %}

### 3. Open Your IDE, Plan and Code your App

{% hint style="success" %}
You can use any IDE you like but we recommend [Visual Studio Code](https://code.visualstudio.com/). When selecting LLMs we have had the most success with the following:

* Claude Sonnet 4.5+
* Claude Opus 4.5+
* Gemini 3
{% endhint %}

Visual Studio Code gives you the flexibility to work with a variety of LLMs, not just one. Depending on the extensions you install, you can choose from models like OpenAI, Gemini, Claude and others.&#x20;

To get started, you’ll need to create a new empty folder and set your LLM to **Plan Mode.** Starting in Plan Mode is important because it helps to ensure the LLM is going to build what you want it to build.

1. Paste the exported prompt directly into the chat window. It will plan the project and present the plan to you. When you are happy, Proceed with implementation and it will create files, and set up the app.&#x20;
2. You will be asked to confirm certain actions (file creation, folder setup, dependency installation).
3. The LLM may build the application in stages, allowing you to test each step and provide additional instructions. It will continue refining and completing the app based on your original prompt as you guide it through each iteration.

{% hint style="info" %}
The LLM will:

* Parse the entire blueprint
* Create the full folder structure
* Generate Dart/Flutter code using at\_client and related packages
* Implement each node as a module or service
* Implement each connection using streams or notifications
* Set up authentication, onboarding, and identity management&#x20;
{% endhint %}

### 4. Build and Run the App

Once the code generation is complete:

1. Follow the build instructions created by your LLM.
2. Run the app on two separate devices/simulators/emulators.
3. When prompted, activate or sign into the app using your two starter-pack Atsigns, one for each device. If you need to access them, log into your [**Atsign Dashboard**](https://my.atsign.com/login).
4. Test sending messages between the two Atsigns.

{% hint style="info" %}
This validates:

* Atsign onboarding
* Secure key management
* Encrypted messaging
{% endhint %}

### 5. Create your own Blueprint

Once you’ve explored the example Blueprint, you’re ready to create your own. Start Simple. A Blueprint doesn’t need to be perfect on the first pass. Its purpose is to help you think clearly about how your application works and to get you to working code quickly and securely.

When designing your Blueprint, focus on three core questions:

1. What are the nodes?
2. What does each node do?
3. How does information move between them?

These three decisions form the foundation of your application’s architecture and guide the LLM as it builds and refines your app.

### Support and Further Help

_If you run into issues, have questions about any step, or want to go deeper into building with the atPlatform, the Atsign team can help. Contact support@atsign.com._
