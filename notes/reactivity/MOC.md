# Reactivity & State -- SolidJS Integration

SolidJS reactive primitives drive Lightning GPU nodes directly. No Virtual DOM, no diffing.

## Notes

- [signal-to-gpu-pipeline](signal-to-gpu-pipeline.md) -- signal -> setProperty -> _sendToLightningAnimatable -> lng
- [style-application-rules](style-application-rules.md) -- applied once at creation; JSX props win over style object
- [states-system](states-system.md) -- $-prefixed keys, _stateChanged, Object.assign merge
- [state-order-specificity](state-order-specificity.md) -- Config.stateOrder: later entries = higher priority
- [flatten-styles-first-wins](flatten-styles-first-wins.md) -- earlier array entries have higher priority (opposite of spread)
- [reactive-transitions](reactive-transitions.md) -- transition prop, animateProp, transition={true} for all props
- [task-scheduling](task-scheduling.md) -- scheduleTask paused during focus changes, resumed on idle
