# What is this?
The goal of this repository is to allow collaboration on finding a good candidate for new official renderer for Resonite. Resonite is a social VR sandbox where you can build anything in-game, realtime and with realtime collaboration and implicit network synchronization. 

You can get Resonite completely for free on Steam here: https://store.steampowered.com/app/2519830/Resonite/

# Why a new renderer?
This is part of our "Road To New Renderer" project, which aims to replace the current Unity-based renderer with one that we have a full control over. This will not only provide performance benefits, but allow us to add new features.

This project has four phases. This is phase 2.

You can learn more about it here: [Road To New Renderer](https://github.com/Yellow-Dog-Man/Resonite-Issues/discussions/5830)

# Reference repositories
Our current Unity renderer is open source! You can look at its code for reference. It's in these repositories:

[Unity Project](https://github.com/Yellow-Dog-Man/Renderite.Unity.Renderer)

[Renderite](https://github.com/Yellow-Dog-Man/Renderite) (main library & IPC mechanism)

[Shaders](https://github.com/Yellow-Dog-Man/Resonite.UnityShaders)

# What's Phase 2?
In phase 1, we have built a list of requirements - things we need (or want) the new renderer to support. Having this list, we can now look for potential candidates - existing projects that we could base our renderer on. 

The goal of this phase is to look for different candidates and evaluate them against our requirements list. Once we gather enough data, we will make a decision on which one to use and move to phase 3.

## Who will decide which candidate to go with?
We (the Resonite team) will make the final decision. The goal of this phase is to gather as much information to make the best informed decision we can.

We'll be commiting to the new renderer for years, potentialy even decades and ensuring long term content compatibility, so we want to make decision we're comfortable with.

# How can I contribute?
If you'd like to help this phase, help us gather information! That's either suggesting new renderer candidates and going through the list of requirements and evaluating how well they match. The faster we gather this information, the sooner we can move on to phase 3.

## Discuss first!
Use the discussions to talk about new potential candidates and coordinate research with others: https://github.com/Yellow-Dog-Man/Renderite.Candidates/discussions

We'll make a new category for each candidate once it's added to the list to help organize discussion around it.

## Suggest changes through PR
To suggest changes to the candidate documents - make edits to the file and open a PR (pull request)!

- Try to keep the changes small and chunked
- It's better to open several PR's individually for different aspects of each renderer, rather than one huge PR
- Coordinate! Use the discussions!

## Suggesting a new candidate
If you'd like to suggest a new candidate, do following:
1) Create a copy of CANDIDATE_TEMPLATE.md file into the Candidates folder
2) Name the file after the candidate
3) Fill out basic information at the top
4) Create PR adding this file

You don't need to fully evaluate the candidate in the initial PR! Just the basic info is okay to start the discussion on it. Additional changes will be done with more PR's!

## Researching requirements
This is the bulk of what's needed to be done - research each candidate and evaluate them against our list.

1) Look at the current candidate requirement list
2) Any Unknown (❓) requirements need to be researched!
  - Do pay attention to others too - if you feel some are incorrect, suggest corrections!
3) Do the research
4) Use Discussions on this repo to coordinate with other users or discuss the details
5) Once you find information, make changes to the document
6) Open a PR to merge your changes
  - Include links to any resources / discussion for context!

> [!TIP]
> Keep PR's small! They are faster to discuss and easier to merge and will conflict with other suggestions less.
> It's perfectly OK to make lots of small PR's.

### Legend:
Use these symbols in the candidate document to mark the status of each requirement.

❓Unknown - hasn't been evaluated yet
  - These are requirements that still need evaluation. Take a look into these and figure out how they match!

🗨In discussion - feature is harder to evaluate and needs discussion
  - If it's hard to tell if the feature is properly supported or not and needs discussion, you can mark the feature like this
  - Include link to the discussion in the note to direct people towards it
  
✅Fully supported - will fullfill our requirements with full feature parity
  - This was determined to be fully supported by this renderer with no gotcha's or strings attached

⚠️Partial - has similar feature
  - This is supported somewhat, but doesn't quite match what we require and may require changes
  - Please add 📖note on what exactly is not matching

🔜In-progress - this is either currently worked on or is on roadmap
  - Please add 📖note with context
  - Include links to documentation / PR's / issues that have more info
  - If possible, include timeline

🛑Not supported
  - The feature is not supported at all and it doesn't seem to be on roadmap
  - We'll need to evaluate how complex this would be to add ourselves
  - Is possible, add 📖note to provide context on potential implementation complexity if you can

### Notes
📖Note
  - Please add notes where possible with important context to the document!
  - Add them as bulletpoints with the 📖 symbol at start for each feature
  - Only important things please!

**Example:**

-❓Some feature

You can change status to:

-⚠️Some feature
  - 📖 Cannot do XYZ
  - 📖 Could be done via ABC

## When will phase 2 end?
This will likely run for at least a few weeks while we focus on other features (like the IK). We'll close this once we feel comfortable making a decision on which renderer to go with.
