---
description: >-
  This walkthrough guides you through loading a Blueprint and exporting a prompt
  using AI Architect, then using that blueprint to generate and run an
  atPlatform app in your IDE.
icon: arrow-progress
---

# Getting Started with AI Architect

{% stepper %}
{% step %}
### [Open AI Architect and Load the Example Blueprint](getting-started-with-ai-architect.md#open-ai-architect) <a href="#open" id="open"></a>

Load the example so you can see how a Blueprint is structured.
{% endstep %}

{% step %}
### [Export the Prompt](getting-started-with-ai-architect.md#id-2.-export-the-prompt)

Export the prompt that will be used to tell you LLM exactly what to build.
{% endstep %}

{% step %}
### [Open Your IDE, Plan and Code your App](getting-started-with-ai-architect.md#id-3.-open-your-ide-plan-and-code-your-app)

Open your IDE, set your LLM to plan, and once happy, let it generate the code.
{% endstep %}

{% step %}
### [Build and Run the App](getting-started-with-ai-architect.md#id-4.-build-and-run-the-app)

Run the app to make sure everything works as expected.
{% endstep %}

{% step %}
### [Test the App with Your Atsigns](getting-started-with-ai-architect.md#test-the-app-with-your-atsigns)

Authenticate using your Atsigns and test the functionality
{% endstep %}

{% step %}
### [Create Your Own Blueprint](getting-started-with-ai-architect.md#id-5.-create-your-own-blueprint)

Make a Blueprint for your own idea and repeat the process.
{% endstep %}
{% endstepper %}

***

### 1. Open AI Architect and Load the Example Blueprint <a href="#open-ai-architect" id="open-ai-architect"></a>

AI Architect is the visual blueprinting tool used to design your app’s structure before generating the LLM prompt.

1. Go to [**aiarchitect.atsign.com**](https://aiarchitect.atsign.com/)  and enter your email to sign in or sign up. You’ll receive a magic link in your inbox. Click it to access the tool. Once you're in, the workspace will open where you can create a new Blueprint or load an existing one.&#x20;
2. Click on **Start with Example Blueprint** and select the Secure Messaging Blueprint to load our prebuilt example Blueprint. AI Architect will populate the canvas.

{% hint style="info" %}
A Blueprint is a visual map of your application. Each box represents a node. This could be a person, a process, an AI agent, a service, or any other entity involved in your system. The lines between nodes show how information flows from one part of the system to another.

Every node includes a Notes section, which acts as the node’s job description. This is where you define:

* what the node is responsible for
* what information it needs
* how it behaves

A clear Blueprint gives the LLM the structure it needs to build your application in stages.
{% endhint %}

### 2. Export the Prompt

Once the blueprint is ready:

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
{% endhint %}

Visual Studio Code gives you the flexibility to work with a variety of LLMs, not just one. Depending on the extensions you install, you can choose from models like OpenAI, Gemini, Claude and others.&#x20;

To get started, you’ll need to create a new empty folder and set your LLM to **Plan Mode.** Starting in Plan Mode is important because it helps to ensure the LLM is going to build what you want it to build.

1. Paste the exported prompt directly into the chat window. It will plan the project and present the plan to you. When you are happy, Proceed with implementation and it will create files, and set up the app.&#x20;
2. You will be asked to confirm certain actions (file creation, folder setup, dependency installation).
3. The LLM will build a **Dart and Flutter app**. It may build the application in stages, allowing you to test each step and provide additional instructions. It will continue refining and completing the app based on your original prompt as you guide it through each iteration.

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

### 5. Test the App with your Atsigns

To test your app, you’ll need two atSigns. These serve as the identities your app uses during testing. When prompted in your app, you can claim your free starter pack Atsigns either inside the app or through your browser.

1. Visit [**my.atsign.com/starterpack**](https://my.atsign.com/starterpack). If you already have a starter pack, you can log into [**my.atsign.com/login**](https://my.atsign.com/login) to access the Atsigns.
2. Verify your email and claim your two free Atsigns.
3. In your app, go to the authentication screen and choose to activate or create a new Atsign.
4. You’ll receive a one‑time password by email to complete activation.
5. After activation, you’ll be asked to save your atKeys (your cryptographic keys).
6. Try the app by sending messgaes from one device to another.

{% hint style="info" %}
This validates:

* Atsign onboarding
* Secure key management
* Encrypted messaging
{% endhint %}

### 6. Create your own Blueprint

Once you’ve explored the example Blueprint, you’re ready to create your own. Start Simple. A Blueprint doesn’t need to be perfect on the first pass. Its purpose is to help you think clearly about how your application works and to get you to working code quickly and securely.

When designing your Blueprint, focus on three core questions:

1. What are the nodes?
2. What does each node do?
3. How does information move between them?

These three decisions form the foundation of your application’s architecture and guide the LLM as it builds and refines your app.&#x20;

For help with this follow our walkthrough on [**How to Think When Creating a Blueprint**](how-to-think-when-creating-a-blueprint.md)**.**

### Support and Further Help

_If you run into issues, have questions about any step, or want to go deeper into building with the atPlatform, the Atsign team can help. Contact support.team@atsign.com._
