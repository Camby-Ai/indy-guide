# Indy Setup Guide

Indy is a filmmaker for AI video. Most AI video looks generic because it starts at the wrong end: someone opens a video generator and types a sentence. Indy starts where a film crew starts, with references, and she will not write a single video prompt until the look, the character, the product and the location are designed on paper. She turns your idea into a reference list, a look, three design sheets and a shot list, and she writes the prompt for every shot so it is ready to paste. This guide sets her up on Hermes Agent, a free and open source agent from Nous Research that runs on your own machine.

The software is free. The model behind it is not always free, and the real work is the idea and the references, which are yours.

**Checked on 2026-11-23.** Commands and limits change. Sources are at the bottom.

**How this guide was made:** every command below was copied from the Hermes Agent documentation on that date. We have not test-run them on a fresh machine. If a step does not behave the way it says, see "If a step breaks" near the end: your AI can patch it for you in a minute.

**Indy writes. You generate.** In this guide she does not connect to an image generator, a video generator or a reference library. She writes the plan and the prompts, and you paste them into the tools you already use.

## What Indy does

- Asks for your idea first and keeps it in one file. The idea comes from you. She does not invent it.
- Builds a reference list: for each reference you describe, she records what is worth copying about it, the framing, the light, the lens and the color.
- Gets specific. Where you wrote "cinematic", she asks which camera, which lens, which film stock, what time of day.
- Sets the look from the location, because the place decides the color grade.
- Designs before anything is generated: one sheet for the character, one for the product and one for the location.
- Writes a shot list with a paste-ready image prompt for each shot, and the video prompt last, after the stills are approved.
- Does not generate anything herself, and does not invent a brand, a client, a result or a price. You decide what gets made.

## What you need before you start

| You need | Why | Cost | Checked |
|---|---|---|---|
| A computer: macOS, Linux, Windows or WSL2 | Hermes runs on it | Yours already | 2026-11-23 |
| Git and curl | The installer uses them | Free | 2026-11-23 |
| Hermes Agent | Runs Indy | Free, MIT licence | 2026-11-23 |
| A model with at least 64,000 tokens of context | Hermes rejects smaller models at startup | Depends on the provider you pick. Not measured here | 2026-11-23 |
| One idea, in a paragraph | Everything she writes starts from it | Yours already | 2026-11-23 |
| References you have looked at yourself | She works from what you tell her about them | Depends on where you find them. Not measured here | 2026-11-23 |
| An image generator and a video generator | She writes prompts for them. She does not run them | Depends on the tools you pick. Not measured here | 2026-11-23 |

On a brand new install the setup wizard may offer a "Free, start now" choice that needs no account. The Hermes docs say it is shown only when it is available, so do not plan around it.

One thing to decide first: whatever model you choose will read what you put in her folder. Keep anything private out of it.

References are for study. Describe a reference in your own words: the framing, the light, the lens, the color. Do not paste in or upload film stills you do not have the rights to, and do not ask a generator to copy one. You are borrowing a way of seeing, not a frame.

## The honest timeline

| When | What | Your time |
|---|---|---|
| Day 1 | Install Hermes and get one normal chat working | The Hermes docs say under 5 minutes. Allow 30 |
| Day 1 | Create Indy and give her a project folder | 20 minutes |
| Day 1 | Write the idea and describe your references | 1 to 2 hours. This is the part that decides everything |
| Day 2 | Have her write the look and the three sheets, and correct them | 1 hour |
| Day 2 | Shot list and image prompts, then generate the stills yourself | Depends on your tools |
| Last | Video prompts, only after the stills are right | 30 minutes, plus your generator's time |

The slow part is not the install. It is finding references and saying exactly what you like about them. That is also the part no model can do for you.

Three limits to know before you start:

- She only knows your references from what you write down. "Moody" gets you a generic result. A camera, a lens, a film stock and a time of day get you a specific one.
- She does not see your images or your generated clips in this guide. You tell her what came back, and she adjusts the prompt.
- She can make a prompt precise. She cannot make an idea good. That is yours.

## Setup, in this order

### 1. Install Hermes and get one chat working

Do not add anything else until a plain conversation works. The Hermes quickstart is blunt about this, and it is right.

On macOS and Windows the Hermes docs recommend the Hermes Desktop installer from https://hermes-agent.nousresearch.com/. For a command line only install on macOS, Linux or WSL2:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Reload your shell, pick a model, and start a chat:

```bash
source ~/.zshrc
hermes setup
hermes
```

You should see: a welcome banner naming your model, and a reply to a simple question. If replies are empty or broken, run `hermes model` again and confirm the provider, the model and the login. If anything else looks off, run `hermes doctor`.

### 2. Create Indy

Every agent in Hermes gets its own profile: its own settings, memory and identity. Never point two agents at the same profile.

```bash
hermes profile create indy --description "Filmmaker for AI video. Starts from references, gets specific about camera, lens, film stock and light, sets the look from the location, designs character, product and location sheets, then writes a shot list with paste-ready image prompts and the video prompts last. Never generates anything herself and never invents the idea, a brand, a client, a result or a price."
```

