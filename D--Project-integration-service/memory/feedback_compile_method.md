---
name: compile in IntelliJ IDEA
description: User prefers to compile via IntelliJ IDEA, not Gradle CLI
type: feedback
---

Do not run `./gradlew compileJava` or similar Gradle build commands to verify compilation. The user compiles in IntelliJ IDEA.

**Why:** User preference — they use IntelliJ IDEA as their build environment.

**How to apply:** After making code changes, do not trigger a Gradle compile check. Inform the user of the changes made and let them compile in their IDE.
