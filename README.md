# Multimodal Live API - Web console

This repository contains a react-based starter app for using the [Multimodal Live API]([https://ai.google.dev/gemini-api](https://ai.google.dev/api/multimodal-live)) over a websocket. It provides modules for streaming audio playback, recording user media such as from a microphone, webcam or screen capture as well as a unified log view to aid in development of your application.


[![Multimodal Live API Demo](readme/thumbnail.png)](https://www.youtube.com/watch?v=J_q7JY1XxFE)

Watch the demo of the Multimodal Live API [here](https://www.youtube.com/watch?v=J_q7JY1XxFE).

---

To get started, [create a free Gemini API key](https://aistudio.google.com/apikey). We have provided several example applications on other branches of this repository:

- [demos/GenExplainer](https://github.com/google-gemini/multimodal-live-api-web-console/tree/demos/genexplainer)
- [demos/GenWeather](https://github.com/google-gemini/multimodal-live-api-web-console/tree/demos/genweather)

Below is an example of an entire application that will use Google Search grounding and then render graphs using [vega-embed](https://github.com/vega/vega-embed):

```typescript
import { type FunctionDeclaration, SchemaType } from "@google/generative-ai";
import { useEffect, useRef, useState, memo } from "react";
import vegaEmbed from "vega-embed";
import { useLiveAPIContext } from "../../contexts/LiveAPIContext";

export const declaration: FunctionDeclaration = {
  name: "render_altair",
  description: "Displays an altair graph in json format.",
  parameters: {
    type: SchemaType.OBJECT,
    properties: {
      json_graph: {
        type: SchemaType.STRING,
        description:
          "JSON STRING representation of the graph to render. Must be a string, not a json object",
      },
    },
    required: ["json_graph"],
  },
};

export function Altair() {
  const [jsonString, setJSONString] = useState<string>("");
  const { client, setConfig } = useLiveAPIContext();

  useEffect(() => {
    setConfig({
      model: "models/gemini-2.0-flash-exp",
      systemInstruction: {
        parts: [
          {
            text: 'You are my helpful assistant. Any time I ask you for a graph call the "render_altair" function I have provided you. Dont ask for additional information just make your best judgement.',
          },
        ],
      },
      tools: [{ googleSearch: {} }, { functionDeclarations: [declaration] }],
    });
  }, [setConfig]);

  useEffect(() => {
    const onToolCall = (toolCall: ToolCall) => {
      console.log(`got toolcall`, toolCall);
      const fc = toolCall.functionCalls.find(
        (fc) => fc.name === declaration.name
      );
      if (fc) {
        const str = (fc.args as any).json_graph;
        setJSONString(str);
      }
    };
    client.on("toolcall", onToolCall);
    return () => {
        client.off("toolcall", onToolCall);
    };
  }, [client]);

  const embedRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (embedRef.current && jsonString) {
      vegaEmbed(embedRef.current, JSON.parse(jsonString));
    }
  }, [embedRef, jsonString]);
  return <div className="vega-embed" ref={embedRef} />;
}

# Problem-Solving and Analytical Thinking Guidelines

## Comprehensive Approach to Solving Complex Problems

### 1. Understand the Problem
Start by rephrasing the issue in your own words, ensuring that you encompass all essential details, goals, and limitations that pertain to the current situation. This step is crucial for achieving a comprehensive grasp of the context.

### 2. Gather Relevant Background Information
Conduct research to collect any significant background information or context that is essential for effectively tackling the problem. It is important to take into account historical, scientific, or situational factors that may influence the possible solutions.

### 3. Identify Key Components
Determine the variables or elements that will substantially affect the outcome of the situation. Take the time to thoroughly evaluate their relationships and dependencies to gain a better understanding of how they might influence the overall resolution.

### 4. Develop Hypotheses
Suggest several potential solutions or outcomes, taking into consideration a variety of perspectives and alternative viewpoints that could be relevant to the issue. This variety in thought can foster more creative solutions.

### 5. Evaluate the Evidence
For each hypothesis you present, perform an analysis of the supporting evidence, possible counterarguments, and any gaps that exist in the available information. It is advantageous to rank the hypotheses based on their feasibility and how well they correspond with the objectives that have been established.

### 6. Apply Logical Reasoning
Utilize either deductive or inductive reasoning to arrive at a conclusion. It is vital to document your reasoning process clearly, offering detailed explanations for each step taken throughout the process.

### 7. Present a Solution
Share your final solution or conclusion, ensuring that it effectively meets all the objectives and constraints that were discussed earlier. Justify your selection by explaining why this specific solution is considered the most effective option available.

### 8. Explore Consequences
Investigate the broader implications of your solution. Consider the long-term effects, ethical considerations, and any potential risks or benefits that may result from the implementation of the solution.

## Example Problem Solving

### Water Shortage Crisis Example

**Problem**: A town is facing a water shortage due to an extended drought.

**Approach**:
1. **Understand the Problem**: The town faces a water crisis due to a drought. The objective is to manage the current shortage while planning for sustainable use.
2. **Gather Context**: Research reveals the town relies heavily on groundwater, which is depleting rapidly.
3. **Identify Core Variables**: Water supply, community needs, infrastructure, long-term sustainability.
4. **Develop Hypotheses**: 
   - Implement strict water rationing
   - Explore water transportation from neighboring regions
   - Invest in water conservation and recycling technologies
5. **Evaluate Evidence**: Analyze cost, feasibility, and long-term impact of each hypothesis
6. **Apply Reasoning**: Develop a multi-pronged approach balancing immediate needs and future sustainability
7. **Present Solution**: A comprehensive water management strategy
8. **Explore Consequences**: Consider social, economic, and environmental impacts

## License
This guideline is open-source and available for adaptation and use.

## development

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).
Project consists of:

- an Event-emitting websocket-client to ease communication between the websocket and the front-end
- communication layer for processing audio in and out
- a boilerplate view for starting to build your apps and view logs

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.