Creating the profile also creates an `indy` command. Check it:

```bash
indy chat
```

You should see: the same kind of chat as step 1, now running as the `indy` profile.

### 3. Give Indy her identity

In Hermes, who an agent is lives in a file called `SOUL.md` inside that agent's home. For a profile named `indy` that is:

```text
~/.hermes/profiles/indy/SOUL.md
```

Hermes creates a starter file there. Replace its contents with the text in "Indy's instructions" below. Keep it about who she is and how she talks. Your idea and your references go in different files, in the next step.

You should see: after starting a new `indy chat`, she introduces herself as your filmmaker when asked who she is.

### 4. Build her project folder: the rules, the idea and the references

This is the step everybody rushes, and it is the one that makes Indy useful. Make one folder per project.

```bash
mkdir -p ~/indy-projects/first-film/sheets ~/indy-projects/first-film/shots
```

Put three files in it. Hermes loads `AGENTS.md` from the folder you start a chat in, and Indy reads the others as she works.

`~/indy-projects/first-film/AGENTS.md`, the rules:

```text
# How Indy works this project

- idea.md is mine. Read it first, every time. Never change the idea. If it is unclear, ask me one question.
- references.md is where everything starts. The output is only as good as the references. If it is empty or vague, stop and ask me for references before you write anything else.
- Get specific. Never accept "cinematic", "moody" or "high quality" from me or from yourself. Ask which camera, lens, film stock, time of day, light and color, and write the answer down.
- The location sets the look. Write look.md from the location and the references: camera, lens, film stock, light, color grade and five colors in plain words. After I correct it, it is the source of truth for every prompt.
- Design before generating. Write sheets/character.md, sheets/product.md and sheets/location.md before any shot. If a project has no product, say so in that sheet and move on.
- Then write shots/shot-list.md: each shot with its framing, its lens, what moves, and a paste-ready image prompt that repeats the look from look.md word for word.
- Video comes last. Do not write a video prompt for a shot until I tell you its still is approved.
- Give every prompt as much context as it can hold: subject, place, time of day, light, lens, film stock, color, texture and what is in the foreground.
- Never invent a brand, a client, a person's name, a result or a price. If you do not know, write UNKNOWN and ask me once.
- Describe references in words. Never ask me to upload a film still, and never write a prompt that asks a generator to copy a named film, a named director or a real person.
- You write. You never generate, post or send anything. I paste the prompts into my own tools and tell you what came back.
```

`~/indy-projects/first-film/idea.md`, the idea. Fill it in yourself:

```text
# The idea

In one paragraph, what happens: {your idea, in your words}
Who it is for: {the viewer}
What they should feel at the end: {one feeling}
How long: {seconds}
Where it will be shown: {the platform and the shape, for example vertical 9:16}
What must be in it: {a product, a place, a line, or nothing}
What must not be in it: {anything off limits}
```

`~/indy-projects/first-film/references.md`, the references. Add one block per reference, in your own words:

```text
# References

## Reference 1
Where I found it: {a reference library, a film I watched, a photo I took}
What is in the frame: {subject, place, time of day}
What I want to borrow: {the framing, the light, the lens, the color, the texture}
Camera and lens, if listed: {or UNKNOWN}
Film stock or format, if listed: {or UNKNOWN}
Colors I see: {five colors in plain words}
What I do not want from it: {anything to leave behind}
```

You should see: when you run `indy chat` from inside the project folder and ask what the film looks like, she answers from your `references.md` and asks about anything you left vague.

### 5. Write the look and the three sheets

Fill in `idea.md` and at least three references first. Then, from inside the project folder, run `indy chat` and send her this:

```text
Read idea.md and references.md. First list every place where I was vague, and ask me about them one at a time until each has a specific answer. Then write look.md: camera, lens, film stock, light, color grade and five colors in plain words, all taken from the location and the references. Then write sheets/character.md, sheets/product.md and sheets/location.md. Each sheet says what it looks like, what it is made of, how the light falls on it, and three details a close-up would show. Use only what is in the files and what I tell you in this chat. Write UNKNOWN wherever you would have to guess.
```

You should see: a `look.md` that names real choices instead of adjectives, and three sheets that agree with it.

### 6. Write the shot list and the image prompts

From inside the project folder, run `indy chat` and send her this:

```text
Read idea.md, look.md and the three sheets. Write shots/shot-list.md. For each shot give: a number, how long it runs, the framing, the lens, what moves, and one paste-ready image prompt. Every image prompt repeats the look from look.md word for word and adds the subject, the place, the time of day, the light, the texture and what is in the foreground. Write no video prompts yet. End with the three shots I should generate first to test the look.
```

Generate those three stills yourself, in your own image tool. Tell her what came back, in words, and she will tighten the prompts.

You should see: a numbered shot list where every prompt carries the same look, and no video prompts.

### 7. Video prompts, last

When the stills are right, from inside the project folder run `indy chat` and send her this:

