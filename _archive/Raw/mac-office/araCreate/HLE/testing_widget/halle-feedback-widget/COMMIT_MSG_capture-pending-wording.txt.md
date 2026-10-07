---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_capture-pending-wording.txt
---

feat: say a picture is being taken instead of "(no picture)"

The review screen opens after CAPTURE_FIRST_PAINT_MS whether or not the
picture is ready, which is correct and unchanged. What was wrong is what it
said: the picture-less state rendered the literal words "(no picture)",
which reads as "this is broken", for as long as the capture was still
working.

There are now two distinguishable states. capturePending says the picture is
still being taken; captureNone says it could not be taken and the report can
still be sent. Only a capture that has actually settled with no picture is
allowed to use the second.

"(no picture)" was also the last hardcoded tester-facing string on the
review screen. Both new strings come from config and are editable in admin,
so the Wording screen guard is satisfied in both directions.

The pending state pulses, and the animation is suppressed entirely under
prefers-reduced-motion — the wording carries the meaning on its own.
