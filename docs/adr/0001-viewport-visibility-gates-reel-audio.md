# 0001. Viewport visibility gates reel playback, audio included

- **Status:** Proposed
- **Date:** 2026-09-19
- **Deciders:** @parul (proposed during the on-camera reel change, flagged for review)

## Context

The "On camera" section holds a 1:27 self-hosted interview clip (3.3 MB). It was a
click-to-play `<video controls>`; the ask was to have it play silently on its own
when a visitor scrolls to it.

Silent autoplay is uncontroversial. The question that actually needed deciding is
what happens to the **audio** once a visitor has turned sound on with the in-frame
toggle and then scrolls past the section. At that point the only pause control —
the toggle and the native controls — is off-screen, while the audio keeps playing.

This is a single-page portfolio with no other media, so whatever we pick here is
the site's whole media-behaviour policy.

## Options considered

### A. Visibility gates everything, audio included
Leaving the viewport always pauses. Returning resumes from where it stopped.
One rule, no special case for the unmuted state.

### B. Sound-on opts out of the gate
Silent playback pauses off-screen; deliberately-unmuted playback keeps going,
on the theory that unmuting is a signal of intent to listen.

### C. Never pause
Play once on arrival and let it run. Simplest code.

## Decision

Option A. The intersection observer pauses the video whenever it leaves the
viewport, regardless of mute state, and resumes it on return.

## Reasoning

B is the tempting one, and it was close — unmuting genuinely is an intent signal,
and pausing someone mid-sentence because they scrolled is a real cost. What
separated them is that the control does not travel with the sound. Under B, a
visitor who scrolls on hears a voice from nothing, and has to scroll back up to
find the button to stop it. The intent signal is real but it expires the moment
the surface leaves the screen, and a 90-second reel is not a podcast someone means
to keep listening to while reading elsewhere.

The decisive constraint is that there is no persistent player UI on this page and
no plan to build one. Under B the fix would be a sticky mini-player — meaningful
scope for one clip on one portfolio page. A is correct precisely because the page
is small.

C was never viable: it plays the clip off-screen to visitors who never reach the
section, which is both the bandwidth problem and the "starts mid-sentence"
problem that prompted this work.

The bandwidth handling follows from the same rule and is not a separate decision:
`preload="metadata"` until a second observer with a 700px margin promotes it to
`preload="auto"`, so the first frame is buffered on arrival without fetching
3.3 MB for visitors who never scroll that far.

## Consequences

Good: one rule to reason about; no audio without its control on screen; visitors
who never reach the section never download the video.

Bad: a visitor who unmutes and scrolls a little — enough to drop below the 30%
threshold — gets cut off mid-sentence and has to scroll back. We accept that;
resume-from-position keeps it from being destructive.

Also: mute and loop state is now owned by JS rather than HTML attributes, because
the React runtime in `support.js` resets those properties from the bare attributes
on re-render. A `volumechange` handler heals any divergence. Any future media on
this page inherits that constraint.

## Revisit trigger

Reopen if the page gains a second video or a persistent/sticky player UI. With a
control that stays on screen, option B becomes safe and is probably better.
