# The Generative AI Application Lifecycle

An important question for all AI applications is the relevance of AI features, as AI is a fast evolving field, to ensure that your application remains relevant, reliable, and robust, you need to monitor, evaluate, and improve it continuously. This is where the generative AI lifecycle comes in.

The generative AI lifecycle is a framework that guides you through the stages of developing, deploying, and maintaining a generative AI application. It helps you to define your goals, measure your performance, identify your challenges, and implement your solutions. It also helps you to align your application with the ethical and legal standards of your domain and your stakeholders. By following the generative AI lifecycle, you can ensure that your application is always delivering value and satisfying your users.

## Introduction

In this chapter, you will:

- Understand the Paradigm Shift from MLOps to LLMOps
- The LLM Lifecycle
- Lifecycle Tooling
- Lifecycle Metrification and Evaluation

## Understand the Paradigm Shift from MLOps to LLMOps

LLMs are a new tool in the Artificial Intelligence arsenal, they are incredibly powerful in analysis and generation tasks for applications, however this power has some consequences in how we streamline AI and Classic Machine Learning tasks.

With this, we need a new Paradigm to adapt this tool in a dynamic, with the correct incentives. We can categorize older AI apps as "ML Apps" and newer AI Apps as "GenAI Apps" or just "AI Apps", reflecting the mainstream technology and techniques used at the time. This shifts our narrative in multiple ways, look at the following comparison.

<img width="1041" height="596" alt="image" src="https://github.com/user-attachments/assets/18434cba-6ba4-4b6f-a3ae-fe829163f918" />


Notice that in LLMOps, we are more focused on the App Developers, using integrations as a key point, using "Models-as-a-Service" and thinking in the following points for metrics.

- Quality: Response quality
- Harm: Responsible AI
- Honesty: Response groundedness (Makes sense? It is correct?)
- Cost: Solution Budget
- Latency: Avg. time for token response

## The LLM Lifecycle

First, to understand the lifecycle and the modifications, let's note the next infographic.

<img width="1041" height="563" alt="image" src="https://github.com/user-attachments/assets/81358088-2f05-4db3-94a8-2cbd41935761" />


As you may note, this is different from the usual Lifecycles from MLOps. LLMs have many new requirements, as Prompting, different techniques to improve quality (Fine-Tuning, RAG, Meta-Prompts), different assessment and responsability with responsible AI, lastly, new evaluation metrics (Quality, Harm, Honesty, Cost and Latency).

For instance, take a look at how we ideate. Using prompt engineering to experiment with various LLMs to explore possibilities to test if their Hypothesis could be correct.

Note that this is not linear, but integrated loops, iterative and with an overarching cycle.

How could we explore those steps? Let's step into detail in how could we build a lifecycle.

<img width="1041" height="563" alt="image" src="https://github.com/user-attachments/assets/f3c0d50d-86a0-4ca7-88c0-8bef82f6113f" />


This may look a bit complicated, lets focus on the three big steps first.

1. Ideating/Exploring: Exploration, here we can explore according to our business needs. Prototyping, creating a [PromptFlow](https://microsoft.github.io/promptflow/index.html?WT.mc_id=academic-105485-koreyst) and test if is efficient enough for our Hypothesis.
1. Building/Augmenting: Implementation, now, we start to evaluate for bigger datasets implement techniques, like Fine-tuning and RAG, to check the robustness of our solution. If it does not, re-implementing it, adding new steps in our flow or restructuring the data, might help. After testing our flow and our scale, if it works and check our Metrics, it is ready for the next step.
1. Operationalizing: Integration, now adding Monitoring and Alerts Systems to our system, deployment and application integration to our Application.

Then, we have the overarching cycle of Management, focusing on security, compliance and governance.

Congratulations, now you have your AI App ready to go and operational. For a hands on experience, take a look on the [Contoso Chat Demo.](https://nitya.github.io/contoso-chat/?WT.mc_id=academic-105485-koreys)

Now, what tools could we use?

## Lifecycle Tooling

For Tooling, Microsoft provides the [Azure AI Platform](https://azure.microsoft.com/solutions/ai/?WT.mc_id=academic-105485-koreys) and [PromptFlow](https://microsoft.github.io/promptflow/index.html?WT.mc_id=academic-105485-koreyst) facilitate and make your cycle easy to implement and ready to go.

The [Azure AI Platform](https://azure.microsoft.com/solutions/ai/?WT.mc_id=academic-105485-koreys), allows you to use [AI Studio](https://ai.azure.com/?WT.mc_id=academic-105485-koreys). AI Studio is a web portal allows you to Explore models, samples and tools. Managing your resources, UI development flows and SDK/CLI options for Code-First development.

<img width="1041" height="563" alt="image" src="https://github.com/user-attachments/assets/711cd6fb-26c3-4693-8f34-29a95762abc0" />


Azure AI, allows you to use multiple resources, to manage your operations, services, projects, vector search and databases needs.

<img width="1041" height="520" alt="image" src="https://github.com/user-attachments/assets/4508b43e-3942-4c34-b7fb-ea91af7b871f" />


Construct, from Proof-of-Concept(POC) until large scale applications with PromptFlow:

- Design and Build apps from VS Code, with visual and functional tools
- Test and fine-tune your apps for quality AI, with ease.
- Use Azure AI Studio to Integrate and Iterate with cloud, Push and Deploy for quick integration.

<img width="1041" height="520" alt="image" src="https://github.com/user-attachments/assets/d1f713d2-6cf9-40fd-85a8-a088e63e81d8" />


## Great! Continue your Learning!

Amazing, now learn more about how we structure an application to use the concepts with the [Contoso Chat App](https://nitya.github.io/contoso-chat/?WT.mc_id=academic-105485-koreyst), to check how Cloud Advocacy adds those concepts in demonstrations. For more content, check our [Ignite breakout session!
](https://www.youtube.com/watch?v=DdOylyrTOWg)
