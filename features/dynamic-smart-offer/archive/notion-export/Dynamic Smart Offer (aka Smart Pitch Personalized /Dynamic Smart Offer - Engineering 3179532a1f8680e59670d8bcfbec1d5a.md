# Dynamic Smart Offer - Engineering

<aside>
💡

Just have AI do everything. 👀

</aside>

## Notes

- Built and tested by end of Q1. That’s March 31. 29 days from now.
- The “Core benefits” `SalesChecklistSection` is the static fallback when this new “Personalized plan benefits” section can’t render. Never show both.
- The “Personalize the coverage” (AKA “Example of coverage”) section will be hidden as well

## Questions

- **Where is the single, public forum where we will communicate about this effort, with all necessary people present?**
- When/if GAIA sends messages to the frontend so we can check items in the UI:
    - What if the event doesn’t arrive or isn’t processed?
    - Can we send all checked items on every send so that they will be caught next time?
    - Can there be a scheduled re-send even if there wasn’t a new item to check off?
        - If we do this, then a missed event will get corrected after a time even if there are no new events to send
- Will the generated smart offer change at all during a call, other than getting events telling the app to check things off? In other words, it doesn’t change during the call unless we click “Try again,” right?
    - “Context-aware tone guidance” seems to indicate that this is all “live” in some way, but the designs don’t indicate that.
    - But this is explicitly marked as a “future requirement”:
        - Dynamic objection handling (rebuttals) triggered during the call (vs. pre-call guidance)
- Who compiles success metrics? What is needed from ExWo to enable that?
- Who keeps the “pool of **pre-approved, marketing-written pitch points”?**
    - Does ExWo keep those in Contentful and then GAIA tells us which to show?
    - Does GAIA keep those and just send us what to render?
        - Less coupling this way