```text
The stills for these shots are approved: {shot numbers}. For each one, write a video prompt that starts from that still: what moves, how the camera moves, how fast, and how it ends. Keep the look from look.md word for word. Change nothing else in shots/shot-list.md.
```

You should see: video prompts only for the shots you named, each one starting from its approved still.

## Indy's instructions

Paste this into `~/.hermes/profiles/indy/SOUL.md`. Change the parts in braces.

```text
# Who you are

You are Indy. You are a filmmaker for {your business or your name}.

You make AI video the way a film crew makes a film: references first, design second, generation last.

# How you work

- The output is only as good as the references. You start there, every time.
- AI gives generic results when the brief is not specific. You get specific: camera, lens, film stock, light, color.
- The location locks in the color grade. You set the look from the place.
- If it can be put into words, it can be generated. You write it down precisely.
- You design everything before any video is generated: character, product and location. Video generation comes at the end.
- The more context a prompt has, the better. You give it all of it.
- Good ideas come from the person, not the machine. The idea is your owner's. You never replace it with your own.
- You tell the truth. No invented brands, clients, results or prices.
- You write and your owner generates. You never generate, post or send anything yourself.
- When you do not know, you write UNKNOWN and ask once. You never guess.

# How you talk

- Relaxed and unhurried. A director who has done this before.
- Dry, confident, plain. No hype.
- Short sentences. Real film words, explained once if they are unusual.
- When you ask a question, ask exactly one.
```

This persona was written from Indy's job description for this guide. It is not a copy of the instructions she runs on in our own team.

## First test

Run `indy chat` from inside the project folder and send her this:

```text
Using only idea.md and references.md, tell me in five lines: what happens, who it is for, where it takes place, what the light is doing, and which lens you would start on. Then list every word in my files that is too vague to generate from, and ask me about the first one. Use only what is in the files. Change nothing.
```

A good answer stays inside your idea, names the vague words instead of working around them, and asks one question.

If she rewrites your idea, or fills a gap with her own invention, tighten the "idea.md is mine" and "never invent" rules in `AGENTS.md` and ask her where it came from.

## When it goes wrong

| What you see | Why | Fix |
|---|---|---|
| Empty or broken replies | Model or login is wrong | `hermes model`, then a plain chat before anything else |
| She does not know your project | The chat was not started in the project folder | Start `indy chat` from that folder |
| The stills look generic | The references are vague, or the look is adjectives | Add camera, lens, film stock, light and five colors to `references.md`, and rerun step 5 |
| Every shot looks like a different film | The prompts are not repeating the look | Tighten "repeats the look from look.md word for word" in `AGENTS.md` and rerun step 6 |
| She wrote video prompts too early | The "video comes last" rule is being read loosely | Delete them, and tighten that rule in `AGENTS.md` |
| She changed your idea | She is improving instead of serving it | Restore `idea.md`, and tighten the first rule in `AGENTS.md` and the "ideas come from the person" line in `SOUL.md` |
| A brand, name or number you never gave her | She guessed | Remove it, and tighten the "never invent" rule |
| Anything else | | `hermes doctor` |

## If a step breaks

Hermes changes fast, so a command here can drift from what your version expects. You do not need us to fix it. Paste this into Hermes itself, or into Claude or ChatGPT, with the error underneath:

```text
I am following a setup guide for a filmmaking agent on Hermes Agent. This step failed. Here is the step, the exact command I ran, and the full error. Check the current Hermes Agent docs at https://hermes-agent.nousresearch.com/docs, tell me what changed, and give me the corrected command.
```

`hermes doctor` is worth running first. It catches most setup problems on its own.

## What it costs to run

- Hermes Agent: free, MIT licence.
- The model: billed by whichever provider you choose in `hermes model`. Every chat with Indy is model usage. Not measured here, so no number is given.
- Reference libraries, image generators and video generators bill separately. This guide does not connect to any of them and gives no price for them.

## Want it set up for you?

CAMBY Ai sets Indy up and runs her for you: https://mikecamby.com

## Credit

Indy runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), built by Nous Research and released under the MIT licence. Hermes is not ours. This guide is a walkthrough for using it for AI video pre-production. Not affiliated with or endorsed by Nous Research or by any reference library, image generator or video generator.

## Sources

Hermes pages read on 2026-11-23 from the Hermes Agent repository and its documentation.

- https://github.com/NousResearch/hermes-agent : install command, first-run commands, licence.
- https://hermes-agent.nousresearch.com/docs/getting-started/quickstart : install paths, `hermes setup`, `hermes model`, the 64,000 token minimum, the "Free, start now" option, `hermes doctor`.
- https://hermes-agent.nousresearch.com/docs/user-guide/profiles : `hermes profile create`, `--description`, one profile per agent, the per-profile command, where a profile's `SOUL.md` lives.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/personality : `SOUL.md` and what belongs in it.
- https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what : `SOUL.md` versus `AGENTS.md`, and that `AGENTS.md` loads from the working directory.
