# Klippy Log Helper

## Intro

This project was created to help debug errors on Klipper based 3d printers.
Shoutout to the Voron community and its dedicated stewards who helped me learn the basics.

When Klipper throws an error, you don't get a simple "check the crimps on your toolhead MCU" error.

I want to fasttrack that answer as correctly as possible so we can get our machines running more easily.

## How to accomplish this?

Using RAG to create a simple troubleshooting database, based on my own research and
lots and lots of scouring Discord and Esotericals' guides. This will create the database of embeddings.

First, we'll review the logs using an LLM and cross reference that to the embeddings database.

We'll match your error to the most common community sentiment.

Something like:

```
Analyzer Klippy.log
Error found.
Error details: Timer too close

Typical causes:

Wiring (85% likely based on community)
Noise in the CAN/USB network (32% confidence - more details required)
...
...
...(there's a lot of potential causes of this, rate them all with short explanation)

Recommended: firm tug on all crimps from x to y.

If issue persists, ....

```

If it's not straight forward or more information
is required, we will ask a few questions to ensure we have an accurate picture.
