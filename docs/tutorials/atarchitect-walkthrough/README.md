---
icon: arrow-progress
cover:
  light: ../../.gitbook/assets/Hero - atarchitect.svg
  dark: ../../.gitbook/assets/Hero - atarchitect - dark.svg
coverY: 0
coverHeight: 497
---

# atArchitect Walkthrough

This walkthrough guides you through creating, loading, and exporting an app blueprint using atArchitect, then using that blueprint to generate and run an atPlatform app in your IDE.

***

{% stepper %}
{% step %}
### Get Your Starter Pack Atsigns <a href="#step1_atarchitect" id="step1_atarchitect"></a>


{% endstep %}

{% step %}
### Open atArchitect


{% endstep %}

{% step %}
### Load the Demo Blueprint


{% endstep %}

{% step %}
### Export Prompt


{% endstep %}

{% step %}
### Open Your IDE


{% endstep %}

{% step %}
### Build and Run the App


{% endstep %}
{% endstepper %}

***

### 1. Get Your Starter Pack Atsigns <a href="#step1_atarchitect" id="step1_atarchitect"></a>

Before building or testing any atPlatform app, you need two Atsigns. These act as the identities your app will use during testing and development.

1. &#x20;Visit [**my.atsign.com/starterpack**](https://my.atsign.com/starterpack)**.**
2. &#x20;Verify your email and claim your two free starter-pack Atsigns.

{% hint style="info" %}
These Atsigns will be used later when you run your generated app to test authentication and secure communication.
{% endhint %}

### 2. Open atArchitect

atArchitect is the visual blueprinting tool used to design your app’s structure before generating the LLM prompt.

1. Go to [**atarchitect.atsign.com**](https://atarchitect.atsign.com/)**.**
2. This opens the workspace where you create or load a Blueprint. The Blueprint you create here becomes the input for your LLM-powered code generation.

{% hint style="info" %}
A Blueprint is a visual map of your application. Each box represents a node. This could be a person, a process, an AI agent, a service, or any other entity involved in your system. The lines between nodes show how information flows from one part of the system to another.

Every node includes a Notes section, which acts as the node’s job description. This is where you define:

* what the node is responsible for
* what information it needs
* how it behaves

A clear Blueprint gives the LLM the structure it needs to build your application in stages.
{% endhint %}

### 3. Load the Demo Blueprint

To help you get started quickly, you can load a prebuilt example: a secure messaging app Blueprint.

1. Download the demo file from this link: [**Secure Messaging Example**](https://drive.google.com/uc?export=download\&id=1trrDHuDG580yBX9NpVnYtyWUNN1naPwK)**.**
2. Select **Load From File** and choose the file you just downloaded.
3. atArchitect will populate the canvas with the full secure messaging blueprint.

{% hint style="info" %}
The demo blueprint is a complete secure‑messaging example which gives you a fully populated map containing:

* 14 nodes (users, processes, data objects)
* 9 connections
* Messaging flows, group logic, encryption paths
* Entities such as messages, threads, files, and voice notes
* Processes such as identity management, policy enforcement, and message pipeline.
{% endhint %}

### 4. Export the Guide

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

### 5. Open Your IDE

{% hint style="success" %}
You can use any LLM or coding tool you like. We had great results with Claude Code, and will refer to it here.
{% endhint %}

Use an IDE that supports conversational code generation. Using an LLM inside your IDE allows it to create files, folders, dependencies, and project structure automatically. Visual Studio Code with Claude Code is recommended because it supports multi‑file generation, project planning, and iterative refinement.

1. Paste the exported prompt directly into Claude Code. It will plan the project, create files, and set up the app. Approve the actions when prompted.
2. You will be asked to confirm certain actions (file creation, folder setup, dependency installation).
3. Don’t be surprised if the LLM says it has finished while the app is still incomplete, or if the first version isn’t perfect. This is an iterative process. The LLM builds the application in stages, allowing you to test each step and provide additional instructions. It will continue refining and completing the app based on your original prompt as you guide it through each iteration.

{% hint style="info" %}
The LLM will:

* Parse the entire blueprint
* Create the full folder structure
* Generate Dart/Flutter code using at\_client and related packages
* Implement each node as a module or service
* Implement each connection using streams or notifications
* Set up authentication, onboarding, and identity management&#x20;
{% endhint %}

### 6. Build and Run the App

Once the code generation is complete:

1. Follow the build instructions created by Claude Code.
2. Run the app locally.
3. When prompted, sign in using your two starter-pack Atsigns. If you need to access them log in to your [**Atsign Dashboard**](https://my.atsign.com/login).
4. Complete onboarding.
5. Test sending messages between the two Atsigns.

{% hint style="info" %}
This validates:

* Atsign onboarding
* Secure key management
* Encrypted messaging
{% endhint %}

### 7. Create your own Blueprint

Once you’ve explored the example Blueprint, you’re ready to create your own. Start Simple. A Blueprint doesn’t need to be perfect on the first pass. Its purpose is to help you think clearly about how your application works and to get you to working code quickly and securely.

When designing your Blueprint, focus on three core questions:

1. What are the nodes?
2. What does each node do?
3. How does information move between them?

These three decisions form the foundation of your application’s architecture and guide the LLM as it builds and refines your app.

### Support and Further Help

_If you run into issues, have questions about any step, or want to go deeper into building with the atPlatform, the Atsign team can help. Contact support@atsign.com._
