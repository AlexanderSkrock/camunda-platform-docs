---
id: orchestrate-apis
title: Getting started with API orchestration
sidebar_label: Getting started with API orchestration
description: "Use Connectors to build low code process automation solutions"
keywords:
  [api endpoints, orchestration, getting started, user guide, connectors]
---

<span class="badge badge--beginner">Beginner</span>
<span class="badge badge--medium">Time estimate: 15 minutes</span>

A Connector is a reusable building block that works out of the box. Each Connector task can be configured with domain-specific parameters without implementing custom business logic.

The concept of a Connector consists of two parts: the business logic is implemented as a job worker, and the user interface during modeling is provided using an element template.

This guide will walk you through working with a REST Connector task as a first time Camunda 8 user.

## Create a REST connector task

To use a **REST Connector** in your process, follow the steps below:

1. Create a BPMN diagram. To do this, navigate to Web Modeler via the **Modeler** tab, and click **New project**.
2. Name your project and select **New > BPMN Diagram > + Create blank**.
3. Give your model a descriptive name and id. On the right side of the page, expand the **General** section of the properties panel to find the name and id fields. For this guide, we'll use `API Orchestration Tutorial` for the name and `api-orchestration-tutorial` for the id.
4. Use Web Modeler to design a BPMN flow with a Connector. Create a Connector by dragging the rectangular task element from the palette, or click the existing start event and the displayed task element to the right of the start event.
5. Change the task type by clicking the wrench icon and select **REST Connector** in the **Connectors** section. Alternatively, you can directly choose a **REST Connector** by using the context pad.

   ![Blank task on Web Modeler canvas with properties panel open](img/connectors-blank-task.png)

6. Add a descriptive name using the **General** section in the properties panel. For this guide, we'll use `Make a request`.

## Make your REST Connector executable

![Connector on Web Modeler canvas with properties panel open](img/connectors-rest-red-properties.png)

To make the **REST Connector** executable, fill out the mandatory **URL** field in the HTTP Endpoint section (highlighted in red) in the properties panel with `https://catfact.ninja/fact` so we can get a random cat fact from the [Cat Fact API](https://catfact.ninja/) for this example.

## Handle your response

The HTTP response will be available in a temporary local response variable. This variable can be mapped to the process by specifying **Result Variable**.
In the **Response Mapping** section use `={"body" : body}` as the **Result Expression** so you can see the entire JSON object returned if it's successful.

## Deploy your process

To deploy your process, take the following steps:

1. Drag the bolded circular end event element from the palette and onto the canvas, or by clicking on the final service task, and then the end event element alongside it. Ensure there is an arrow connecting the service task to the end event.
2. In the top right corner click the blue **Deploy diagram** button. Your diagram is now deployed to your cluster.
   :::note
   If you have not yet created a cluster, clicking **Deploy diagram** will take you to the console to create a cluster. Once you make your cluster creation request, you will automatically be redirected back to Modeler. The creation of a cluster can take 1 to 5 minutes. To read more about creating clusters, visit our documentation on [creating a cluster](create-cluster.md).
   :::
3. Start a new process instance by clicking on the blue **Start instance** button.
4. To the right of the **Start instance** button, click the honeycomb-shaped **Applications** icon. Navigate to Operate to see your process instance with a token waiting at the service task by clicking **View process instances**.

## Additional resources and next steps

- Dive deeper into [Connectors](/components/connectors/introduction.md).
- Learn more about Camunda 8 and what it can do by reading [What is Camunda 8](/components/concepts/what-is-camunda-8.md) or watching our [Overview video](https://bit.ly/3TjNEm7) in Camunda Academy.
