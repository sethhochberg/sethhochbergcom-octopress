---
layout: post
title: "SpaceyDelayer - A Multi-Tap Tape Delay Plugin"
date: 2013-04-12 01:28
comments: false
categories: sound code
---

A fun, VST-compatible audio processing plugin that emulates classic multitap tape delay hardware like the Boss RE-20.

The novel feature of SpaceyDelayer is that the offset time between taps in the delay is configurable - in a typical multitap tape delay, the user can specify the time between the initial playback and the first delay, but with SpaceyDelayer, the interval between the plugin's 3 delay taps is also configurable.

Nonlinear waveshaping and bandpass filtering round out the "tape" sound, emulating saturation and the frequency response weaknesses of these older devices.

All of the DSP "guts" are contained within SpaceyDelayer.cpp and DDLModule.cpp - in each file's respective ProcessAudioFrame() function. The real magic is the getPastSample() function, which allows me to pull an arbitrary sample from the delay line without inserting a new sample (since we pull multiple taps within a single sample period).


{% codeblock lang:cpp %}
// From CSpaceyDelayer::ProcessAudioFrame()

float xn_l = pInputBuffer[0];
float yn_l = 0.0;

// Do LEFT (MONO) Channel; there is always at least one input/one output
// (INSERT Effect)
m_DDL_Left.processAudioFrame(&xn_l, &yn_l, 1, 1);
if(m_uMultipleTaps == On)
{
    yn_l = (yn_l + m_DDL_Left.getPastSample((int)m_fTapSize));
    yn_l = (yn_l + m_DDL_Left.getPastSample((int)m_fTapSize*2));
    yn_l = yn_l * 0.33; //scale back to original-ish amplitude after 3 delay sums
}
pOutputBuffer[0] = m_fWetness*yn_l + (1.0 - m_fWetness)*xn_l;

{% endcodeblock %}

Do a little scaling and summing to allow for wetness values and to avoid clipping, repeat more times if needed for other channels, and you've got the gist of it.

[See the full source code here.](https://github.com/sethhochberg/SpaceyDelayer)


