---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_shorthand.txt
---

refactor: use shorthand where rename left x: x pairs

The snake_case rename kept outside names by writing { a: a } wherever the
old and new names met; once both sides were renamed the pair says the
same thing twice. 66 pairs in 27 files become plain { a }.
