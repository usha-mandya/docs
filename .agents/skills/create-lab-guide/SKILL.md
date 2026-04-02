---
name: create-lab-guide
description: >
  Create a guide page for a Labspace. This includes writing the markdown content for the guide, 
  structuring it according to Docker docs conventions, and ensuring it provides clear instructions 
  and information about the Labspace. Includes learning about the lab itself, extracting out its
  learning objectives, and combining all of that into a well-structured guide markdown file.
---

# Create Lab Guide

You are creating a new guide page for a labspace. The guide should be structured according to Docker docs conventions, 
with clear sections, learning objectives, and instructions for users to get the most out of the lab.

## Inputs

The user provides one or more guides to migrate. Resolve these from the inventory below:

- **REPO_NAME**: GitHub repo in the `dockersamples` org (e.g. `labspace-ai-fundamentals`)

## Step 1: Clone the labspace repo

Clone the guide repo to a temporary directory. This gives you all source files locally — no HTTP calls needed.

```bash
git clone --depth 1 https://github.com/dockersamples/{REPO_NAME}.git <tmpdir>/{REPO_NAME}
```

Where `<tmpdir>` is a temporary directory on your system (e.g. the output of `mktemp -d`).

## Step 2: Learn and extract key information about the lab

The repo structure is:

- `<tmpdir>/{REPO_NAME}/README.md` — the main README for the lab
- `<tmpdir>/{REPO_NAME}/labspace/labspace.yaml` — a YAML document outlining details of the lab, including the sections/modules and the path to their content
- `<tmpdir>/{REPO_NAME}/labspace/*.md` — the content for each section/module (only reference the files specified in `labspace.yaml`)
- `<tmpdir>/{REPO_NAME}/.github/workflows/` — the GHA workflow that publishes the labspace. It includes the repo URL for the published Compose file, which will be useful for the "launch" command
- `<tmpdir>/{REPO_NAME}/compose.override.yaml` - lab-specific Compose customizations

1. Read `README.md` to understand the purpose of the lab.
2. Read the `labspace/labspace.yaml` to understand the structure of the lab and its sections/modules.
3. Read the `labspace/*.md` files to extract the learning objectives, instructions, and any code snippets.
4. Extract a short description that can be used for the `description` and `summary` fields in the guide markdown.
5. Determine if a model will be pulled when starting the lab by looking at the `compose.override.yaml` file and looking for the any top-level `model` specifications.


## Step 2: Write the guide markdown

The markdown file must be located in the `guides/` directory and have a filename of `lab-{GUIDE_ID}.md`.

Sample markdown structure, including frontmatter and content:

```markdown
---
title: "Lab: { Short title }"
linkTitle: "Lab: { Short title }"
description: |
  A short description of the lab for SEO and social sharing.
summary: |
  A short summary of the lab for the guides listing page. 2-3 lines.
keywords: AI, Docker, Model Runner, agentic apps, lab, labspace
aliases: # Include if the lab is an AI-related lab
  - /labs/docker-for-ai/{REPO_NAME_WITHOUT_LABSPACE_PREFIX}/
params:
  tags: [ai, labs]
  time: 20 minutes
  resource_links:
    - title: A resource link pointing to relevant documentation or code
      url: /ai/model-runner/
    - title: Labspace repository
      url: https://github.com/dockersamples/{REPO_NAME}
---

Short explanation of the lab and what it covers.

## Launch the lab

{{< labspace-launch image="dockersamples/{REPO_NAME}" >}}

## What you'll learn

By the end of this Labspace, you will have completed the following:

- Objective #1
- Objective #2
- Objective #3
- Objective #4

## Modules

| # | Module | Description |
|---|--------|-------------|
| 1 | Module #1 | Description of module #1 |
| 2 | Module #2 | Description of module #2 |
| 3 | Module #3 | Description of module #3 |
| 4 | Module #4 | Description of module #4 |
| 5 | Module #5 | Description of module #5 |
| 6 | Module #6 | Description of module #6 |
```

Important notes:

- The learning objectives should be based on the content of the labspace as a whole.
- The modules should be based on the sections/modules outlined in `labspace.yaml`.
- All lab guides _must_ have a tag of `labs`
- If the lab is AI-related, it should also have a tag of `ai` and aliases for `/labs/docker-for-ai/{REPO_NAME}/`
- If the lab pulls a model, add a `model-download: true` parameter to the `labspace-launch` shortcode to show a warning about model downloads.


## Step 3: Apply Docker docs style rules

These are mandatory (from STYLE.md and AGENTS.md):

- **No "we"**: "We are going to create" → "Create" or "Start by creating"
- **No "let us" / "let's"**: → imperative voice or "You can..."
- **No hedge words**: remove "simply", "easily", "just", "seamlessly"
- **No meta-commentary**: remove "it's worth noting", "it's important to understand"
- **No "allows you to" / "enables you to"**: → "lets you" or rephrase
- **No "click"**: → "select"
- **No bold for emphasis or product names**: only bold UI elements
- **No time-relative language**: remove "currently", "new", "recently", "now"
- **No exclamations**: remove "Voila!!!" etc.
- Use `console` language hint for interactive shell blocks with `$` prompts
- Use contractions: "it's", "you're", "don't"

