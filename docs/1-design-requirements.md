# System Requirements: ReadyForm AI

## Introduction

### Purpose

This document describes the system requirements for our proof-of-concept
AI-first approach to ensuring citizens are able to navigate the complexity of
the public service. It particularly enables traditionally marginalized groups to
take independently access government services without the friction and waiting
times of in-person service.

This system is built to push forward the key area of "accessibility" for the G7
GovAI 2025 challenge.

### Roadmap

To implement this system, we have 3 key points that must be developed and
rigorously tested to a high standard:

 1. Ingesting government forms to json, primarily in the form of pdfs. It must
    further then take a filled-out json and fill out the form that the json was
    derived from.

 2. A highly accessible UI for users to see the current state of their progress.
    This includes highlighting fields which need to be filled, the type of
    input expected in that field (money, weight in kg, etc.), a large
    dyslexic-friendly font, and very good compatibility with
    screen-readers/screen-pointers.

 3. A powerful audio-in-text-out llama which guides the user through filling out
    the form. Changes are reflected in real time, and the llama must be able to
    handle non-linear progression through the form, including fixing previous
    values. The llama is entirely powered through audio, so the user could
    presumably close their eyes and get the entire form filled out.
    Additionally, the llama must be able to use a combination of RAG and tool
    calling to provide extended details about each field.


### Definitions

**Llama** - LLM agent with omi-modal input. Particulary,
[ultravox](https://huggingface.co/collections/fixie-ai/ultravox-v06) or
[voxtral](https://huggingface.co/mistralai/Voxtral-Mini-3B-2507) suit our needs.

**kokoro tts** - Light and self-hosted text-to-speech model, available in a
variety of languages and capable of running on commodity hardware in real time.

**Form** - One of the many forms required to complete for access to government
services. For instance a [grain
receipt](https://grainscanada.gc.ca/en/resources/forms/pdf/application-reporting-licensees/grain-receipt-en.pdf).


## Project Details

### Json Representation of Forms

To unify the interface for accessibly editing pdf forms, we need to convert them
into a expressive yet structured format. Json is widely adopted for this task
for its representational power and structure. In fact, json is widely compatible
with structured llm output parsers, which can ensure llm outputs are valid json.

Our system must create a 1-to-1 mapping between json and pdf forms, that is both
invertible and deterministic.

**Testing**. We will test the requirements of our json converter by constructing
an integration test suite using a large number of forms from the Canadian
government's datasets, which have been provided for this challenge.


### Highly Accessible UI

To maintain citizens' trust in our system, we provide a live-updating UI with
access to all the same features that voice mode provides. The large font size
helps seniors and other far-sighted individuals easily confirm that their
choices were correctly understood by the llama. The high contrast and dyslexic
friendly font ensures accessibility by a wide range of citizens. Further, it's
compatible with major screen readers.

For each field, the UI contains the type (money, weight in kg, etc.), and a
clear emoji representing if the field is filled, not filled and required, or not
filled and optional.

### Vocal Llama

While the UI contains visuals for enabling citizens to reference their answers,
the actual answering is done primarily vocally through a llama. We want this to
feel like a conversation, so that citizens are more interested in using this
system as opposed to relying on someone else, giving them a higher level of
independence. The llama will iteratively go through form fields until they're
all full, however a user can interrupt and change their answer for a previous
field.

Once the json is filled out, the llama reviews the information with the citizen
verbally. Once the citizen confirms all the information is correct, the llama
triggers a tool call on our integrated MCP server to convert the json back into
a filled-out pdf form.

Some fields on government forms can be vague or unfamiliar to citizens filling
out this form for the first time. For instance "Dockage" would confuse most
people without additional context, even if they were told that this form is
a grain receipt. Our llama is able to use RAG queries across the entire
public-facing government website to quickly retrieve additional context for
fields on demand. This is also part of preprocessing that allows our UI to
display the type required in each field. MCP tools further enhance this by
allowing execution of custom validators on types, for instance ensuring a phone
number contains all digits.
