# unity-ml-agents-tutorial
This is the workflow we collectively developed at the Media Design Master (HEAD–Genève) in order to build an ML Agents project from scratch using Unity 2020.LTS

### Installation

- [Getting Started](https://github.com/Unity-Technologies/ml-agents/blob/main/docs/Getting-Started.md)
- [Installation](https://github.com/Unity-Technologies/ml-agents/blob/main/docs/Installation.md)

1. Install Unity 2020.4 LTS via [Unity Hub](https://unity3d.com/get-unity/download)
    - Always use the `Unity Hub` to install your prefered version of Unity on your computer
    - Avoid directly downloading this Unity version, install it from within `Unity Hub` instead
2. Install Python (3.6.1 or higher)
    - Cf. [Using Virtual Environment](https://github.com/Unity-Technologies/ml-agents/blob/main/docs/Using-Virtual-Environment.md)
3. Clone the [Unity ML Agent Examples](https://github.com/Unity-Technologies/ml-agents/blob/main/docs/Learning-Environment-Examples.md) from GitHub onto your computer using this command from your terminal: `git clone --branch release_16 https://github.com/Unity-Technologies/ml-agents.git
4. On your computer, start the `Unity Hub` app
5. From `Unity Hub`, `Add` the `Projects` sub-folder of the `ml-agents` clone you just downloaded
6. From `Unity Hub`, open `Projects`
7. Once this project is loaded in Unity, open `Window > Package Manager` and import `ML Agents`
8. Open one of the examples: e.g. `Project > Assets > ML-Agents > Examples > 3DBall`

#### Install the Python ml-agents Tool
Follow the “Install ML Agents Python Package” instructions here:
[Installation](https://github.com/Unity-Technologies/ml-agents/blob/main/docs/Installation.md)

We had lots of Windows problems. This is (perhaps) a solution for Windows peoples:
- <https://forum.unity.com/threads/mlagents-learn-is-not-recognized-as-an-internal-or-external-command-operable-program-or-batch-fil.909716>

Training a new brain:

```
mlagents-learn config/ppo/3DBall.yaml --run-id=HelloMediaDesign
```

### Training Our Own Project from Scratch

Make sure your Python installation of ML Agents is working (cf. above)
- Create a new Unity Project (version 2020.4) 
- Window > Package Manager. 
- Change the Packages tab to : Unity Registry
- Find “ML Agents” and click Install
- Create a Scene (a plane, a primitive 3D shape…) 
    - and create your Agent
- Add a Behaviour Parameter component to your agent (this is what you want your brain to do)
- Name your behavior in Behavior Name. This name will be used later in the config.yaml file.

When you add a ray cast sensor, set your vector observation space size = 0. The idea here is that sensors add themselves automatically to the Behaviour Parameter‘s brain, so you do not need to declare them in your space size. If you add any observers in your code, you will need to add those to your count of the space size.
Define your number of branches (= number of actions you’d like to be controlled by your agent). Define if these branches as discrete (like an int) or continuous (like a float).

If you have more than one “type” or “branch” of actions, you have to enter the number of actions for each type. For example: if you have two branches, one for walking and the other for jumping, the first branch (“branch 0”) might have 4 discrete actions (left, right, forward, back), and the second branch (“branch 1”) might have two discrete actions (“jump”, “eat pasta”).

- We will leave the Model empty for now, because we haven’t trained it yet
- If required, add sensors to your agent or to it’s children. (eg ‘ray perception sensor’)
- If you add a RayCast
- add a Tag to the objects you want to be detected by the RayCast 
- To add a tag to an object, scroll down the tag’s tab and press “Add Tag”. Create your new tag and then assign your tag to the object by selecting it.

In your Agent’s ‘Ray perception sensor’ component, write the name of your tag in Detectables Tag (NOT DRAG AND DROP and NOT THE OBJECT ITSELF)
Create a C# script named “AgentTrainer” (or any other name you want)

- Assign the script to your Agent
- Open the script 
- Add : using Unity.MLAgents; into the libraries section
- change Monobehavior to Agent
- From Unity, Add a Decision Requester component to your Agent
- Set the Decision period = 1

This value of 1 is lazy design, but we’re just starting. That means every frame it will calculate everything which often isn’t very elegant or relevant. Later, you should activate the sensors/actions directly from within your code

- In AgentTrainer script, add all the code required to train your behavior and to react once it has been trained
- See code below
- At the root of your main Unity project’s folder (the one that contains the “Assets” folder) create a folder TrainingConfig
- Copy the config.yaml from another example project
- Inside the file, change the name of the behavior to the name assigned at step 5.a.i

- To start training, open Terminal
- Go to the folder of the trainer config (cd /pathToTrainingConfig)
- Line of code to start training in the terminal : mlagents-learn TrainerConfig.yaml --run-id=WhateverNameForYourBrain
- Type “ls” to see files inside to be sure you have “config.yaml” file in the folder
- Type in Terminal:log
- If the unity logo appears, congrats! Just go to Unity and press play
- Wait for training, make yourself a coffee <3 
- When training is done
- Create a Models folder in your assets in Unity
- Import the “NameOfTraining.onnx” file inside
- Drag the onnx file to “model” in the Behaviour Parameters component on your Agent
- Your agent has a brain now!
- If you want to “see” the state of your training
- Open up a second Terminal and point it to your project folder, i.e. the one containing your Assets folder
- If your results folder is named results, type tensorboard --logdir results
- Open the http: link using your navigator

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;

public class AgentTrainer : Agent
{
   public override void Initialize()
   {
       // all the instructions for how to setup
       // the simulation when we start the training
   }

   public override void OnEpisodeBegin()
   {
       // set the starting conditions for each Training Episode
       // where the Agent should be, its speed, its orientation, etc.
   }

   public override void Heuristic(float[] actionsOut)
   {
       // the heuristic method defines a human-defined intelligence
       // either "hard coded" or by user interaction. We control this brain
       // by directly modifying the "actionsOut" list of float values
   }

   public override void OnActionReceived(float[] observationList)
   {
       // we receive from the behavior parameters a list of float values
       // decide what to do with these values (run, jump, move, whatever)
   }

   public void OnCollisionEnter(Collision collision)
   {
       // robot did a bad thing. Give it a negative reward (i.e. punishment)
       if (collision.gameObject.tag == "BadThing")
       {
           AddReward(-1.0f);
           // reset this Learning episode and start a new one
           EndEpisode();
       }
   }
}
```
