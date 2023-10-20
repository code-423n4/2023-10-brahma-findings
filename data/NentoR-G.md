# Gas Report

## **`currentCommit`** in **`PolicyRegistry.updatePolicy`** could be inlined to save some gas

It's possible to inline currentCommit in the "if" block below it to save some gas. It's used only once and there is no point in storing it into memory.
